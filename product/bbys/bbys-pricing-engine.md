---
last_updated: 2026-08-29 (Loop-3 reconciliation, same day: §"the business policy behind
Exceptions" gets a 🔴 cross-cite — its "25% comp discount" reading conflicts 3× with
[[comp-plans]]'s coded `* 0.25` formula on the same field. See [[numbers-that-disagree]] row 33,
[[risk-register]] row 46.)
status: current
source: homelight/hapi @ c14079c76c
scope: BBYS pricing engine — templates, variable pricing, exceptions, fee storage
source: HomeLight-Vault/context/bbys-pricing-engine.md
imported: 2026-09-29
---

# BBYS pricing engine

How a BBYS fee is actually determined. **Pricing is data, not code** — templates and
schedules are database rows, versioned and priority-ordered.

Read with [[repo-hapi]], [[bbys-unit-economics]], [[bbys-overview]].

> ⚠️ The code references a spec at **`docs/bbys-pricing-engine.md`** in the HAPI repo
> (cited from `ResolvePricingTemplate`, `PricingEngineSnapshot`). **That file does not
> exist in the repo.** Either it was never committed or it was deleted. This vault file is
> currently the only written description.

## The two tables

### `pricing_templates`

| Column | Meaning |
| --- | --- |
| `key`, `name`, `version` | versioned; unique on `(key, version)`, and **only one active row per key** |
| `pricing_model` | `flat_fee` or `variable` |
| `pricing_basis` | `percentage` or `dollar_amount` |
| `base_fee_percent` / `base_fee_amount` | the starting fee |
| `maximum_fee_percent` / `maximum_fee_amount` | cap |
| `minimum_fee_type`, `minimum_fee_amount` | floor |
| `variable_pricing_schedule_id` | link to the day-ramp schedule |
| `contingency_removal_only` | **the CRO fork** — see resolution below |
| `eligible_states[]` | empty = all states |
| `eligible_partner_slugs[]` | empty = all partners |
| `elite_only` | restricts to elite LOs |
| `eligible_minimum_valuation` | minimum DR valuation |
| `priority` | **lower number wins** |
| `effective_from` / `effective_to`, `active` | time-boxing |
| `configuration` | jsonb escape hatch |

### `variable_pricing_schedules`

A **day-based fee ramp** keyed off days on market:

| Column | Meaning |
| --- | --- |
| `starting_day`, `starting_fee_percent` | fee before the ramp begins |
| `daily_increase_start_day` / `_end_day` | the ramp window |
| `daily_increase_percent` | per-day increment (precision 15, **scale 12**) |
| `maximum_fee_start_day`, `maximum_fee_percent` | the cap and when it applies |
| `applicable_states[]` | |

## 🔑 The seeded catalog

From `lib/tasks/bbys_seed_pricing_catalog.rake` (v1, idempotent).

### Variable pricing schedules

| Schedule | States | Day 0–30 | Ramp (days 31–89) | Day 90+ |
| --- | --- | --- | --- | --- |
| **`standard_vp`** | all | **1.50%** | **+0.015%/day** | **2.40%** |
| **`florida_vp`** | FL | **1.50%** | **+0.0233333%/day** | **2.90%** |

> This is the "day 30–90 mechanics" referenced in ops Slack. **The fee starts at 1.50% and
> only reaches the 2.4% / 2.9% headline rate at day 90.** A deal that closes fast pays
> materially less than the standard flat rate — that is the entire pitch for variable
> pricing.
>
> Sanity check: `1.50% + 59 days × 0.015% = 2.385%` ≈ the 2.40% cap. Florida:
> `1.50% + 59 × 0.0233333% = 2.877%` ≈ 2.90%. The ramp is engineered to land exactly on
> the cap at day 90.

### Pricing templates

| Key | Name | Model | Basis | Base | Min fee | Schedule | States | Partners | Min valuation | Priority | Active |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `partner_florida_vp` | Florida VP | variable | % | 1.50% (max 2.90%) | $9,000 | `florida_vp` | FL | VP partners | **$425,000** | **190** | ✅ |
| `partner_standard_vp` | Standard VP | variable | % | 1.50% (max 2.40%) | $9,000 | `standard_vp` | all | VP partners | **$425,000** | **200** | ✅ |
| `florida_bbys` | Florida BBYS | flat_fee | % | **2.90%** | $9,000 (type: "Buy Before You Sell Loan") | `florida_vp` | FL | all | — | 300 | ✅ |
| `standard_bbys` | Standard BBYS | flat_fee | % | **2.40%** | $9,000 | — | all | all | — | 400 | ✅ |
| `standard_cro` | Standard CRO | flat_fee | % | **1.00%** | **$5,000** | — | all | all | — | 400 | ✅ (CRO fork) |
| `standard_dti_drop` | Standard DTI Drop | flat_fee | **dollar_amount** | **$2,500** | $2,500 | — | all | all | — | 400 | ❌ **inactive** |

### 🔑 Variable-pricing partners

Sourced from the Flagsmith flag **`sales-app-partner-config.variable_pricing`**, snapshotted
into the seed:

**`apm` · `envoy` · `arbor-financial` · `nfm-lending` · `21st-century`**

> ⚠️ **Doc drift already:** the rake task's own description lists four partners
> (apm, envoy, arbor-financial, nfm-lending); the code array has **five** — `21st-century`
> was added without updating the docstring.
>
> 🔴 **Drift observed (2026-08-29 Slack sweep):** the live Flagsmith flag held only FOUR
> partners (`apm, envoy, arbor-financial, nfm-lending`) through at least 2026-08-13 — the
> seed's fifth (`21st-century`) was ahead of the flag, i.e. drift in the opposite direction
> of the warning below. Also: a 2026-07-29 backfill (`hapi#19718`) wrote `flat_fee` onto
> ~50 VP-eligible leads (tls, apm, nfm-lending, envoy, uwm) because the catalog then had a
> VP template only for `lennar`; ≥3 leads still carried bad agreements on 07-31, resolution
> unconfirmed. See projects/2026-08-29-slack-partners-6mo.md.
> ⚠️ **This is a snapshot, not a live read.** The seed comment says: *"Update this list (and
> re-seed) when partners are added/removed from the flag."* **Changing the Flagsmith flag
> does not change pricing eligibility** — someone must re-seed. If a partner is on the flag
> but not in `eligible_partner_slugs`, they will not get VP.

### 🔴 `standard_dti_drop` is seeded but INACTIVE

Verbatim from the seed: *"Future replacement for standard_bbys (flat $2,500). Keep inactive
until go-live."* and *"activate it and deactivate standard_bbys at go-live."*

This is **exactly** the change Joel described on 2026-08-11 (flat $2,500, 180-day program
period). **The template exists and is ready; it has not been switched on.** Until it is,
every $2,500 DTI Drop in the wild is a **manual per-deal exception** — which is precisely
why $2,500, $3,500 and $5,000 are all appearing simultaneously. See
[[2026-08-28-vault-refresh-slack-findings]].

> Also note the seed warns it *"does not remove previously seeded placeholder rows (e.g.
> `lennar_intro`)"* — those must be deactivated manually per environment. Taylor confirmed
> in Slack (8/06) that `lennar_intro` was removed from the backfill.

## Template resolution

`LeadDataService::Bbys::ResolvePricingTemplate`:

1. **If locked and not forced** → keep the fee's current template.
   **Lock condition:** `BbysLead.bbys_pricing_agreement_signed?` — i.e. **once the pricing
   agreement is signed (clear-to-fund onward), the template stops auto-re-resolving.**
2. Filter active templates by the **CRO fork** (`contingency_removal_only` must match the
   lead's flag). CRO and non-CRO templates never compete.
3. Filter by eligibility: effective dates · `elite_only` · partner slug · state ·
   `eligible_minimum_valuation`.
4. **Pick the lowest `priority`.**
5. Fall back to `standard_bbys` (or `standard_cro` on the CRO fork).

**Elite statuses that satisfy `elite_only`:** `elite` · `diamond` · `obsidian` — the same
three tiers as the leaderboard badges in [[repo-homes-fe]].

> 🔑 Resolution is **pure** — it does not stamp the fee or write snapshots. Callers
> (`bootstrap`, calculator, `recalc`) own the writes. So "which template applies" and
> "which template is recorded on the fee" can diverge if a write path is missed.

## Where the fee is stored: `bbys_lead_fees`

One row per lead (`has_one :bbys_lead_fee`).

**Fee amounts:**
`estimated_program_fee_percent` / `_amount` · `actual_program_fee_percent` / `_amount` ·
`loan_officer_agent_value_estimated_program_fee_amount` · `misc_fees_costs` /
`misc_fees_description`

**Resolution record:**
`pricing_model` · `pricing_basis` · `pricing_template_id` · `variable_pricing_schedule_id` ·
`template_minimum_fee_amount` · `effective_minimum_fee_amount` ·
`resolved_agent_lender_valuation` · `resolved_valuation_source`

**Exception record:**
`exception_applied` (bool) · `exception_type` · `exception_types` (jsonb) ·
`exception_approved_by` (string) · `exception_notes` (text)

> ⚠️ **Estimated and actual are separate columns with no enforced link.** Joel's open concern
> (2026-08-11): a pricing exception applied to the *estimated* fee does **not** automatically
> transfer to the *actual* fee, and actual fees are often never filled in at all. For flat-fee
> templates the two should move together; for variable pricing the real fee isn't knowable
> until the DR is in escrow and a day count exists. **This is unresolved.**

## Exceptions

`ApplyPricingException` / `ClearPricingException` write:
`exception_applied`, `exception_type(s)`, `exception_approved_by`, `exception_notes`,
plus `program_fee_discount` and `program_fee_surcharge` on the lead, and can override
`pricing_template_id`, `variable_pricing_schedule_id`, `pricing_model`, and
`template_minimum_fee_amount`.

**`NotifyPricingExceptionToDealChannel`** posts a before/after Slack message to the deal
channel on `apply` and `clear`, tagging the deal's **LOS and LRM** when Slack IDs resolve.
It **soft-fails — Slack errors never fail the fee write.**

Fee keys shown in the Slack diff: `estimated_program_fee_percent` / `_amount`,
`actual_program_fee_percent` / `_amount`, `effective_minimum_fee_amount`,
`program_fee_discount`, `program_fee_surcharge`.

> 🔑 `exception_approved_by` is a **free-text string**, not a user reference. Combined with
> soft-failing Slack notification, **the approval audit trail is partly unstructured and
> partly best-effort.** This matches ops reality — Ian Pardo asking in-channel *"was variable
> pricing even approved?"* ([[2026-08-28-vault-refresh-slack-findings]]).

### 2026-08-29 — the business policy behind Exceptions (draft, unratified)

`sops/2026-06-22-discount-exception-policy.md` (owner Nick Santulli/RevOps, DBD-53 Wave 4;
**status: draft, pending Jake Vogel + Nick Friedman sign-off** — the approval-magnitude
matrix and margin floor are explicitly `[DECISION NEEDED]` placeholders, never ratified as
of this pass) is the only written attempt to codify who is allowed to grant the
`ApplyPricingException` writes above. **Stated**, not verified in code:

- **`bbys_opportunity_source` drives the baseline discount, not an exception:** `Lender` →
  full comp/standard fee · `Agent` → **25% comp discount** (automatic, property-driven) ·
  `Client` (D2C) → **$0 comp** (automatic, property-driven). Only deviation *from* this
  baseline counts as a governed "exception."

  🔴 **2026-08-29 — this "25% comp discount" phrase conflicts with [[comp-plans]]:184's coded
  formula, 3× apart, and neither doc previously cited the other.** [[comp-plans]] §3/§5
  documents the same `bbys_opportunity_source = "Agent"` field as a `* 0.25` multiplier — read
  literally, the LSM is paid **a quarter** of headline bps. Read literally, *this* doc's "25%
  comp discount" means the LSM is paid **three quarters**. Two readings, 3× apart, on the same
  live comp field: one from an app-notes build spec (the coded formula), one from this draft,
  unratified SOP. Do not pick a winner here — neither source is code, and this SOP is itself
  unratified (see the `[DECISION NEEDED]` placeholders above). **Needs Jake Vogel + Nick
  Friedman to adjudicate.** See [[numbers-that-disagree]] row 33 and [[risk-register]] row 46.
- **UWM conflict split:** when an AE relationship and an existing LO assignment conflict,
  the AE relationship wins — **50/50 commission split on the first 3 conflicted deals**
  only. Owner: Jake / LSM. Described as "a key 2026 decision" but not otherwise dated or
  cited elsewhere in the vault.
- **Promotional bps discounts** are time-boxed (example given: 50 bps through ~Apr 14) and
  are supposed to carry an explicit start/end date and a tracked promo code/segment —
  owned by Marketing + Jake.
- **Today, Jake Vogel holds blanket one-off exception approval authority** with no
  delegation tiers. The draft policy's whole purpose is to push small/routine cases down to
  the LSM level and reserve Jake/Friedman for material ones — **not yet approved**, so as of
  this pass Jake remains the sole approver for every exception regardless of size.
- **No margin floor is set.** The policy names a floor "below which no deal may be
  discounted without GM sign-off" but leaves the dollar/percent value as `[DECISION NEEDED]`
  — i.e. **there is currently no enforced minimum-margin guardrail on discounting**, which is
  the exact revenue-leak risk the draft was written to close.
- **Logging is interim/manual until Wave 4 tooling ships:** deal ID, discount type/magnitude,
  reason, approver, and date, posted to the deal's Slack channel and a free-text deal note —
  matches the `exception_approved_by` free-text-string / soft-fail-Slack pattern already
  documented above, confirming the code's "partly unstructured" audit trail is a known,
  named gap the business side is actively trying to close, not just an implementation detail.

⚠️ Because this policy is unratified, do not cite the 25%/$0/50-50/UWM-3-deal figures as
settled company policy without checking whether Jake/Friedman have since signed off —
check `sops/2026-06-22-discount-exception-policy-onepager.md` for the ratification status.

## Snapshots

`PricingEngineSnapshot` is **STI on the shared `snapshots` table**
(`type`, `provider_lead_id`, `payload` jsonb, `description`, `uuid`) — the same table used by
`EconomicModelSnapshot`, `PcrSnapshot`, `PropertyConditionQuestionnaire`,
`ExpressApprovalSnapshot`, `LiabilityReviewSnapshot`.

Navigation IDs and point-in-time state live **inside the jsonb payload**, not as columns —
so querying snapshots requires `payload ->> 'bbys_lead_id'`, and there is no index-friendly
foreign key.

**`CHANGE_SOURCES`:** `system` · `template` · `manual_recalc` · `manual_save` · `exception` ·
`backfill` · **`dual_run`**

> `dual_run` indicates the engine was run **in parallel with the legacy path** during
> rollout — useful for identifying which snapshots are shadow results rather than applied
> pricing.

Surfaced in sales-app as `PricingEngineSnapshotTimeline` ([[repo-sales-app]]).

## The interactor set

`BootstrapPricingEngine` · `ResolvePricingTemplate` · `CalculatePricing` · `SavePricingFees` ·
`ApplyPricingEngine` · `RunPricingEngineRecalc` · `CreatePricingEngineSnapshot` ·
`ApplyPricingException` · `ClearPricingException` ·
`NotifyPricingExceptionToDealChannel` · `BackfillPricingEngine`

## Related: the economic model freeze

`bbys_leads.econ_model_frozen` is gated by a dedicated role.

**Runbook** (`docs/runbooks/econ_model_admin_role.txt`): only users with the
**`econ_model_admin`** role may freeze/unfreeze via
`PUT /lead-data-service/v2/bbys/:id/econ-model-frozen`. Everyone else sees status text only,
no controls. Assigned through the internal user/role admin flow.

## Open questions

- **When does `standard_dti_drop` go live**, and does `standard_bbys` get deactivated at the
  same moment? Both must happen together.
- Is `eligible_minimum_valuation = $425,000` intentional for VP only? Non-VP templates have
  no valuation floor.
- Who re-seeds the catalog when the Flagsmith VP partner list changes? Nothing automates it.
- Does the **2.25%** tier discussed in `#proj-dynamic-pricing-bbys` become a new template, or
  a modification of `standard_bbys`?
- 🔴 **Update 2026-08-29.** A separate, apparently unrelated fee-structure track is landing:
  `homelight/hapi#20147` "[sc-192861] D1: Add partner compensation columns to `BbysLeadFee`"
  (trwong, opened 2026-08-29) adds LO add-on amount/percent, borrower-paid plan total, and
  borrower-paid discount/surcharge fields — explicitly labeled "D1" of a staged plan (D2
  engine, D3 LO add-on, E2 ops discount/surcharge referenced as future PRs). Columns are
  **provisional pending product confirmation**. This does not answer the 2.25%-tier question
  above, but it is a second, concurrent change to how BBYS fees are computed and stored — the
  two efforts should be reconciled before either ships, and this doc will need a new section
  once D1–E2 land. Source: [[../projects/2026-08-29-open-pr-roadmap]].
- `florida_bbys` is `flat_fee` but still carries `variable_pricing_schedule: florida_vp` —
  is that schedule ever consulted on a flat-fee template, or is it vestigial?
