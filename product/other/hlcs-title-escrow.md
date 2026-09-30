---
last_updated: 2026-08-29
status: current
source: homelight/hapi@c14079c76c9a94bc1c9686fba7ec32614bde0dd3 (2026-08-28)
scope: HLCS — HomeLight Closing Services (title + escrow): order lifecycle, Qualia integration, notary/e-signing, settlement teams, disbursements
source: HomeLight-Vault/context/hlcs-title-escrow.md
imported: 2026-09-29
---

# HLCS — HomeLight Closing Services (title & escrow)

HomeLight owns and operates title & escrow companies. In HAPI this is the
**`escrow_order_service` engine — 341 files, the third-largest engine in the repo** and
the largest non-BBYS, non-disposition one. Supporting cast: `ClosingSigningService` (60),
`DisbursementService` (90), plus `TransactionService`/`TransactionDataService`.

Object-model background in [[repo-hapi-platform]]; marketplace context in
[[homelight-agent-marketplace]].

> ⚠️ **HLCS is a real title/escrow business, not a referral.** Unlike agent matching,
> HomeLight is the counterparty: it opens the order, holds escrow, runs the settlement
> team, schedules the notary, disburses, and records. That's why this engine dwarfs the
> others.

## Where a closing lives in the object model

```
Lead(user_type: "escrow" | "title")
  └── ProviderLead(providable_type: "EscrowOfficeLead" | "TitleOfficeLead" | "CommonOfficeLead")
         └── EscrowOfficeLead ──belongs_to──> ServiceOffice ──> State
                │                └──> ServiceSource (which back-office platform)
                └──has_one──> EscrowOrderService::EscrowOrder   (the Qualia mirror)
```

An escrow lead is usually a **child lead**: `EscrowOfficeLead has_one :parent_lead,
through: :lead`. The parent is the agent-referral or BBYS lead that produced the closing.
`Order#escrow_lead` and `#all_stage_escrow_lead` pull it back from the Order bundle.

`ProviderLead::SERVICE_PROVIDABLE_MAPPING` maps all three office-lead types to
`"title_escrow"`, but `Order::CLOSING_SERVICES` calls the same business unit
`"closing_services"`. 🔴 **Two names for one business line — check which vocabulary a
report is using.**

`ProviderLead#reference_id` for escrow leads is `<STATE>-<7 chars>` (e.g. `TX-9KQ2MNP`),
not the 16-char code used everywhere else — that's the number ops quotes to a client.

### Buyer side vs seller side

`EscrowOfficeLead.buyer` (boolean) is how one escrow lead is assigned a side.
`Order#transaction_leads` joins it explicitly:

```sql
(escrow_office_leads.buyer = :buyer AND leads.user_type = 'escrow'
   AND leads.stage IN (:FUNCTIONING_ESCROW_LEAD_STAGES))
 OR leads.user_type IN (:user_type_list)
```

`LeadStage::FUNCTIONING_ESCROW_LEAD_STAGES` = `new · requested · awaiting_agent ·
processing · on_hold · in_pre_escrow · in_escrow · pending_order`. `Lead.functioning_escrows`
uses this; `Lead.progress_tracker_escrows` narrows to `secondary_user_type = "buyer"`.

### EscrowOfficeLead stages (11) — a vocabulary of its own

`new · processing · on_hold · in_pre_escrow · in_escrow · closed · closed_paid ·
awaiting_agent · pending_order · duplicate · failed`

`ACTIVE_STAGES = new · processing · in_pre_escrow · in_escrow` (note: **`on_hold` is not
"active"**, and `awaiting_agent`/`pending_order` are not either — but all three *are*
"functioning" per `LeadStage`). ⚠️ Two different "is this escrow alive" definitions live
side by side.

`EscrowOfficeLead#transaction_type` is derived, not stored:
`refinance → "Refinance"`, `started_as_pre_escrow → "Pre-Escrow"`,
`seller_coordination → "Seller Coordination"`, else `"Purchase"`.

Other flags: `hl_lead?` (walks to the parent lead's `hl_lead`), `instant_open?`
(`lead.source_form == "instant_open"`), `order_placed_within_sales_app?`
(`source_page_type == "sales_app"`), `unmapped` scope (`order_identifier IS NULL` — an
escrow lead with no back-office order yet).

## Service offices and service sources — the operating footprint

| Model | Role |
| --- | --- |
| `ServiceOffice` | a physical/legal title-escrow office. `belongs_to :state`, has `settlement_agency_id`, `platform_api` (default `"homelighttitle"`), contact details, dashboard |
| `ServiceOfficer` | staff at an office |
| `ServiceSource` | the **back-office platform adapter** an office uses |
| `ServiceOfficeSource` | join; `service_name` is what `ServiceOffice#supported_services` returns |
| `SettlementAgencyTeam` (a `Team`) | the staffing model, joined to `ServiceOffice` by `external_id ↔ settlement_agency_id` |

`ServiceSource` validates that `EscrowOrderService::#{adapter_class_name}ServiceSourceAdapter`
resolves. Adapters that exist:

`BaseServiceSourceAdapter::ADAPTER_NAMES` (7): `HomeLight · HomeLightAZ · HomeLightFL ·
HomeLightIL · HomeLightPA · HomeLightTX · ChicagoTitle`. Two more adapter **classes**
exist on disk but are **not** in `ADAPTER_NAMES`: `SnapNHDServiceSourceAdapter` and
`TestServiceSourceAdapter` — so a `ServiceSource` row could reference SnapNHD and pass
`adapter_exists` validation while being invisible to anything iterating `ADAPTER_NAMES`.

> 🔑 **The state-suffixed adapters (AZ, FL, IL, PA, TX) are the observable footprint of
> HomeLight's own title/escrow entities.** Plus a `ChicagoTitle` adapter for a partner
> underwriter. `BaseServiceSourceAdapter::DEFAULT_SENDER` is
> `title@homelight.com / "The HomeLight Title Team"`, and `default_cc` copies the same
> address on everything. `ServiceOffice::TITLE_ONLY_SERVICE_OFFICE = "Glendale - HomeLight
> Title & Escrow Company"` — one office is title-only. `SO_CAL_COUNTIES` (10 counties) is
> hardcoded for NorCal/SoCal routing.

`fallback_settlement_agencies` and `preferred_escrow_officer_fallback_contact_information`
are SalesSettings — routing fallbacks are ops-editable.

### Settlement team roles

Two role vocabularies, and they are **not** the same list:

- `SettlementAgencyTeam::ROLES` = `branch_manager · state_manager · escrow_officer ·
  escrow_assistant` (4).
- `TransactionTeam::CLOSING_SERVICES_ROLES` = `order_opener · escrow_officer ·
  escrow_assistant · branch_manager · state_manager` (5 — adds `order_opener`).

`TransactionTeam::ROLES` overall carries **32 roles** across the whole company
(client_advisor, listing_specialist, lender_relationship_manager, lo_sales_owner,
strategic_relationship_manager, transaction_coordinator, valuation_analyst,
post_closing_specialist, builder_representative, …). Note `DEPRECATED_ROLES =
client_manager · loan_officer · loan_officer_assistant · loan_officer_additional_contact`,
marked "remove as soon as Centralized Lead Distribution is fully enabled."

> ⚠️ Two individual employee emails are **hardcoded** in `TransactionTeam`:
> `PROCESSOR_USER_EMAILS = ["rhyna.coloma@homelight.com", "pat.torres@homelight.com"]`
> and `PROCESSOR_TEAM_MANAGER = "derek.lupien@homelight.com"`. Also
> `BUILDER_POD_NAMES = ["Builder - Michael Coffey Team", "Builder - Tiffany Traxler Team"]`.
> These break on personnel change without a deploy.

`EscrowOfficeLead#escrow_officer_settlement_team_member` matches on
`role = TransactionTeam::ESCROW_OFFICER[1]` — the **human-readable** "Escrow Officer",
while `primary_service_officer` also uses the string `"Escrow Officer"`. Elsewhere the
snake_case form is used. 🔴 Role matching is inconsistent between snake_case and Title
Case in this engine.

## Qualia — the system of record for the order itself

HAPI does **not** run the escrow file. **Qualia** does. `escrow_orders` is a mirror.

- `escrow_orders`: `external_id` (Qualia order id, required), `order_number`, `status`,
  `purchase_price`, `earnest_amount`, `estimated_closing_at`, `contract_date`,
  `disbursement_date`, `cancelled_date`, `source_of_funds`, `last_synced_at`,
  `accepted_at`/`rejected_at`, `order_placer_id`, closing address fields +
  `closing_property_uuid`.
- Status vocabulary is Qualia's, in **CAPS**: `active` = `PRE_OPEN · OPEN · "" · nil`;
  `syncable` adds `ON_HOLD`; plus `CLOSED`, `CANCELLED`.
  🔴 **A blank/NULL status counts as active** — orders that never synced look open.
- `pending_orders` scope = `rejected_at IS NULL AND accepted_at IS NULL AND external_id
  IS NOT NULL` — the acceptance queue.

18 sub-models hang off `EscrowOrder`: appointments (+ attendees, places), charges (+
payees), contacts, organizations, disbursements, documents, information requests, loans,
logs, notifications, properties, receipts, settlement team members, signings (+ location,
notary, progression, raw data), statement lines, tasks.

Two Qualia interfaces: `QualiaPlatformInterface` (the API) and `QualiaConnectInterface`
(the partner/marketplace channel). ~20 Sidekiq workers under `qualia_worker/` do the
syncing: `sync_open_escrow_orders`, `sync_pending_escrow_orders`,
`sync_recently_closed_escrow_orders`, `sync_recently_cancelled_escrow_orders`,
`sync_escrow_orders_by_time_range`, `sync_escrow_order_documents`,
`sync_escrow_order_settlement_team`, `sync_escrow_order_tasks`, `sync_settlement_agencies`,
`audit_orders`, `guess_qualia_account`, `send_purchase_contract_to_qualia`,
`trigger_notary_scheduling`, `trigger_utilities_change_of_ownership_notification`.

> 🔑 **`guess_qualia_account` exists**, which tells you order↔account mapping is
> heuristic, not deterministic. `audit_orders` is the reconciliation job. Treat
> `escrow_orders` as eventually-consistent with Qualia and always check `last_synced_at`.

### EVA

`Eva::` controllers (`agents`, `escrow_leads`, `service_office_lookups`) plus
`forward_notification_to_eva` / `forward_task_completion_to_eva` workers. EVA is a
downstream consumer of Qualia notifications and task completions — HAPI acts as the relay.

### Order opening eligibility

`EscrowOrderService::OrderOpeningEligibility.eligible?(lead)` uses two opposite lists
depending on lead type: for `buyer`/`seller` leads an **allow**-list of stages, for every
other user_type an **exclude**-list keyed by user_type
(`Global::EscrowOrder::OpeningEligibility`). ⚠️ Inverted logic in one method — easy to
misread.

## Closing signings — notary scheduling and e-closing

`ClosingSigningService` covers the signing appointment, separate from the escrow order.

| Enum | Values |
| --- | --- |
| `ClosingSigningOrder::SIGNING_METHODS` | `full_eclosing_with_enote · full_eclosing · hybrid_with_enote · hybrid_without_enote · wet` |
| `ClosingSigningOrder::STATUSES` | `pending · open · in_progress · completed · cancelled · closed · rescheduled` |

- `ClosingSigning` is polymorphic on `attachable` **and** `sub_attachable`. For BBYS the
  attachable is the lead and the sub_attachable is a `BbysLoan`;
  `original_eu_closing_signing?` is true when `sub_attachable` is nil or its
  `loan_type == "original"` — i.e. **re-papered loans get their own signing**, driven by
  distinct task templates (`BbysScheduleClosingSigningTask` vs
  `BbysScheduleRepaperClosingTask` vs `BbysTriggerRepaperDocumentSigningTask`).
- 🔴 `ClosingSigning::INELIGIBLE_PARTNERS = ["orchard"]` — Orchard deals are routed to
  `schedule_ineligible_partner_signing`, a separate path. (The vault notes the team no
  longer works Orchard; **the exclusion is still live in code**, as it is in
  `EliteProgramLevel`.)
- Two vendor paths: **Snapdocs** (`snapdocs/` — order creation, notary lookup, webhooks,
  document download, client tutorial email, notary confirmation to client and to LO/agent,
  Slack posts on schedule/reschedule/email-status) and **SignatureSync** (`signature_sync/`
  — webhooks, scheduling interface, audit worker). `GetAllowedSigningMethod` picks between
  them / picks the method.
- Every scheduling outcome is posted to Slack (`post_signing_to_slack`,
  `post_reschedule_to_slack`, `post_email_status_to_slack`).

## Post-close: document processing and recording

`hlcs_postclose_document_processing/` is a 7-stage worker pipeline over
`PostcloseDocument` / `PostcloseDocumentProcessorResult` / `PostcloseProcessingResult`:

```
scrapper → splitter → qualifier → composite_builder → qc → recorder → upload
```

with `hlcs_document_processing_tools/`: `basic_recordable_builder`,
**`mud_recordable_builder`** (Texas Municipal Utility District notices), `pdf_creator`,
`run_quality_check`. This is automated county recording-package assembly with an
automated QC gate.

**Reconveyances** (`reconveyances`, `reconveyance_batches`) close the loop on paid-off
HomeLight loans: every column is annotated with its **MeridianLink** source field
(`sLNm`, `custLoan361` "HLHL Paid Date", `custLoan2/8/14/20`, `sFinalLAmt`, `sRecordedD`,
`sRecordingInstrumentNum`, `sSpAddr`…). Batches are emailed to **TSI** (`email_sent_at`
comment: "Email sent to TSI at"). 🔑 This is the BBYS equity-unlock loan being reconveyed
after payoff — a title operation driven off LOS data, batched and outsourced.

## Client-facing tasks: information requests

`escrow_order_information_requests` + `information_requests/` interactors pull Qualia
information requests and turn them into **client tasks**:
`client_task_json/personal_info`, `client_task_json/confirm_legal_and_prior_names`,
`complete_client_task`, `dismiss_client_task`, `ensure_task_belongs_to_current_user`,
`organize_client_task_fulfillment`. Documents flow the other way via
`document_uploads/` (`process_escrow_order_document_uploads`, `sync_order_after_upload`,
`upload_documents_for_signing`, `upload_recordable_item_to_qualia`, `upload_to_sync`).

Also in-engine: `seller_net_estimates_controller` (seller net sheets),
`agent_placed_order_email_filter`, `place_qualia_order_filter`,
`create_closing_services_user_events`, `clean_up_escrow_order_links`, HOA-docs webhook
ingress (see [[repo-hapi-platform]] for Rexera).

## DisbursementService — paying money out

Not escrow disbursement (Qualia does that); this is **HomeLight paying LOs, brokers, and
partners**, via **Trolley** (`TrolleyClient`).

| Concept | Values |
| --- | --- |
| `Payment::STATUSES` | `created · not_eligible · eligible · pending · offline_pending · open · paid · returned · needs_internal_review · failed · cancelled · return_cancelled` |
| `PAYMENT_SOURCES` | `manual · payment_plan · elite_program` |
| `PAYMENT_CATEGORIES` | `regular · ad_hoc · exception · referral` |

Models: `PaymentPlan` (+ `PaymentPlanOwner`, `PaymentPlanDisbursement`,
`LeadPaymentPlanCohort`), `Invoice`, `Batch`, `PaymentRecipient`, `PaymentDetail`,
`ReferralPayment`.

- `Payment#elegible_for_payout?` (sic) = plan eligible for processing **and** plan accepts
  this lead **and** `lead.reached_payout_stage?`.
- `cancel!` raises if the payment is already in Trolley unless `force: true`.
- Incentive emails are first-class interactors: `bbys_ir_closed_lo_payment_incentive_email`,
  `bbys_ir_closed_dr_in_escrow_lo_payment_incentive_email`,
  **`bbys_ir_escrow_tls_broker_payment_incentive_email`** ("TLS broker" — the
  `RecipientBroker` payee, see [[homelight-agent-marketplace]]).
- `LoanOfficer#payments_flow` returns `"trolley"` if any payment exists with that LO as
  recipient under the resolved plan, else `"homelight"` — **two parallel payout rails**.
- Webhooks: `webhook_actions/process_{batch,payment,recipient}_all_actions` with
  `WebhookIpVerification`.
- Ops reporting: `generate_ap_reports`, `filter_inactive_payees_and_notify_ap_to_pay`,
  `notify_lsm_payment_profile_report`, `review_and_approve_payments_email`,
  `scheduled_payments_summary_email`, `process_rollover_invoice_payments`,
  `pay_offline_check_payments`.

## TransactionService / TransactionDataService — a different "transaction"

🔴 **Naming trap.** These two engines have nothing to do with closings. `Transaction` is
an **MLS sale record** used to rank agents:

- `belongs_to :listing_agent` / `:selling_agent` (both `Agent`), `belongs_to :mls`, plus
  city/zip/neighborhood/county/state.
- `default_scope` is aggressive: `status = 'sold' AND needs_resolution = false AND
  selling_price > 5000 AND selling_price IS NOT NULL`. **Every query silently excludes
  unresolved and non-sold records unless unscoped.**
- Guardrails: `MIN_PRICE = 1_000`, `MAX_PRICE = 100_000_000`; `metrics_guardrails` also
  requires bedrooms, property_type, property_sub_type and a selling_date.
- `MANUAL_MLS = 51` — a sentinel MLS id for hand-entered transactions.
- Property-type normalization lists are messy by necessity: `SINGLE_FAMILY_PROPERTY_TYPES`
  includes `"Single+family+home"` (URL-encoded) among 7 spellings.
- `HiddenTransaction` hides a record from search; `visible` scope enforces it.
- `TransactionDataService` (74 files) handles CSV **import** (agent-submitted transactions,
  Zendesk notification loop, verification file URLs) and the Elasticsearch index.
  `agent_transaction_import_queue`, `TransactionImport` track submissions.

These feed `agent_*_metric` / `agent_*_rank` tables (city, county, zip, neighborhood, area)
which feed the matching algo — see [[homelight-agent-marketplace]].

## Open questions

- Which `ServiceOffice` rows are actually live, and in which states does HomeLight hold
  title/escrow licenses today? The adapters imply AZ, FL, IL, PA, TX + CA (Glendale), but
  that's code, not licensure. Needs a DB read.
- What is the split between HomeLight-owned offices and `ChicagoTitle`/`SnapNHD` volume?
- Snapdocs vs SignatureSync: is SignatureSync replacing Snapdocs, or are they regional?
  Both have live webhook controllers.
- `escrow_orders.status` blank/NULL counting as "active" — how many rows are in that
  state, and are they real open orders or sync failures?
- Who consumes EVA, and is it internal tooling or a vendor?
- Are the hardcoded processor emails in `TransactionTeam` still the right people
  (Rhyna Coloma, Pat Torres, Derek Lupien)? Cross-check [[team]].
- `ClosingSigning::INELIGIBLE_PARTNERS = ["orchard"]` — dead code now that Orchard is
  out, or still routing live deals?
