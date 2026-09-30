---
last_updated: 2026-08-29
status: current
source: homelight/hapi @ c14079c76c
scope: Equity Boost — the approval pipeline, data model, and why equity_boost=true is not intent
source: HomeLight-Vault/context/bbys-equity-boost.md
imported: 2026-09-29
---

# Equity Boost — system model

How Equity Boost is actually approved and recorded. The business/product view lives in
[[heloc-product]] (the Equity Boost umbrella covers **Asset EB** and **HELOC EB**); this is
the system view.

Read with [[bbys-buy-box-and-eligibility]], [[bbys-lifecycle-operations]], [[repo-hapi]].

## 🔴 The approval pipeline runs through a Google Sheet

From the controller's own documentation:

> *"This endpoint is called by an external automation owned by the approvals team: **a Google
> Sheet whose changes are relayed through Zapier / Make to HomeLight**. As approvers fill in
> the sheet (approved EB amounts, asset breakdowns, CLTV, or document requests), the
> automation POSTs here so the BbysLead and its EB tasks are kept in sync without manual
> data entry in sales-app."*

```
Approvals team fills Google Sheet
   → Zapier / Make
      → POST /lead-data-service/bbys/equity-boost-automations
         → BbysLead updated + tasks managed + Slack posted
```

### Authentication

Employee auth is **intentionally skipped** (`skip_before_action :authenticate_employees!`).
Instead: `Authorization: Bearer <token>` compared **constant-time** against
`ENV["EQUITY_BOOST_AUTOMATIONS_SECRET"]`. Missing/blank/mismatched → **403**.

### Workflows

| `workflow` param | Interactor | Effect |
| --- | --- | --- |
| **`data_update`** | `EquityBoost::ProcessAutoApproval` | approves EB + writes approved/asset amounts |
| **`document_request`** | `EquityBoost::ProcessDocumentRequest` | manages the `upload_equity_boost_documents` task |
| anything else | — | **422** |

Logging prefixes: `[EB_AUTOMATION]` (controller), `[EB_AUTO_APPROVAL]`, `[EB_DOC_REQUEST]`.

> 🔴 **A Google Sheet is a load-bearing production system for BBYS approvals.** Risks worth
> naming:
> - Spreadsheet edits write directly to production lead records with **no employee attribution**
>   — `whodunnit` on the resulting `versions` row will be system/NULL
>   ([[bbys-lifecycle-operations]]).
> - **Re-running overwrites**: *"Re-running re-approves and overwrites amounts with the
>   latest values; duplicate suppression for the endpoint is handled upstream, not here."*
>   A sheet re-sync silently restates approved amounts.
> - The Zapier/Make automation is **owned by the approvals team**, not engineering — it is
>   outside the repo, outside CI, and outside the deploy process.
> - Compare with the n8n "BBYS Applications" sync that keeps auto-deactivating
>   ([[2026-08-28-vault-refresh-slack-findings]]) — this is the same class of dependency.
>
> This belongs on the automation inventory. See [[data-bridge-automation-inventory]] and
> [[n8n-workflows]].

### What `data_update` writes

| Sheet field | HAPI column |
| --- | --- |
| `total_eb_approved` | **both** `equity_boost_amount` **and** `equity_boost_amount_used` |
| `checking_savings_cds` | `equity_boost_checking_savings_account` |
| `ira_brokerage_annuity` | `equity_boost_ira_brokerage_account` |
| `amount_401k` | `equity_boost401k` |
| `cltv_google_sheet` | **not stored** — only decides which Slack note posts |

Plus: `equity_boost = true` and `equity_boost_stage = "approved"`.

Currency inputs are sanitized by stripping `$` and `,`.

> ⚠️ **`total_eb_approved` populates two columns at once.** `equity_boost_amount` (approved)
> and `equity_boost_amount_used` (actually used) are set to the **same value** at approval —
> so they only diverge if something later updates `_used`. Do not treat "approved vs used"
> as a meaningful comparison without checking whether `_used` was ever independently written.

### Slack side effects

Posts a summary of applied values to the deal channel, plus **one** of:
- a **"prior to approval team review"** note when the lead is still in the **NEW** stage
  (i.e. approval based solely on submitted assets), or
- a **"Max EB based on CLTV"** note when `cltv_google_sheet >= 84.99`

Slack errors are rescued and logged — they never fail the interactor.

> `cltv_google_sheet >= 84.99` corresponds to the **`equity-boost-85`** product's 85% CLTV
> ceiling in [[repo-homes-fe]]. The threshold is 84.99, not 85 — a rounding guard.

### Failure modes

`422` when `lead_id` is blank or the BbysLead fails to save · `404` when the Lead or its
latest BbysLead can't be found.

> ⚠️ **"its latest BbysLead"** — the automation resolves to the *most recent* BBYS lead for
> that lead ID. A client with more than one BBYS lead could receive the approval on the wrong
> record.

## 🔑 `equity_boost = true` has at least three sources

This resolves the open question from Joel's 2026-05-21 observation
([[2026-08-28-vault-refresh-slack-findings]]).

| # | Path | Set by |
| --- | --- | --- |
| 1 | **Application flow opt-in** | LO or client in `homes-fe` (`equity_boost_opt_in` / `_opt_out` events) |
| 2 | **`ProcessAutoApproval`** | the **Google Sheet automation** — not a person in the UI |
| 3 | **`ApplyEquityBoostFromAgreement`** | sales-app "agreement section", recorded with `change_reasons: ["apply_from_agreement_section"]` and a lead note |

> 🔑 **`equity_boost = true` is not a reliable indicator of LO or client intent.** It is a
> state that three different actors can set, only one of which is an opt-in.
> **Any "Equity Boost adoption / opt-in rate" metric built on this boolean is measuring
> something else.** Use the `equity_boost_opt_in` / `equity_boost_opt_out` PostHog events
> from [[repo-homes-fe]] for intent, and `equity_boost_stage` for approval state.
>
> Joel's specific observation — that EB flips to true after lead creation when the target
> equity isn't met, despite opting out — is consistent with paths 2 and 3, though the exact
> trigger for his case was never confirmed in channel.

## 🔑 `equity_boost_stage` — its own stage vocabulary

> 🔴 **Correction 2026-08-29.** This section previously called `equity_boost_stage`
> "a **sixth** stage vocabulary" and listed **four** values. Both were wrong.
> Ordinal counting is retired — the full inventory now lives in
> [[stage-vocabularies-master]] (39 vocabularies; this is #16). And the enum has **five**
> values, stored as **title-cased labels**, not snake_case keys.

`hapi/app/models/bbys_lead.rb:283` is a Rails `enum` with string values, so
`bbys_leads.equity_boost_stage` holds the *value* side:

| Ruby key (what code sees) | Column value (what SQL / BI / HubSpot see) |
| --- | --- |
| `not_apply` | `N/A` |
| `waiting_for_application` | `Waiting for application` |
| `in_review` | `In-review ` ← **trailing space** |
| `approved` | `Approved` |
| `denied` | `Denied` |

> ⚠️ `WHERE equity_boost_stage = 'in_review'` returns **zero rows**. The correct
> predicate is `= 'In-review '`, trailing space included. `bbys_lead.rb:1041`
> (`equity_boost_stage.in?(%w[in_review approved denied])`) works only because Rails' attribute
> reader returns the key, not the stored value.

`not_apply` (rather than `not_applied` or `n/a`) is the "did not pursue" key.

## Data model

14 `equity_boost_*` columns on `bbys_leads`:

| Column | Notes |
| --- | --- |
| `equity_boost` | boolean — **see caveat above** |
| `equity_boost_stage` | 5-value enum stored as labels — see above and [[stage-vocabularies-master]]#4.6 |
| `equity_boost_amount` | approved |
| `equity_boost_amount_used` | used (set equal to approved at auto-approval) |
| `equity_boost_target_amount` | |
| `equity_boost_funded` | |
| **`equity_boost401k`** | 401(k) assets — note the column name has no underscore before `401k` |
| `equity_boost_checking_savings_account` | |
| `equity_boost_ira_brokerage_account` | |
| `equity_boost_account_types` | |
| `equity_boost_client_credit_score` | |
| `equity_boost_additional_client_credit_score` | |
| `equity_boost_exception_home_value` | |
| `combined_loan_to_value_ratio_equity_boost_exception_value` | one of the three CLTV columns |

`equity_boost_amount_used` is one of the 25 `ECONOMIC_INPUT_FIELDS` — EB feeds the economic
model directly.

**Credit bands** (`EQUITY_BOOST_CREDIT_SCORES`): `poor` = "Poor (lower than 620)" ·
`fair` = "Fair (620-679)" · `excellent` = "Excellent (680+)".

> ⚠️ These three bands are **coarser than HELOC's eleven** (`below_620` through `above_800`,
> [[heloc-product]]). The two products band credit differently, so EB and HELOC credit
> distributions are not directly comparable.

## Documents

`EQUITY_BOOST_DOCUMENTS_FOLDER_PREFIX = "EB Documents"` — the Google Drive folder prefix
(`GOOGLE_DRIVE_URL = "https://drive.google.com/drive/u/1/folders/"`,
`document_drive_folder_id` on the lead).

Interactors: `ProcessDocumentRequest` (manages `upload_equity_boost_documents`),
`AddAdditionalTitleHolderTask`, `ReviewEquityBoostApplicationNotification`.

`homes-fe` `equity-boost` feature: `schema.ts`, `asset-defaults.ts`, `step-constants.ts`,
`EquityBoostView`, `useEquityBoostLeadBootstrap`, `useEquityBoostAttachableTask`,
`api/upload-equity-docs`, `api/bbys-equity-lead`. Route `/client/equity-boost`.

Tasks: `bbys_client_equity_boost` · `bbys_heloc_eb_upload_task` (flag-gated —
`ApplyEquityBoostFromAgreement` readies a **legacy** apply task only when this flag is
**off**, so two task paths coexist).

Events: `bbys_equity_boost_application_submitted` · `bbys_equity_boost_notification` ·
`eb_asset_provided` · `equity_boost_opt_in` / `equity_boost_opt_out`.

Periscope: `equity_boost_approved` · `equity_boost_closed` · `equity_boost_fundings` ·
`equity_boost_ir_coe_fund` snippets, plus `equity_boost_product_tracker`,
`equity_boost_conversion_and_utilization`, `bbys_equity_boost_checks`,
`ad_hoc___finance_equity_boost` dashboards ([[repo-bi-periscope]]).

## Known gaps

- **Portal asset-upload access** — an LO with a client holding ~$800K in assets could not
  find the asset-statement upload option in the portal (2026-08-26); engineering was asked to
  reopen access. Unresolved.
- **Hawaii:** HELOC unavailable, but **Asset Equity Boost still proceeds** — a documented
  portal HELOC-task edge case.
- **Florida:** EB is one of the paths through `couldnt_qualify_for_eb` in the failure
  taxonomy ([[repo-hapi]]).

## Open questions

- Who owns the EB Google Sheet and the Zapier/Make automation? What happens if it breaks
  silently — is there any monitoring on `[EB_AUTOMATION]` log volume?
- Is `EQUITY_BOOST_AUTOMATIONS_SECRET` rotated? It is the only auth on a production write
  endpoint.
- Does anything ever write `equity_boost_amount_used` independently of approval? If not, the
  column is redundant.
- Which of the three `equity_boost = true` paths dominates in practice? That determines
  whether adoption metrics are salvageable retroactively.
