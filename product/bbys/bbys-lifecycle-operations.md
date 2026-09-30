---
last_updated: 2026-08-29 (Loop-3 reconciliation: §"2026-08-29 — from the Data Bridge knowledge
base" gets a 🔴 — the "21st Century is automatic for all files" KB claim had no caveat against
[[partners]]'s Flagsmith-verified finding that 21st-century was absent from prod
`variable_pricing` through 2026-08-13. See [[bbys-pricing-engine]] for the seed-vs-flag drift.)
status: current
source: homelight/hapi @ c14079c76c
scope: BBYS approval types, task catalog, notifications, extensions and repapering
source: HomeLight-Vault/context/bbys-lifecycle-operations.md
imported: 2026-09-29
---

# BBYS lifecycle operations

Approval tiers, the task catalog that drives ops work, the notification catalog, and the
extension/repaper lifecycle. All of it is **configuration, not code** — YAML and database
rows.

Read with [[repo-hapi]], [[bbys-pricing-engine]], [[bbys-stage-progression]],
[[deal-operations]].

## 🔑 Approval types

```ruby
enum approval_type: { express: "express", instant: "instant",
                      light: "light", rapid: "rapid" }, _default: :express
```

**Four tiers, `express` is the default.** `bbys_leads` carries three related columns:

| Column | Meaning |
| --- | --- |
| `approval_type_intake` | what was determined at intake |
| `approval_type` | current |
| `final_approval_type` | at decision |

> 🔑 **These are the `express` / `light` / `rapid` suffixes in Slack deal channel names**
> (`#uwm-appr-hartog-…-ca-express`). The channel name encodes the approval tier.
> **`instant` does not appear in observed channel names** — worth checking whether instant
> approvals get channels at all.

Each tier has its own task template and its own submission notification:
`bbys_approval_type_instant` / `_express` / `_light` / `_rapid`, and
`bbys_lender_lead_submission_instant_approval` / `_express_approval` / `_light_approval`.

Feature flags gate the newer tiers: **`bbys-light-approvals`**, **`bbys-rapid-approvals`**,
`bbys-rapid-approvals-create-tasks`. Interactors: `CreateRapidApprovalTask`,
`UpdateExpressApprovalTask`, `HandleApprovalLeadDisposition`.

Related: `ExpressApprovalSnapshot` (an STI snapshot type) and
`bbys_self_service_express_approval`.

> From Slack: *"missing House Canary AVM can make PCQ n/a; EUR = $0 blocks max-light
> treatment"* — "max light" is ops shorthand around the `light` tier.

## The task system

### Structure

`tasks` is a single STI table: `title` · `note` · `due_at` · `expires_at` · `ready_at` ·
`deferred_at` / `deferred_until` · `completed_at` / `completion_note` / `completed_by_id` ·
`dismissed_at` · `assigner_id` / `assignee_id` / `assigned_at` · **`template_slug`** ·
`type` · `priority` · `task_category_id` · `metadata` (jsonb) · polymorphic `attachable`
**and** `sub_attachable`.

> The **dual polymorphic attachment** (`attachable` + `sub_attachable`) lets one task hang
> off both a lead and a sub-object (a loan, an extension, a document).

### Task automations — data-driven

`task_automations`: `slug` · `title` · `description` · `task_type` (STI) · `priority` ·
`category` · **`assignee`** (a reference to a custom method name) · **`due_at_rule`** (jsonb)
· `extra_attributes` (jsonb) · `enabled` · **`after_run`** (array of follow-up actions) ·
**`condition`** (a reference to a custom method returning true/false).

`task_automation_triggers`: per-automation `condition` + `action` + `metadata`, unique on
`(task_automation_id, action)`.

> 🔑 `assignee` and `condition` are **string references to Ruby methods**, not data. So the
> automation table is only half-configurable — adding a new condition requires a deploy.

### The BBYS task catalog

`services/task_service/config/task_templates.yml` — **157 templates total, 38 BBYS**.
Each has `slug`, `title`, `description`, `priority`, and optional `lead_user_types`.

**Inspection:** `bbys_request_inspection` (Request inspections) · `bbys_schedule_inspection`
· `bbys_complete_inspection` · `bbys_review_complete_inspection`

**Contracts:** `bbys_upload_ir_contract` · `bbys_upload_dr_contract` (Upload DR Client Sale
Contract) · `bbys_review_ir_contract` · `bbys_review_dr_contract` ·
`bbys_under_contract_questionnaire`

**Backup offers:** `bbys_create_backup_offer` · `bbys_review_backup_offer` ·
`bbys_correct_backup_offer` · `bbys_send_approved_backup_offer`

**Closing:** `bbys_trigger_equity_unlock_document_signing` · `bbys_schedule_closing_signing`
· `bbys_schedule_heloc_closing_signing` · `bbys_generate_closing_package` ·
`bbys_record_deed_of_trust`

**Client-facing:** `bbys_submit_property_photos` · `bbys_property_questionnaire` ·
`bbys_document_upload` · `bbys_upfunnel_document_request` · `bbys_soft_credit_check_task`
(*"Identity Check Authorization"* — note the title differs from the slug)

**Approval tiers:** `bbys_approval_type_instant` / `_express` / `_light` / `_rapid`

**Repaper / extension (the 120-day clock):**
`bbys_repaper_loan` · `bbys_day_91_repaper_follow_up` (*"Day 91 - Follow up on Extend vs
Buy"*) · `bbys_day_100_repaper_follow_up` · `bbys_extension_expiring_in_20_days` ·
`bbys_extension_expiring_in_10_days` · `bbys_listing_ops_refresh_valuation` (*"Day 85 -
listing ops refresh valuation"*) · `bbys_purchasing_dr_refresh_valuation` ·
`bbys_trigger_repaper_document_signing` · `bbys_schedule_repaper_closing` ·
`bbys_create_repaper_notary_order`

**Other:** `bbys_hlcs_switch_follow_up` (*"Follow up on switch to HLCS"*)

> 🔑 **The task titles encode the operating calendar**: Day 85 refresh valuation → Day 91
> extend-vs-buy → Day 100 extend-vs-buy → 20 days to expiry → 10 days to expiry. That is the
> back half of the 120-day program.

`DR_CONTRACT_TASK_TEMPLATE_SLUGS = ["bbys_upload_dr_contract", "bbys_review_dr_contract"]`
is called out as a constant on `BbysLead`.

## 🔑 Extensions and repapering

### The constants (`BbysExtension`)

| Constant | Value |
| --- | --- |
| **`BASE_DAYS`** | **120** — the base program period |
| **`MAX_EXTENSIONS`** | **3** |
| **`BASE_FEE_PERCENT`** | **1.2%** |
| **`BASE_FEE_PERCENT_WITH_CONTINGENCY_REMOVAL_ONLY`** | **0.5%** |

> 🔴 **Discrepancy with Slack.** Joel stated on 2026-08-11 that the BBYS program period
> would become **180 days**. `BASE_DAYS` is still **120** at `c14079c76c` (2026-08-28).
> Either the change hasn't shipped, or it lives somewhere other than this constant.
> Worth confirming — it changes every extension expiry date.

### Terms

```ruby
enum term: { "120_days" => "120 Days", "60_days" => "60 Days",
             "pro_rated" => "Pro-rated", "other" => "Other" }
```

Only `120_days` and `60_days` convert to a day count; **`pro_rated` and `other` convert to
0 days** in `convert_term_to_days_number`. Recalculation on term change is explicitly
limited to `%w[120_days 60_days]`.

> ⚠️ An extension with term `pro_rated` or `other` **adds zero days to the expiry
> calculation**. If ops uses those terms expecting a real extension, the maturity date will
> not move.

### Expiry math

```
expires_at = base_date + BASE_DAYS(120) + sum(previous extension days) + term_days
```

`prefill` defaults a new extension to **`60_days`** and returns `nil` once the lead already
has `MAX_EXTENSIONS` (3).

`update_bbys_lead_maturity_date` fires on create, update, and destroy — extensions drive
`bbys_leads.maturity_date`.

### Fee recalculation

`recalculate_fee_charged` and `recalculate_expiration_date` run on term change;
`recalculate_expiration_date_for_next_extensions!` cascades to later extensions, and
`BbysExtension.recalculate_on_sale_price_change!` re-runs when the DR sale price moves —
so **extension fees are a function of sale price**, not fixed at signing.

`extension_risk_adjusted_value` derives from `fee_percentage` and
`dp_gp_percentage_of_valuation`.

### Change tracking

An unusually heavy audit structure: `bbys_extension_change` → `bbys_extension_change_set` →
three typed change components:

- `BbysExtensionExpirationDateChangeComponent`
- `BbysExtensionFeeChargedChangeComponent`
- `BbysExtensionRiskAdjustedValueChangeComponent`

plus `BbysExtensionChangeTracking`. Gated by the **`bbys-extensions`** flag and the
**`REPAPER_EXTENSION_TRACKING`** env var.

### `bbys_loans` — Original vs Repaper

`loan_type` is `'Original'` or `'Repaper'`. `BbysLead has_one :original_bbys_loan`
(`where(loan_type: "original")`).

Loan columns: `loan_funding_amount` · `loan_number` · `loan_lead_number` ·
`warehouse_facility` · `recording_fee` · `notary_fee` ·
`documentary_and_intangible_taxes` · `repaper_started_at` / `repaper_funded_at` /
`repaper_note_at` · `loan_maturity_date` · `loan_closing_date` · **`cltv`** · **`ltv`** ·
`los_current_milestone` · `loan_channel` · `expected_equity_unlock_loan_funded_date` ·
`paid_off_date` · `disbursed_to_title` · `warehouse_funded_at` · `signed_docs_received_at` ·
`modification_date` · `mod_maturity_date` · `incoming_residence_expected_disburse_date`

> 🔴 **`REPAPER_EXTENSION_TRACKING = true` in production** (confirmed in
> `docs/queries/bbys_fee_change_history.md`). This creates a **read/write asymmetry**:
> `BbysLead` **delegates the getters** for `notary_fee`, `recording_fee`,
> `documentary_and_intangible_taxes`, `loan_number`, `loan_funding_amount`,
> `warehouse_facility` to `original_bbys_loan` — **but the setters still write to the
> `bbys_leads` columns.**
>
> Consequences:
> - A change on `BbysLoan` immediately changes what `bbys_lead.notary_fee` *returns*,
>   without touching `bbys_leads.notary_fee`.
> - A change via `BbysLead` writes `bbys_leads.notary_fee` but **does not propagate** to
>   `bbys_loans` — so the write may be invisible to every reader.
>
> **Querying `bbys_leads.notary_fee` directly does not tell you what the application sees.**
> This resolves the open question in [[hapi-bbys-domain]].

## 🔑 Why LPV goes stale — the econ model bug explained

From `docs/queries/bbys_fee_change_history.md`. `loan_payoff_value` (LPV) is recalculated by
`LeadDataService::Bbys::V2::BuildTotalLoanPayoff`, but **only on some paths**:

| Write path | Writes to | LPV recalculated? |
| --- | --- | --- |
| `AssignLeadChanges` (sales-app BbysLead save) | `bbys_leads` | ✅ yes — `BuildTotalLoanPayoff` is in the organizer |
| **`UpdateBbysLoan`** (sales-app "equity and loan information" form) | `bbys_loans` | ❌ **no** |
| MeridianLink `SyncLoanFileFromWebhook` / `SyncLoanFileFromWebservice` | `bbys_loans` | depends — a subsequent BbysLead save may trigger it |

The doc calls the `UpdateBbysLoan` behavior **intentional**: *"LPV recalculation for these
fields is handled by the MeridianLink sync pipeline, not by manual edits."*

> 🔑 This is the mechanism behind Joel's 2026-08-18 report that the econ-model fee was
> *"stuck at $8K"* after the agent value was updated to $800K (which should give $15K). An
> edit on the wrong path updates a value without triggering recalculation.
> `maintenance_reserve_amount` is the exception — it exists **only** on `bbys_leads`, always
> goes through `AssignLeadChanges`, and therefore **always** recalculates LPV.

### Observable audit patterns

| Pattern | Signature |
| --- | --- |
| First MeridianLink sync | `BbysLoan` · fee `NULL→0` · `whodunnit NULL` · LPV unchanged |
| MeridianLink after closing costs set | `BbysLoan` fee `0→150`, then **~130ms later** a `BbysLead` row where LPV moves by the sum of the fee deltas |
| Sales-app user via `AssignLeadChanges` | `BbysLead` · fees **and** LPV in the **same version row** · `whodunnit = user_id` |
| Sales-app `UpdateBbysLoan` edit | `BbysLoan` · fee changes · **LPV unchanged** |

## Auditing changes: the `versions` table

BBYS change history lives in PaperTrail's `versions` table, with a **custom `new_object`
jsonb column** holding `saved_changes`.

| Column | Notes |
| --- | --- |
| `item_type` | `'BbysLead'`, `'BbysLoan'`, `'Lead'`, … |
| `item_id` | primary key |
| `event` | `create` / `update` / `destroy` |
| **`whodunnit`** | user ID **as a string**; **`NULL` = system / background job** |
| `object` | standard PaperTrail previous state |
| **`new_object`** | **custom** — `saved_changes` as JSON |

### ⚠️ The double-encoding gotcha

`new_object` is declared `jsonb` but stores a **JSON string**, so `jsonb_typeof()` returns
`'string'` and **key lookups silently return false**:

```sql
new_object ? 'notary_fee'                    -- ❌ always false
(new_object #>> '{}')::jsonb ? 'notary_fee'  -- ✅ correct
```

Values are `[old, new]` pairs; **absent keys mean the field did not change**. A `NULL` old
value means it was database-NULL, not zero.

### ⚠️ `bbys_leads` has no `lead_id`

Join through the polymorphic `provider_leads`:

```sql
JOIN provider_leads pl
  ON pl.providable_id = bbys_leads.id
 AND pl.providable_type = 'BbysLead'
-- pl.lead_id is leads.id
```

### The in-repo query library

`hapi/docs/queries/` — `README.md`, `AGENTS.md`, and two ready-to-run BBYS audits:
`bbys_fee_change_history.sql` (30-day window, **200-row cap, newest-first — older matches
are silently dropped**) and `bbys_fee_change_history_by_lead.sql` (no limit).

> This library is genuinely good and largely unknown outside engineering. Point people at it
> before they hand-roll audit SQL.

## Notification and email catalog

From `lead_milestone_service` — the BBYS notification surface:

**Stage transitions:** `bbys_lead_stage_update_to_approved` · `_to_agreement_signed` ·
`_to_ir_contract` · `_to_ir_closed` · `_to_dr_closed` · `_to_failed`

**Submission:** `bbys_pre_lead_submission` · `bbys_lead_agent_submission` ·
`bbys_lo_submission_confirmation_next_steps` · `bbys_lender_lead_submission_*_approval`
(instant/express/light) · `bbys_self_service_application_through_lead_capture_link` ·
`bbys_self_service_lead_abandoned` · `bbys_self_service_lead_submission_client_advisor_sms`

**Documents & photos:** `bbys_missing_photos` · `bbys_photo_upload_reminder` ·
`bbys_photo_upload_task_completed` · `bbys_submit_property_photos` · `bbys_document`

**Inspection:** `bbys_order_inspection` · `bbys_order_inspection_reminder` ·
`bbys_client_channel_inspection_reminder_for_client` *(the "not yet built" template from
Slack — the slug now exists)* · `bbys_property_conditions_and_repairs_request` (+
`_client_facing`)

**Notary/closing:** `bbys_notary_scheduling_request` · `bbys_notary_scheduling_email_setup` ·
`bbys_notary_scheduling_valid_template` · `bbys_notary_closing_scheduling_reminder` ·
`bbys_closing_signing_confirmation` · `bbys_send_wire_confirmation`

**Agreement:** `bbys_agreement_expiring_reminder` · `bbys_final_agreement_signed` ·
`bbys_program_agreement` / `bbys-program-agreement-orchard`

**Failure:** `bbys_deal_failure_denial_reason` · `_notes` · `_property_address` ·
`_signatory_details`

**Calculators:** `bbys_agent_equity_unlock_calculator` · `bbys_lender_equity_unlock_calculator`
· `bbys_els_equity_unlock_calculator`

**Orchard-specific:** `bbys_lpv_value_orchard_only`

**Client typing:** `homelight_bbys_all_cash_buyer` · `homelight_bbys_existing_lender` ·
`homelight_bbys_needs_lender`

**Other:** `bbys_equity_boost_notification` · `bbys_backup_offer` · `bbys_next_steps` (+
`_flags`) · `bbys_ir_closed_post_eu_funding_agent_check` · `bbys_application_link`

## BBYS feature flags (65+)

**Application/quiz slugs:** `bbys-application` · `bbys-application-client-direct` ·
`bbys-application-eligible` · `bbys-application-for-agents` · `-for-builders` ·
`-for-lenders` (+ `-v2`, `-v3`) · `-for-lenders-with-heloc` (+ `-2`, `-2-a`, **`-ab-3`**,
**`-ab-3-a`**) · `bbys-prelead-intake` (+ `-for-builders`) · `bbys-quiz-app`

> The `-ab-3` / `-ab-3-a` pair is the A/B test Anirudh rolled to 50% on 2026-07-13.

**Approvals:** `bbys-light-approvals` · `bbys-rapid-approvals` (+ `-create-tasks`)

**Failure:** `bbys-deal-failures-v2` · `bbys-deal-failure-reasons`

**Pricing:** `bbys-pricing-engine-bootstrap` · `bbys-pricing-engine-recalc` ·
`bbys-pricing-engine-ui` · `bbys-pricing-exception-editors` · `bbys-lead-fees`

**Eligibility checks:** `bbys-listing-period-eligibility-check` ·
`bbys-property-value-eligibility-check`

**Ops/roles:** `bbys-partner-managers` · `bbys-payment-operations-admin` ·
`bbys-contact-only-deal-owners` · `bbys-dropout-deal-users` · `bbys-lead-reassign` ·
`bbys-lead-priority-queue` · `bbys-lead-priorities`

**Integration:** `bbys-hubspot-deal-sync` · `bbys-application-hubspot-deal-create`

**Docs/signing:** `bbys-closing-docs-signing` · `bbys-new-notary-signing-flow` ·
`bbys-notary-confirmation-to-lo-or-agent-enabled` · `bbys-final-agreement-regeneration`

**Files:** `bbys-files` (+ `-pre-submission`, `-staging`, `-test`) · `bbys-bucket`

**Self-service:** `bbys-self-service` (+ `-skip-photos`) ·
`bbys-simple-client-approval-flow-email` · `bbys-client-loan-application` ·
`bbys-under-contract-questionaire` *(sic)*

**Other:** `bbys-portal` · `bbys-ca-deals` · `bbys-extensions` · `bbys-loans` ·
`bbys-opportunity` · `bbys-workflows` · `bbys-tasks` · `bbys-next-steps` ·
`bbys-nhc-sourced-leads` · `bbys-referral-lead-labels` · `bbys-lender-lead-submission-flow`
· `bbys-lead-departing-properties` · `bbys-lead-dr-mls-statuses` ·
`bbys-client-task-links-homelight-root`

## 2026-08-29 — from the Data Bridge knowledge base (KB, source: `knowledge_documents`)

New task/pricing/inspection mechanics from the compiled Data Bridge KB (see
[[data-bridge-kb-index]] for the full corpus). Stated (compiled ops answers), not code-verified
here — companion to [[bbys-pricing-engine]] for the pricing facts.

- 🔑 **Variable pricing day-mechanics (BBYS, exception-approved unless a partner auto-rule
  applies):** departing-home close within **day 0–30** of funding → **1.5%** fee. From **day 31**,
  the fee rises **0.015 percentage points per day**. Caps at the standard **2.4%** around
  **day 90**. ("BBYS Variable Pricing: Day 30–90 Fee Mechanics")
- **Automatic variable-pricing partners (no exception routing needed):** APM Equity Unlock deals
  (the **$9,000 minimum fee still applies**) and Arbor. **21st Century** is automatic for *all*
  files regardless of home value — unlike APM, which is gated by home-value/deal-context
  conditions. ("Partner-Specific Variable Pricing: APM and Arbor Guidelines"; "21st Century
  Variable Pricing Partner Guidelines")

  🔴 **2026-08-29 — this KB article's "automatic for all files" claim has no caveat, and it
  contradicts a Flagsmith-verified loop-1 fact.** [[partners]] §"Variable-pricing partner list:
  contradicts bbys-pricing-engine's seed-catalog claim" carries a 🔴: prod Flagsmith
  `variable_pricing` held only **four** partners (`apm, envoy, arbor-financial, nfm-lending`) —
  **`21st-century` not among them** — as of both **2026-08-04** and **2026-08-13** (Slack,
  Taylor Wong / Karly Trota / Carter Marks). [[bbys-pricing-engine]] §"Variable-pricing
  partners" independently notes the seed catalog's fifth entry (`21st-century`) "was ahead of
  the flag" — drift in the *opposite* direction from the usual doc-lags-code pattern. This KB
  article is dated **2026-08-12**, one day before the last confirmed-absent Flagsmith read
  (08-13), so the two may in fact reconcile if 21st-century went live mid-August — but as
  written here it asserts settled policy with no caveat or link, which is the exact "stated
  KB fact silently overwrites a Flagsmith-verified one" failure mode. **Confirm against a live
  Flagsmith read before treating 21st Century as automatic.** See [[risk-register]] for this
  program's other open items.
- **Flat-fee vs. variable-fee exceptions behave differently:** for flat-fee BBYS templates, an
  estimated-fee exception should sync to the actual fee (or the system must disclose it applies
  to both). For variable-fee deals, estimated and actual fees must stay **independent** — the
  actual fee depends on day-count economics that aren't knowable until the departing residence
  goes under escrow. ("Pricing Engine: Flat-Fee vs. Variable-Fee Exception Handling in BBYS
  Deals")
- ⚠️ **A variable-pricing approval does not automatically carry over if the file lands on DTI
  Drop.** If a file with variable-pricing approval ultimately selects the DTI Drop path, the
  operator must flag the pricing/ops owner to update the Sales App economics — the approval was
  made in the context of a different (or undetermined) path. ("Variable Pricing Approval & DTI
  Drop: Sales App Update Protocol")
- 🔑 **Task-approval refresh boundary:** a prior approval within 60 days can be refreshed; older
  than 60 days, the approval process must restart from scratch. A DTI-Drop-Light approval may
  skip property photos, but switching from DTI Drop Light to Equity Unlock requires photos before
  approval. ("Task Approval Refresh & Photo Requirements Guide")
- **Departing-home inspection timing:** the inspection link activates only after BOTH a fully
  executed purchase contract AND a signed HomeLight agreement are in place. Clients generally do
  not pay upfront — cost is carried by the program and settled through lien/program expenses at
  closing. Safety-related repairs may be required before funding; less-significant repairs move
  to pre-HomeLight-purchase repair handling. ("Departing Home Inspection: Timing, Costs, and
  Repair Classification")

### 2026-08-29 addendum #2 (LOOP-2 KB sweep — remaining ~290 title-only rows read via the
compiler's own `summary` column, not full content_text; see [[data-bridge-kb-index]] for method)

- 🔑 **DocuSign is the required flow, not merely preferred.** BBYS agreements must go through
  standard DocuSign to preserve downstream automation; if wet signatures are used as a fallback,
  operators must collect **every page**, including all signature pages, before the package is
  acceptable. Clients sharing an email need a secondary email or documented individual phone
  consent per signer. ("BBYS Agreement Signature Requirements"; "BBYS Agreement Signatures: Keep
  in Normal DocuSign Flow"; "BBYS Agreement Signature: Shared Email & DocuSign Compliance
  Guidance")
- **Late-listing / delayed-vacate requests are limited, reviewable exceptions, not standard
  accommodations.** Route modest delays with a specific vacate date for exception review;
  month-plus or open-ended delays pose real risk to the 120-day sale/payoff timeline and should
  not be approved as routine. HLTE/HLCS involvement can support a modest late-listing exception
  due to earlier transaction visibility, but present it as a conditional lever requiring early
  closing-team handoff, not a blanket waiver. ("BBYS Late-Listing & Delayed-Vacate Guidance";
  "Late-Listing Support: HomeLight Title & Escrow / Closing Services as a Conditional Exception
  Lever")
- **BBYS bridge loans require an executed purchase contract for the incoming property** in every
  scenario — including custom builds, self-builds, escrow-funded, and raw-land purchases — to
  validate fund usage and meet warehouse term-sheet requirements. ("BBYS Bridge Loan: Purchase
  Contract Requirement")
- 🔑 **LO licensing for multi-state files is gated by the purchase-loan state, not the departing-
  home state.** HomeLight covers bridge-loan licensing for the DH state itself, removing a common
  LO-participation barrier on cross-state files. ("BBYS LO Licensing: State Scope for Multi-State
  Files")
- **Offer-type distinction:** BBYS supports non-contingent purchase offers broadly, but an offer
  is only truly "all-cash" when the client's available equity covers the full purchase price —
  don't use the terms interchangeably with clients or agents. ("BBYS Offer Types: Non-Contingent
  vs. All-Cash Offers")
- ⚠️ **The six-month sale timeline is a selective pilot for specific LO-flagged situations, not a
  default program term.** When an agreement nears expiration, check UC status: refresh if not
  under contract, or treat the approval as locked in if the file will move to IR-under-contract
  before expiration. ("BBYS Agreement Timelines: Six-Month Pilot and Expiry Refresh Rules")
- **Cancellation fee boundary:** pre-funding cancellations (before HomeLight funds the loan at
  closing) are generally no-fee for any reason; post-funding cancellations must follow the
  funded-loan/title process and are no longer the standard no-fee path. ("BBYS Cancellation:
  Pre-Funding vs. Post-Funding Boundary")
- **Structural issues on a departing residence (e.g. foundation cracks) should not trigger
  automatic rejection** — require a structural-contractor inspection first, then have Approvals
  classify the finding before issuing final guidance. ("Handling Structural Issues on Departing
  Residences")

## Open questions

- **Is the program period 120 or 180 days?** Code says 120; Slack says 180 was decided.
- Do `pro_rated` / `other` extension terms intentionally add zero days?
- Do `instant` approvals get Slack deal channels? No `-instant` suffix observed.
- `bbys-under-contract-questionaire` is misspelled in the flag key — is the correctly-spelled
  variant also in use anywhere?
