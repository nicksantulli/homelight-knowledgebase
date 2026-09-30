---
last_updated: 2026-08-29 (added §6 — HELOC review canvas operational evidence, army sweep instrument 2)
status: current
source: hapi@c14079c76c | sales-app@bdff5432728 | #homes-ops-and-efficiency-pod (2026-07-20, 2026-08-19) | projects/2026-08-29-shipped-timeline-6mo.md | projects/2026-08-29-slack-product-pods-6mo.md | Slack canvas F0AP55AKHUN "Jacquelyn's Data Issues Log — HELOC Reviews" (read 2026-08-29)
scope: DUS (Document Upload Service) migration — architecture, rollout state, and how it feeds the BBYS AI document-extraction bots
source: HomeLight-Vault/context/bbys-document-pipeline.md
imported: 2026-09-29
---

# BBYS document pipeline — DUS migration

**Document Upload Service (DUS)** is a live, in-flight rewrite of how every BBYS document
gets uploaded, stored, and routed to AI extraction. Taylor Wong's foundational build started
2026-07-14; 15+ PRs across `hapi` and `sales-app` through August. **Zero vault coverage
before this doc** — flagged by [[../projects/2026-08-29-shipped-timeline-6mo|shipped-timeline-6mo]]
as a 🔴 gap. Companion to [[repo-hapi]], [[bbys-lifecycle-operations]], [[hapi-bbys-domain]].

## 1. What DUS replaces, and why

**Old flow (pre-DUS, still the fallback path today):** browser → presigned S3 URL → direct
S3 upload → a synchronous "pdf-merge" step → Google Drive. Extraction triggers, where they
exist, are bolted onto unrelated signals — most visibly HOI declaration extraction, which
today fires off a **task-metadata boolean flipping `false → true`** (`update_bbys_upload_documents_task_metadata.rb`).
That signal doesn't fire reliably across every frontend/API path, and there's a graveyard of
compensating workers around it (`RetryHoiDeclarationProcessingWorker`,
`HoiDeclarationTriggerTimeoutWorker`, `HoiDeclarationExtractionTimeoutWorker`,
`RecoverHoiDeclarationSubmissionWorker`) — four separate workers whose entire job is
papering over one unreliable trigger `(observed, hapi services/lead_data_service/app/workers/lead_data_service/bbys_worker/)`.

🔑 **The core bug that motivated the DUS-extraction integration:** a task can carry up to
**~28 different document slots** (mortgage statement, HOI declaration, credit report, etc.),
most sharing the same generic document type. Knowing "a file landed for task X" doesn't tell
you *which* of the 28 documents it is, especially on multi-file batch uploads. A filename-matching
fallback existed in code to solve this but "nothing actually ever populates the data it
depends on" — dead code, never reachable in production. `(stated, Bruno Gonzalez, #homes-ops-and-efficiency-pod, 2026-07-20)`

DUS fixes this at the root: uploads are explicit, stateful **sessions** (not fire-and-forget
POSTs), each file gets a durable `resource_id`/`resource_type`/`document_type`, and the
pipeline itself — not a side-channel metadata flip — is the extraction trigger.

## 2. Architecture

### Storage: S3 (temp) → Google Drive (permanent)

Documents are **not** stored long-term in S3. S3 is a staging area only:

| Layer | Role |
| --- | --- |
| **S3** (`temp-documents-{env}` bucket, `DocumentUploadService::S3::TempBucket`) | Transient. Source files land at `tmp/{upload_id}/{file_id}.ext`; sanitized/merged output at `sanitized/{upload_id}/{source_id}.pdf`. Presigned PUT URLs, 1-hour expiry. Deleted by the pipeline's final `delete_temp_s3` step. |
| **Google Drive** | Permanent. Final merged/sanitized PDF (or XML) is uploaded via a Lambda step, then the lead's Drive file/folder id is what every downstream system references. |

`bucket_name` picks `production` vs `staging` off `Rails.env` `(observed, s3/temp_bucket.rb)`.

### The upload session model

`DocumentUpload` (table `document_uploads`) has an explicit state machine:
`collecting_files → awaiting_upload → staged → processing → awaiting_external_confirmation
→ confirmed / failed / canceled`. Each session has many `DocumentUploadFile` (role: source /
sanitized_output / merged_output) and many `DocumentUploadStep` (ordered, per-pipeline).

### Two upload modes

| Mode | Flow | Why |
| --- | --- | --- |
| **Staged** (`bbys_deferred`) | `create → register → upload → confirm → stage`, then on lead submit: `POST /internal/uploads/:id/process` enqueues the real pipeline | For app-flow uploads (equity app, public lead submission) that happen *before* a lead exists — files sit in temp S3 only until submit |
| **Immediate** (`bbys_merge_to_drive` / `bbys_single_to_drive`) | `POST /uploads/:id/complete` enqueues the pipeline right away | Everything else (sales-app, LO portal, questionnaires) |

⚠️ **`complete` always enqueues the pipeline — never call it for `bbys_deferred` sessions.**
Use `/stage` instead, or you'll force-process a lead that hasn't been submitted yet.
`(observed, services/document_upload_service/README.md)`

### Pipeline steps (`PipelineRegistry`)

| Pipeline | Steps |
| --- | --- |
| `bbys_deferred` | (none — held until submit) |
| `bbys_merge_to_drive` | verify_sources (hapi) → sanitize_sources (λ) → merge_sources (λ) → upload_google_drive (λ) → drive_document_callback (hapi) → delete_temp_s3 (hapi) |
| `bbys_merge_to_drive_with_extend` | same first 5, then **trigger_extend** (hapi) → delete_temp_s3 |
| `bbys_single_to_drive` | verify_sources → sanitize_sources → upload_google_drive → drive_document_callback → delete_temp_s3 (no merge step — single file) |

🔴 **`bbys_merge_to_drive_with_extend` is dead code as of this SHA.** It's defined in
`pipeline_registry.rb` but **nothing in the codebase ever selects it** — no controller,
worker, or interactor references the string anywhere outside the registry and its spec.
Its `trigger_extend` step runner (`DocumentUploadService::Steps::TriggerExtend`) is also
unimplemented — it inherits `Base#call`, which raises `NotImplementedError`. If this
pipeline were ever selected today, it would crash. This is Taylor Wong's placeholder for
the extraction-trigger work Bruno and Jose scoped on 2026-08-19 (see §3) — **not yet
implemented**, despite being registered. `(observed, services/document_upload_service/app/classes/document_upload_service/{pipeline_registry,step_runner_registry}.rb, cross-checked against full-repo grep for callers)`

Heavy compute (sanitize, merge, Drive upload) runs in **AWS Lambda**, invoked via
`DocumentUploadService::Lambda::Invoker`; `hapi_worker` steps run as ordinary Sidekiq-backed
interactors, one step at a time via `RunNextDocumentUploadStep`, which locks the next
pending step, dispatches it, and re-enqueues itself.

### Feature flags (Flagsmith)

| Flag | Gates | State (2026-08-28) |
| --- | --- | --- |
| `document-upload-service-enabled` | Whether DUS is used at all vs. the old presign→S3→merge path. Enforced server-side (`RequireFeatureEnabled` concern, HTTP 403 if off) and checked client-side in sales-app. | **Enabled** — confirmed live in Slack: "that's controlled by `document-upload-service-enabled` feature flag, which is currently enabled" `(stated, Bruno Gonzalez, 2026-07-20)` |
| `meridianlink-loan-creation-integration-phase2` | Whether **sensitive** document types (1003s, credit reports, mortgage statements, payoff letters, PR visa/passport) route through DUS with their real `document_type`, vs. the non-DUS "other loan docs" bucket. Also gates non-sensitive loan-document DUS routing generally (`handle_loan_document_upload`). | Name is shared with the MeridianLink loan-creation integration — **it is not a DUS-only flag**, it's being reused as the sensitive-document gate. Confirm current on/off state directly in Flagsmith before relying on it. |

⚠️ Despite the brief's framing of "`document-upload-service-enabled` + a phase2 flag," there
is no dedicated `document-upload-service-enabled-phase2` flag in the codebase — the second
gate is the MeridianLink flag repurposed. Anyone reasoning about DUS rollout state should
check both flags by their actual names.

### Extraction routing after Drive upload (`DriveDocumentCallback`)

The `drive_document_callback` pipeline step (a real, implemented interactor — not to be
confused with the unimplemented `trigger_extend`) calls
`LeadDataService::Bbys::V2::RunDriveDocumentCallback`, which branches on `document_type` /
`upload_source` / `source_application`:

| Document type | Handler | Downstream |
| --- | --- | --- |
| `bbys_ir_contract` | `handle_ir_contract_upload` | If `automate_ir_contract_review-pilot` flag on: enqueues `TriggerExtractDataFromContractWorker` → `ProcessIrContractExtraction`. Else: posts to Slack for manual review. |
| `bbys_dr_contract` | `handle_dr_contract_upload` | Always enqueues `TriggerExtractDataFromContractWorker` → `ProcessDrContractExtraction`. |
| `bbys_backup_offer` | `handle_backup_offer_upload` | Moves file to contracts folder — feeds the BUO review bot (§5). |
| `equity_boost_questionnaire` | `handle_equity_boost_questionnaire_upload` | `HandleEquityBoostDocumentUploadWorker`. |
| Sensitive (1003, credit report, mortgage statement, payoff letter, PR visa/passport) | `handle_sensitive_document_upload` | Gated on `meridianlink-loan-creation-integration-phase2`. Enqueues `HandleSensitiveDocumentsUploadWorker` — `primary_1003` runs immediately, everything else delayed 3 minutes. |
| HELOC-targeted | `handle_heloc_document_upload` | Falls through to generic notification path. |
| LO portal / questionnaire, other | `handle_loan_document_upload` + `handle_other_document_uploads` | Also gated on the MeridianLink phase2 flag; renames file in Drive, notifies via debounced worker. |

🔑 `lite_parser_extract_run_id` is threaded through the sensitive-document path by reading
a `Snapshot` keyed on the BBYS lead's `uuid`, payload path
`lite_parser_extract_run_ids[document_type]` — this is how the 1003-lite-parser result (§5)
gets attached to the same worker call that also kicks off full Extend extraction.

## 3. Rollout state — the 2026-08-19 extraction-trigger redesign

`(stated + observed, #homes-ops-and-efficiency-pod, 2026-08-19, message_ts 1787173528 thread; artifact: claude.ai/code/artifact/ff1585fb-1414-4fbd-b76a-2830816e1edc)`

Bruno Gonzalez walked Jose Herrera through a plan to fix HOI declaration extraction, which
was failing for **~47 unprocessed documents** because its trigger (task metadata
`uploaded: false → true`) doesn't fire on every frontend/API path (the attachable-tasks API
in particular never fires it).

**Decisions made (Jose approved):**
1. Move the HOI extraction trigger off task metadata and into the DUS pipeline itself —
   extraction fires only once the file is confirmed in Drive, eliminating the 5-minute retry
   band-aid.
2. **Rename `trigger_extend` → `trigger_extraction`**, reusing Taylor's empty placeholder
   classes, to keep the step provider-agnostic (Extend today, Gemini and future providers
   later — see `sensitive_gemini_extract_worker.rb`, which already exists for sensitive-doc
   extraction).
3. New pipeline: `single_to_drive_with_extraction_steps` (not yet in the registry as of
   `c14079c76c`).
4. **Scope limited to HOI.** Backup offer, IR/DR contract ("ten-three" per Bruno's phrasing),
   and credit/mortgage extraction stay on their current triggers — not touched by this
   migration.
5. Known risk flagged in the same discussion: the current Drive → StringIO → HAPI stream
   flow for HOI will eventually break on large documents; that gets replaced as part of this
   work.

**Status as of `c14079c76c` (2026-08-28), i.e. 9 days after the plan was approved:** ⚠️ **not
yet implemented.** No `trigger_extraction` step, no `single_to_drive_with_extraction_steps`
pipeline, and `trigger_extend` is still the unimplemented stub in the live registry (§2). The
HOI extraction trigger is still the fragile task-metadata flip as of this SHA. Confirm with
Bruno directly before assuming this has shipped.

> 🔴 **Update 2026-08-29 — the answer to §7 Q1 is "in flight, not merged."**
> `homelight/hapi#20127` "Harden document extraction workflows and add extraction pipelines"
> (bruno-uy, opened 2026-08-29, still **DRAFT**) is **PR 1 of 2** for exactly this migration.
> It adds `document_upload_service/app/interactors/.../steps/trigger_extraction.rb`,
> `pipeline_registry.rb`, and `step_runner_registry.rb` — but the PR body states the registry
> ships **deliberately empty**: no document type is activated, no HOI handler is registered,
> and production HOI behavior does not change yet. The fragile task-metadata trigger is still
> live in production as of this note. PR 2 (activation, typed checklist category, task
> backfill, legacy trigger removal) has not been opened. Source:
> [[../projects/2026-08-29-open-pr-roadmap]].

## 4. What's cut over vs. still on the old path

| Surface | Status |
| --- | --- |
| IR contract upload | DUS (`uploadIrContractViaDus.ts`), replacing presign→S3→pdf-merge |
| DR contract upload | DUS (`uploadDrContractViaDus.ts`), same replacement |
| Backup offer upload | DUS (`uploadBackupOfferViaDus.ts`) |
| 1003 (primary/additional) | DUS **iff** `document-upload-service-enabled` AND MeridianLink phase2 flag both on (Story 11, per code comment) |
| Credit report | Same dual-flag gate (Story 8) |
| Mortgage statement, payoff letter | Same dual-flag gate |
| HOI declaration | Uses DUS for storage/Drive merge (per Taylor, 2026-07-20: "HOI declaration should be migrated to DUS"), but its **extraction trigger is not yet on the DUS pipeline** — see §3 |
| Equity app (equity-app pages in hl-fe-monorepo) | ⚠️ Per Faaz, 2026-07-20: "equity-app might not use DUS" for at least some flows — not confirmed cut over |
| Non-sensitive "other loan documents" | DUS if `document-upload-service-enabled` alone is on (no phase2 gate) — see `shouldUseDusForOtherLoanDocument.ts` |

## 5. The AI document-extraction angle — how DUS feeds the review bots

DUS is the **plumbing**; `document_data_extraction_service` (DDE) + **Extend AI** is the
**intelligence layer** it feeds. DDE's `ExtendDataExtractionManager::WORKFLOW_MAPPING` maps
document types to Extend workflow IDs — this is the authoritative list of what HAPI can
extract today: `real_estate_purchase_contract`, `backup_offer_contract`, `primary_1003` /
`additional_1003`, `equity_boost_questionnaire`, `primary_credit_report` /
`additional_credit_report`, `mortgage_statement`, `payoff_letter`, `hoi_declaration`
`(observed, extend_data_extraction_manager.rb)`.

Three consumer products sit on top of this extraction layer — documented in
[[../projects/2026-08-29-slack-product-pods-6mo|slack-product-pods-6mo]], cross-linked here:

| Product | Trigger today | Extraction | Vault status |
| --- | --- | --- | --- |
| **BUO (Backup Offer) review** | `handle_backup_offer_upload` in `DriveDocumentCallback` | Extend AI (`backup_offer_contract` workflow) | Most mature; live since 2026-06-18; 99.07% claimed field accuracy at launch |
| **HOI declaration review** | Task-metadata flip (fragile — mid-migration to DUS, §3) | Extend AI (`hoi_declaration` workflow); 95%-of-RAV coverage floor is a hard-coded gate | Live, catching real shortfalls; bot has override authority over human-set verdicts (⚠️ per prior sweep, sign-off on that authority is unconfirmed) |
| **1003 lite parser (HELOC)** | Enqueued alongside sensitive-document extraction via `lite_parser_extract_run_id` (§2) | A **separate, lighter parser pass** from the full Extend workflow — `LeadDataService::Heloc::Detect1003Anomalies` compares lite-parser fields (`property_value`, `purchase_mortgage_interest_rate`, `total_gross_monthly_income`, `all_liabilities`) against Sales-App HELOC data (`heloc_home_value_estimate`, etc.) and can **auto-stop HELOC prequalification** via the `stop_auto_prequal_on_file_upload` flag when a discrepancy is detected | Not previously documented anywhere in the vault — this doc is first coverage |

🔑 **The 1003 lite parser is not the same thing as the full 1003 Extend workflow.** Both
extract from the same document type, but the lite parser is a distinct, faster pass whose
output specifically feeds HELOC anomaly detection and can block auto-prequalification —
don't conflate "1003 extracted" with "lite parser ran."

DDE also has a Gemini-based extraction path for sensitive documents
(`sensitive_gemini_extract_worker.rb`) alongside the Extend-based one — worth a closer read
before assuming Extend is the only extraction provider in play; this is likely the reason
Bruno's 2026-08-19 rename (`trigger_extend` → `trigger_extraction`) explicitly aimed for
provider-agnostic naming.

## 6. Operational evidence of extraction-quality gaps (2026-08-29 addition)

**Source:** Slack canvas "Jacquelyn's Data Issues Log — HELOC Reviews" (`F0AP55AKHUN`,
created by Marc Kaplan 2026-03-26; auto-updated end-of-weekday from
`#heloc-review-notifications`). Read via `slack_read_canvas` 2026-08-29 — first vault
coverage of this canvas.

The canvas is an operator-run incident log: a HELOC reviewer (Jacquelyn Houston, then Marc
Kaplan covering, then Sean Gilliland) manually flags deals where Sales-App (SA) data pulled
from BBYS documents was missing, wrong, or unparseable. **This is the human-QA signal
sitting downstream of the extraction pipeline in §5** — it does not identify which parser
(Extend vs. the 1003 lite parser vs. Gemini) produced each bad field, but it is direct
evidence of extraction failure rate and failure mode over a 5-month window.

**72 deals flagged, 2026-03-16 → 2026-08-10** (stated, per the canvas's own running
summary as of 2026-08-26 — no new flags that day). Issue-type breakdown (rows can carry
more than one tag):

| Issue type | Count | Example |
| --- | --- | --- |
| Incorrect Data | ~37 | Field present but wrong — e.g. income parsed as $17K instead of $2M+/mo, blowing DTI (Lead 15275034, flagged by Sean Gilliland 2026-08-10) |
| System/Parse Error | ~20 | Automated parse produced bad data outright — e.g. a departing-residence (DR) liability marked "to be paid off at/before closing" still counted against DTI, requiring manual re-run (Lead 15032728) |
| Missing Data | ~15 | Required field absent from SA/application — no purchase price, no liabilities, no IR info |
| Incomplete App | 8 | App itself incomplete, or docs uploaded to the wrong slot — e.g. credit report and 1003 uploaded in swapped locations, breaking the optimizer amount (Lead 14766998, "$3k approved vs $2k true max") |

🔴 **Stated dominant pattern: DR mortgage/departing-residence liabilities, ~39% of all 72
flagged deals.** The canvas's own summary calls this out by name as "the dominant systemic
issue" — consistent with, and a concrete quantification of, the 1003 lite-parser →
`Detect1003Anomalies` liabilities comparison in §5 (`all_liabilities` is one of the four
fields that pipeline checks). This 39% figure is **stated by the canvas author, not
independently re-tallied here** — the per-row issue-type tags don't cleanly map to a single
root-cause category, so treat 39% as directionally right, not exact `(stated)`.

⚠️ **Coverage gap, not just a data-quality gap:** the reviewer role itself has been
unstable — Jacquelyn flagged 42 deals through 5/15, Marc Kaplan covered and flagged 29 more
5/26–6/12, then the log goes quiet until Sean Gilliland's single 8/10 flag. A ~6-week
silent gap (mid-June to early August) in a log that's supposed to auto-update daily reads
either as genuinely zero issues (unlikely given the rate before it) or as **the manual
review step itself lapsing** — the canvas gives no way to distinguish the two. Not
reconciled against [[bbys-document-pipeline]] §3's extraction-trigger redesign timeline
(2026-08-19) — worth checking whether the redesign coincides with the review gap.

## 7. Intersection with the BBYS task/document catalog

[[bbys-lifecycle-operations]] documents the **task template layer** (`task_templates.yml`,
157 templates, 38 BBYS) — DUS operates one layer below that, as the **upload transport**
those tasks trigger. Relevant task slugs from that catalog and how they map to DUS document
types:

| Task slug (from [[bbys-lifecycle-operations]]) | DUS `document_type` | Notes |
| --- | --- | --- |
| `bbys_upload_ir_contract` | `bbys_ir_contract` | §2 routing table |
| `bbys_upload_dr_contract` | `bbys_dr_contract` | §2 routing table |
| `bbys_document_upload` | generic / varies by slot | The default document-center upload task; DUS resolves the specific slot via `resource_id`, not the task slug alone |
| `bbys_upfunnel_document_request` | varies | `BbysUpfunnelDocumentRequestBackdoor.tsx` checks `documentUploadServiceEnabled` directly |
| *(not in task catalog)* `bbys_under_contract_questionnaire` uploads | HOI declaration primarily | This is the flow with the `resource_id` bug Bruno found 2026-07-20 — the questionnaire never sent `task_id`, so DUS couldn't correlate the file to a specific document slot |

Do not duplicate the task-catalog table here — [[bbys-lifecycle-operations]] is the source of
truth for task slugs, titles, and the operating-calendar semantics (Day 85/91/100). This doc
only adds the upload-transport layer underneath those tasks.

## 8. Open questions

- ✅ **Answered 2026-08-29** — see §3 update. `hapi#20127` is open (DRAFT, PR 1 of 2) but
  ships an empty registry; production behavior is unchanged. Remaining open question: when
  does PR 2 of 2 (activation) land, and is there a date?
- What is the **current** on/off state of `meridianlink-loan-creation-integration-phase2` in
  Flagsmith? This doc could only confirm it exists and what it gates, not its live value —
  Flagsmith wasn't queried directly.
- Does `equity-app` (hl-fe-monorepo) use DUS for any upload flow today, or is Faaz's
  2026-07-20 "might not use DUS" still true? Unconfirmed either way.
- Is `bbys_merge_to_drive_with_extend` genuinely dead, or is it mid-flight scaffolding for
  a different in-progress PR not yet visible at this SHA?
- What triggers extraction for **credit report / mortgage statement / payoff letter** today
  — the same `handle_sensitive_document_upload` path as 1003s? Not independently verified;
  inferred from shared `SENSITIVE_DOCUMENT_TYPES` list.
- Who owns the DUS migration overall — Taylor Wong built the foundation, Bruno Gonzalez is
  driving the HOI extraction-trigger fix, Faazil Shaikh is credited with "DUS wiring" in the
  shipped-timeline author table. No single named owner found.
- Does the 47-unprocessed-HOI-documents backlog from 2026-08-19 get backfilled once the new
  trigger ships, or are those leads permanently missing extraction?

## Open questions carried from source docs

- [[../projects/2026-08-29-slack-product-pods-6mo|slack-product-pods-6mo]] asks whether
  ops signed off on the HOI bot's authority to overwrite human-set verdicts — still
  unanswered; relevant here because that override behavior sits directly on top of the DUS
  extraction pipeline described in §5.
