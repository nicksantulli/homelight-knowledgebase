---
last_updated: 2026-08-28
status: current
source: homelight/hapi @ c14079c76c
scope: the BBYS buy box — service states, automated eligibility checks, risk adjustment
source: HomeLight-Vault/context/bbys-buy-box-and-eligibility.md
imported: 2026-09-29
---

# BBYS buy box and eligibility

The actual, code-level buy box: which states, which properties, which automated checks run
per approval tier, and how the risk-adjusted percentage is derived.

Read with [[bbys-overview]], [[bbys-lifecycle-operations]],
[[bbys-priority-partners-settings]], [[repo-hapi]].

> Two authoritative sources exist and they are **different in kind**:
> - **`config/bbys_deal_failure_reasons.yml`** — the *decline vocabulary* ops picks from
>   ([[repo-hapi]])
> - **`PropertyService::Eligibility::Constants` + `InstantApproval::EligibilityHelper`** —
>   the *automated gates* the system evaluates (this file)
>
> They overlap but do not match. Use the YAML for "why was it declined," use this for
> "would it auto-approve."

## 🔴 There are at least FOUR state lists, and they disagree

| Source | Count | Purpose |
| --- | --- | --- |
| **`MarketplaceProgram`** (`marketplace_program_states.enabled`) | database, per program | **the runtime gate** for `bbys` and `heloc` |
| `Eligibility::Constants::BBYS_SERVICE_STATES` | **39** | `service_state_check` |
| `Eligibility::Constants::BBYS_INSTANT_APPROVAL_SERVICE_STATES` | **37** | instant-approval `service_state_check` |
| `InstantApproval::EligibilityHelper::ELIGIBLE_STATES` | **32** | a separate eligibility path |

### `BBYS_SERVICE_STATES` (39)

`AK · AL · AR · AZ · CA · CO · CT · DC · DE · FL · GA · IA · ID · IN · KS · KY · LA · MD ·
ME · MI · MN · MS · MT · NC · NE · NJ · OH · OK · OR · PA · SC · SD · TN · TX · WA · WI ·
WV · WY · HI`

`BBYS_INSTANT_APPROVAL_SERVICE_STATES` is the same list **minus AK and HI** (37) — Alaska
and Hawaii are served but never instant-approved.

**Not in `BBYS_SERVICE_STATES` (12):**
`IL · MA · MO · ND · NH · NM · NV · NY · RI · UT · VA · VT`

### `ELIGIBLE_STATES` (32) — a different list again

`AL AZ AR CA CO CT DC GA ID IL IN IA KS KY LA ME MD MI MN MS NJ NC OH OK OR PA SC TN TX
WA WI WY`

> ⚠️ **`ELIGIBLE_STATES` includes `IL`, which is absent from `BBYS_SERVICE_STATES`**, and
> **omits `FL`, `DE`, `MT`, `NE`, `SD`, `WV`**, which are present there. These two lists
> cannot both be the buy box.

### 🔴 Discrepancy to verify: Virginia

`VA` is **not** in `BBYS_SERVICE_STATES` or `ELIGIBLE_STATES` — yet **live VA deals exist**
in Slack deal channels (`#go-rascal-appr-cook-…-va-express`,
`#nfm-lending-iruc-pugin-…-va-express`, week of 2026-08-25).

Possible explanations, none confirmed: the constants are stale; `service_state_check` is a
warning rather than a hard denial on the express path; or `MarketplaceProgram` is the real
gate and these constants only govern the automated tiers. **Worth confirming with
engineering** — if the constants are stale, automated approvals are silently unavailable in
states the business is actively serving.

### DTI-Drop-only states

```ruby
DTI_DROP_ONLY_STATE_CODES = %w[NY AK MA]
```

**New York · Alaska · Massachusetts** get **DTI Drop only** — no equity unlock. This
confirms the Slack note *"New York: DTI Drop offered, but no 0% bridge."*

Note MA also appears in `BbysLead::EXTENSION_EMAIL_REGULATORY_BLOCKED_STATES` — Massachusetts
is regulatorily constrained on two axes.

### States excluded from rapid approval

```ruby
RAPID_EXCLUDED_STATES = %w[FL NY AK MA HI WY MT]
```

Florida, New York, Alaska, Massachusetts, Hawaii, Wyoming, Montana.

## Economic constants

| Constant | Value |
| --- | --- |
| `MAX_LOW_LOAN_PAYOFF_VALUE` | **$1,900,000** |
| `MAX_HIGH_LOAN_PAYOFF_VALUE` | **$2,000,000** |
| `BBYS_FEE` | **2.4%** |
| `FLORIDA_BBYS_FEE` | **2.9%** |
| **`SELLING_COST_PERCENTAGE`** | **8%** |
| `ESTIMATED_PROPERTY_VALUE` | **$350,000 – $1,500,000** |

> 🔴 **Selling costs are 8% here but 6% in the `homes-fe` calculator**
> (`CALCULATION_DEFAULTS.sellingCostsPct = 0.06`, [[repo-homes-fe]]). The LO-facing
> calculator is therefore **more optimistic** about net proceeds than HAPI's own eligibility
> math, by 2 percentage points of the DR value — roughly **$14,000 on a $700k home**.
> This is a real, quantifiable inconsistency between what an LO quotes and what the system
> underwrites.

> The property-value band `$350k – $1.5M` is the automated band. Note the VP pricing
> templates separately require `eligible_minimum_valuation = $425,000`
> ([[bbys-pricing-engine]]) — **a property between $350k and $425k can be eligible but
> cannot get variable pricing.**

## The automated eligibility checks

Two entry points, both versioned event schemas:

| Event | Version | Notes |
| --- | --- | --- |
| `bbys_eligibility_check` | **2.4.0** | `preliminary: false` = **pre-lead** check; `true` = **post-lead** check |
| `automated_bbys_eligibility_check` | **2.9.5** | splits `denial_checks_result` vs `warning_checks_result` |
| `sales_bbys_eligibility_check` | — | sales-app initiated |
| `bbys_coastal_check` | 1.0.0 | Florida coast rule, records `distance_mi` + lat/long |

> 🔑 **Denials and warnings are separate result sets.** A property can pass the denial gate
> while accumulating warnings. Reporting that only reads `result` misses the warning tier.

### The named checks

`listing_period_check` · `sq_footage_check` · `property_type_check` · `lot_size_check` ·
`room_count_check` · `zoning_description_check` · `property_value_check` ·
`agent_valuation_check` · `ltv_check` · `service_state_check`

Property data captured alongside: `uuid` (+ `uuid_source`), `listing_mls_status`,
`listing_listing_contract_date`, `living_area` / `listing_living_area`, `property_type` /
`listing_property_sub_type`, `lot_size_acres` / `lot_size_sqft`, bedroom/bathroom counts,
`zoning_description`, `property_value_mean`, `ltv_mean`.

`automated_bbys_eligibility_check` also carries lender identity
(`lender_name`/`_email`/`_phone`/`_company`), `liens_present`, `agent_valuation`,
`agent_estimated_loan_payoff`, `target_unlock_amount`, and `source` / `source_form`.

Feature flag: **`automated-eligibility-improvements`** switches between
`LegacyEligibilityChecks` and `ImprovedEligibilityChecks` — **both paths are live**.

### Property type

```ruby
ALLOWED_PROPERTY_TYPES = ["Single Family Residence / Townhouse"]
ALLOWED_PROPERTY_LISTED_STATUS = ["Active", "Pending", "Active Under Contract", "Contingent"]
```

`ALLOWED_PROPERTY_SUB_TYPES` comes from `PropertyService::Engine.config.listing_properties_mapping`.

`HC_ACTIVE_PROPERTY_LISTING` (HouseCanary statuses treated as active): `Active` · `Coming` ·
`Soon` · `Pending` · `Under Contract` · `In Escrow` · `Accepting Backups`

> ⚠️ `"Coming"` and `"Soon"` are **separate array entries** — almost certainly a split of
> `"Coming Soon"` that was never rejoined. As written, a status of `"Coming Soon"` matches
> neither. Worth flagging.

## Checks by approval tier

### Instant approval
`state_check` · `user_value_to_property_value_mean_check` ·
`instant_approval_property_value_check` · `equity_unlock_threshold_check`

Ratio band: **0.75 – 1.25** (`ESTIMATED_VALUE_TO_PROPERTY_VALUE_MEAN_INSTANT`)

### Risk-on
`listings_under_contract_percent_check` · `sale_to_list_price_original_median_check` ·
`months_of_supply_median_check` · `risk_on_instant_approval_property_value_check`

Market thresholds: listings-under-contract ≥ **35%** · sale-to-list median ≥ **99.0** ·
days-to-contract median ≤ **60** · value-to-closed-price ratio ≤ **1.5** · **FSD ≤ 0.15**

### Light approval
`user_value_to_property_value_mean_check` · `requested_cltv_check`

Ratio ceiling: **1.10**

### Rapid approval (13 checks)
`rapid_state_check` · `rapid_photos_uploaded_check` ·
`rapid_user_value_to_property_value_mean_check` · `rapid_target_equity_unlock_check` ·
`rapid_year_built_check` · `rapid_property_type_check` · `rapid_lot_size_check` ·
`rapid_hc_avm_available_check` · `rapid_square_feet_check` · `rapid_cumulative_dom_check` ·
`rapid_recent_delisting_check` · `rapid_hc_data_present_check` · `rapid_basement_check`

| Threshold | Value |
| --- | --- |
| Ratio ceiling | **1.15** |
| `RAPID_TARGET_EQUITY_UNLOCK_MAX` | **$500,000** |
| `RAPID_SQUARE_FEET` | **1,000 – 3,000 sq ft** |
| **`RAPID_YEAR_BUILT`** | **1950 – 2021** |
| `RAPID_LOT_SIZE_MAX_ACRES` | **1 acre** |
| `RAPID_CUMULATIVE_DOM_MAX` | **90 days** |
| `RAPID_RECENT_DELISTING_DAYS` | **180 days** |
| `RAPID_LISTING_PHOTOS_MAX_AGE_YEARS` | **5 years** |

> 🔴 **`RAPID_YEAR_BUILT` tops out at 2021.** Any home built 2022 or later **fails rapid
> approval on year built alone**. Given new-build volume and the builder channel, this looks
> like a stale constant rather than a policy — new construction is exactly the segment the
> builder channel targets. Worth raising.

> ⚠️ Rapid's lot-size ceiling is **1 acre**, versus the 5-acre limit in the decline
> vocabulary. A 2-acre property is *eligible for BBYS* but *never rapid-approved*. Both are
> correct; they answer different questions.

### Two rapid checks are deliberately soft

```ruby
RAPID_APPROVAL_DISABLED_CHECKS = %i[rapid_photos_uploaded_check]
RAPID_ORCHARD_BYPASSED_CHECKS  = %i[rapid_target_equity_unlock_check]
```

- **`rapid_photos_uploaded_check` is evaluated and logged but excluded from the success
  determination.** The comment says *"Empty this list to re-enable a check as a hard gate."*
- **Orchard leads bypass the target-equity gate.** In-code rationale: Orchard leads *"arrive
  without a target equity unlock, so the dollar gate can never pass. Per stakeholder guidance
  we keep evaluating and logging them but exclude the target-equity-unlock gate … so we can
  measure how many Orchard leads qualify on property characteristics alone."*

> 🔑 **Rapid-approval pass rates are not comparable across partners** — Orchard is scored on
> a 12-check basis, everyone else on 13 (12 effective, given photos is soft). If Orchard
> rapid rates look better, that is the bypass, not the book.

Rapid approval is gated by flag **`bbys-rapid-approvals`**, with a dedicated diagnosis
logger (`[BBYS_RAPID_APPROVAL]` log lines, `rapid_diagnosis_check_lines`) — useful for
debugging why a specific property failed.

## 🔑 Risk-adjusted percentage (the RAP tables)

`risk_adjustment_percentage` on `bbys_leads` comes from a lookup keyed on the
**estimated-value-to-property-value-mean ratio**. Three tables exist:

### `RISK_ADJUSTMENT_PERCENTAGES` (the base table)

| Value ratio | RAP |
| --- | --- |
| 0 – 0.75 | 0.60 |
| 0.76 – 0.80 | 0.70 |
| 0.81 – 0.84 | 0.75 |
| 0.85 – 0.93 | 0.80 |
| 0.94 – 0.97 | 0.84 |
| 0.98 – 1.00 | **0.85** ← peak |
| 1.01 | 0.84 |
| 1.02 | 0.83 |
| 1.03 | 0.82 |
| 1.04 | 0.81 |
| 1.05 | 0.80 |
| 1.06 | 0.78 |
| 1.07 | 0.76 |
| 1.08 – 1.10 | 0.70 |
| 1.11 – 1.15 | 0.65 |
| 1.16 – 1.25 | 0.60 |
| 1.26 – 1.49 | 0.50 |

### `RISK_ON_RAP` (marked "V 1.3") — more generous

| Ratio | RAP |
| --- | --- |
| 0.75 – 0.95 | 0.83 |
| 0.96 – 1.00 | 0.82 |
| 1.01 – 1.05 | 0.80 |
| 1.06 – 1.08 | 0.77 |
| 1.09 – 1.13 | 0.75 |
| 1.14 – 1.20 | 0.75 |
| 1.21 – 1.25 | 0.73 |

### `RISK_OFF_RAP` — more conservative

| Ratio | RAP |
| --- | --- |
| 0.75 – 1.08 | 0.75 (flat) |
| 1.09 – 1.13 | 0.73 |
| 1.14 – 1.20 | 0.72 |
| 1.21 – 1.25 | 0.70 |

> 🔑 **RAP peaks at 0.85 when the agent's value roughly matches the AVM (ratio 0.98–1.00),
> and falls off in both directions.** Over-valuing the home is penalised faster than
> under-valuing it: ratio 1.25 gives 0.60, ratio 0.75 gives 0.60 — symmetric at the extremes,
> but the drop is much steeper above 1.05.
>
> This is the single most important number in BBYS unit economics — it sets
> `risk_adjusted_value`, which drives the guaranteed price. See [[bbys-unit-economics]].

> ⚠️ **Risk-on vs risk-off is a market posture switch that changes every deal's RAP by up to
> 8 points.** Establish which table is active before modelling anything. `RISK_ON_RAP` is
> versioned in a comment (`V 1.3`); the base table is not versioned at all.
>
> ⚠️ Note the ranges use Ruby float ranges (`0.76..0.80`) — **a ratio of 0.755 or 1.005 falls
> in no bucket.** Whether that returns nil or a default is worth checking.

> 🔴 **2026-08-29 — answered, and worse than the question implied.** Verified in
> `property_service/.../instant_approval/eligibility_helper.rb:44–89,305`
> (`working/2026-08-29-critic-technical.md` §2.5). The gap isn't limited to boundary values
> like 0.755/1.005: the **0.94–1.07 keys in the peak band are matched by exact float equality**
> (`elsif key.is_a?(Float) && key == value`), not by range membership — so the entire peak
> band silently misses on **any** ratio that isn't one of those exact literal values, not just
> the ones sitting between listed ranges. On any miss, the lookup **falls through and returns
> `nil`** (`:305`) — `risk_adjustment_percentage` is then `nil`, which propagates into
> `risk_adjusted_value` and leaves the guaranteed price undefined for that lead. This is a
> materially higher-frequency failure mode than the original open question suggested.

## 2026-08-29 — DR Underwriting Buy Box (business SOP, cross-checked against code)

Vault archaeology pass promoted these from `sops/2026-07-01-bbys-dr-underwriting-buy-box-source-of-truth.md`
(owner Jason Smith, Director of Real Estate Operations; approval DRIs Jason Smith + Nick
Friedman) and `sops/2026-07-01-florida-bbys-full-reopening.md` — the human-authored
underwriting policy layer that sits on top of the code constants above. **Stated**, not
observed in code, unless noted.

- **Chicago, IL: 70% CLTV limit** (HL internal valuation) — a geographic special case
  alongside Florida, not previously documented here. Same numeric ceiling as `base-unlock-70`
  below.
- **Florida re-opened in full July 2026** after an Oct-2024 suspension for hurricane/cat-risk
  losses, via a phased rollout (Orlando/Jacksonville Jan 2025 → Central/East Apr 2025 → Tampa
  Jan 2026 → full state Jul 2026). County-level CLTV split — **75% inland / 70% coastal**
  counties (Equity Unlock only; Asset Equity Boost stays 85%, HELOC Boost stays 90% statewide)
  — full 1,495-ZIP reference at `exports/florida-zip-county-cltv-reference-july-2026.csv`.
  Florida-only property rules: single-family detached ONLY (no condos/townhomes), must be
  **>2 miles** from Gulf/Atlantic coast, no FEMA Special Flood Hazard Area, no known active
  sinkhole areas, replacement-cost insurance required, and every HOA property gets a CC&Rs/
  bylaws review before approval. `FLORIDA_BBYS_FEE = 2.9%` (matches the code constant above).
- **DR property eligibility detail not in the code excerpt above:** 750–5,500 SF above grade
  (outside → Director of RE Ops exception), lot ≤5 acres, condo buildings <5 stories (5+ needs
  exception), HOA fees <1% of property value (over needs exception), min credit score **620**,
  vacant of tenants from agreement-signed onward, listed 150+ days in the trailing 365 (incl.
  under contract/contingent/pending) → **ineligible**, FSBO → **ineligible**. Full ineligible
  list (log/A-frame/dome/"Barndominum", co-op, 55+ community, tenancy-in-common, mixed use,
  active new-construction community, mobile/manufactured/modular, active foreclosure/short-
  sale/bankruptcy, HOA in litigation, <$150k value, vacant land) in the SOP.
- **Second mortgages / HELOC balances must be paid off with Equity Unlock proceeds.** Solar
  liens must be fully paid; a solar *lease* need not be paid off but requires a **min $10k
  holdback** from EU in the econ model.
- **Max Loan Payoff Value $2M** — matches `MAX_HIGH_LOAN_PAYOFF_VALUE` above. Anything over
  requires President-of-HLH + Director-of-RE-Ops sign-off before conditional approval.

### 🔴 Contradiction: Equity Unlock max CLTV — SOP says 85%, code says 70%

The DR Underwriting SOP's table states **"Equity Unlock (EU) — HL valuation: Max CLTV 85%."**
Every code-derived source in this vault — this file's own product-ceiling table (via
[[bbys-overview]] and [[repo-homes-fe]]), [[bi-metrics-definitions]], [[glossary]] — instead
gives the **base/default** Equity Unlock product (`base-unlock-70`) a **70%** ceiling, with
85% reserved for the `equity-boost-85` product (Asset Equity Boost) and 90% for `heloc-90`.
The SOP's own "Chicago: 70% CLTV" line and the Florida-summary doc both independently cite
70% as a below-baseline special case — consistent with 70% being the true EU default, which
makes the SOP's headline 85% figure look like a copy/transcription error (conflating EU with
Equity Boost) rather than a real policy change. **Not resolved here** — flagging per house
style rather than silently trusting either side. Confirm with Jason Smith / product before
quoting an EU CLTV ceiling in a partner-facing or comp-affecting context.

## 2026-08-29 — from the Data Bridge knowledge base (KB, source: `knowledge_documents`)

New ops-answer facts from the compiled Data Bridge KB (never previously cross-checked against
this doc). See [[data-bridge-kb-index]] for the full corpus. All observed as **stated** (compiled
KB answers, not code) unless noted.

- 🔑 **Credit score: the LOWER of two borrowers' scores controls**, not the higher — both
  generally need the 620 floor cited above. A frequent LO miscommunication is quoting the higher
  score as controlling; near-threshold files may get an operator-requested exception.
  ("Credit Score Eligibility: Multiple Borrowers and the Lower Score Rule")
- 🔑 **Private wells need two separate items**, often outside the standard Inspectify link and a
  borrower/LO out-of-pocket cost: (1) well pump/equipment review, (2) water-quality testing
  (E. coli, nitrates, plus any local-law contaminants). Proof that testing was *ordered* can
  support moving the well condition to a pre-HL-purchase path while results are pending.
  ("BBYS Operations: Private Wells, Trust Title, and Final Agreement Rounding")
- ⚠️ **Existing homestead designation on the departing home is not a hard BBYS stop**, but
  HomeLight documents may require representing the DH as a second home / IR as primary —
  which **can forfeit the client's homestead tax benefit** on the departing property. Must be
  disclosed to the client before signing. ("BBYS Occupancy & Homestead Designation Guidance")
- 🔑 **Non-warrantable condo = hard stop** — don't chase an HOA questionnaire (multi-week
  turnaround) once non-warrantability is confirmed. For a **warrantable** condo with a prior
  lender denial, get the denial letter/reason *first*, before ordering the questionnaire — the
  denial may reveal an independent disqualifier or clarify whether a BBYS path still exists.
  ("HOA/Condo Denial Triage: Eligibility Blockers vs. Information Gathering")
- **Titleholder mismatch workaround:** if the departing-home titleholder won't be on the new
  purchase loan, two case-by-case paths exist (route through title/legal): add the titleholder to
  new-home title, or QCD the purchase-borrower onto the departing home's title.
  ("BBYS: Titleholder Alignment Between Departing and Incoming Home")
- **Credit-block workaround:** if a current DH titleholder fails credit, a qualifying
  individual (spouse/family/co-borrower) can be added via quitclaim deed — a narrow, title-based
  fix, not a blanket eligibility waiver. Same file also covers a separate no-trustee-removal
  pattern: a QCD can add non-trustee family members *alongside* a trust's trustee without pulling
  the property out of the trust. ("BBYS Credit Block Workaround…"; "BBYS Operations: Private
  Wells, Trust Title, and Final Agreement Rounding")
- HELOC Equity Boost is **unavailable in Hawaii**, but Asset Equity Boost still proceeds via
  asset-document review — the portal may wrongly auto-generate a HELOC task on HI files; treat
  it as a known edge case, not a real blocker. ("Hawaii Equity Boost…")
- A first-lien amount over the **standard** conforming limit can still qualify for HELOC Equity
  Boost under the **county-specific high-balance Fannie Mae limit** — check the county limit
  before declining. ("HELOC Equity Boost: High-Balance Fannie Mae Limit Eligibility Guidance")

### 2026-08-29 addendum (LOOP-2 KB sweep — remaining ~290 title-only rows read via the compiler's
own `summary` column, not full content_text; see [[data-bridge-kb-index]] for method)

- 🔑 **Trust status is a hard eligibility branch:** a **revocable** departing-residence trust may
  proceed normally; an **irrevocable** DR trust is NOT proceedable unless title/legal resolves the
  ownership structure first. If the trust is being used for income/qualification purposes, request
  the trust document specific to the DR, not a generic trust doc. ("BBYS Trust Guidance: Revocable
  vs. Irrevocable Trusts for Departing Residence")
- 🔴 **Life estates are ineligible under the standard BBYS path.** An LO proposal to remove the
  life estate via quitclaim deed and re-title into a trust must be treated as a title/legal
  exception requiring specific trust-detail review — not a routine workaround. ("BBYS Ownership
  Eligibility: Life Estate Properties and Trust Workarounds")
- ⚠️ **New-build subdivisions: an active ZIP code is not enough.** Operators must confirm true
  arms-length resale comps exist (not builder sales or ZIP-level activity) before approving a DR
  in an active new-construction subdivision — frame thin evidence as a marketability/comps risk.
  High inventory-to-size ratio in the subdivision can itself be a saturation-risk denial, treated
  as adjacent to (not identical to) the existing ongoing-construction rule already in this doc's
  buy-box table. ("New-Build Buy-Box Guidance: Resale Comps and Departing Residence Approvals";
  "BBYS Buy-Box: New-Build Subdivision Saturation Denial")
- **Ladybird deed transfers with a deceased grantor** are not an automatic probate blocker if
  supported by recorded documentation of the death — may proceed for vesting/legal-description
  support; ambiguous or state-specific cases still route to title review. Complements the existing
  deceased-titleholder workflow: obtain the death certificate, confirm the deed, and — if the DR is
  still in probate — BBYS cannot proceed regardless of mortgage-payment status or a will's
  existence. ("Ladybird Deed Transfers…"; "Handling Deceased Title Holders and Probate Situations
  in BBYS")
- 🔴 **Active HOA litigation is an independent disqualifier**, even when every other HOA
  questionnaire answer (resale restrictions, corporate-title restrictions, special assessments) is
  clean. Don't let a clean questionnaire override confirmed litigation. ("HOA Litigation as a BBYS
  Eligibility Disqualifier")
- **Deed-vesting vs. 1003 marital-status mismatch** (deed shows single, 1003 shows married)
  requires a marriage certificate via Sales App plus government ID from the LO to confirm legal
  name/marital-status alignment before proceeding. ("Deed Vesting vs. 1003 Marital Status
  Mismatch – Resolution Process")
- **PCQ unavailable + high CLTV → evaluate max-light, but a prior severe-storm-area flag
  disqualifies from max-light** even if CLTV would otherwise allow it. Rural properties at ~5+
  acres get the same treatment — held for photos, routed to full express review, not max-light,
  regardless of other fast-track signals. ("PCQ Unavailability: High CLTV, Max Light Eligibility,
  and Severe Storm Area Flags"; "Rural Acreage Handling: Max-Light Path Exceptions")
- Manufactured/mobile/modular homes are ineligible **nationally**, not just under the
  Florida-specific buy box already in this doc — operators should decline these inquiries
  immediately rather than routing to standard approval review. ("HomeLight BBYS: Manufactured
  Homes Are Ineligible" — corroborates the existing Florida buy-box row, doesn't change it)
- Massachusetts' DTI-Drop-only status (already documented above via
  `DTI_DROP_ONLY_STATE_CODES`) is independently corroborated by a 2026-07-14 KB article framing it
  as "Standard BBYS now unavailable in MA, alongside AK and NY" — same fact, second source, no new
  information.

## Related

`bbys_65_days_eu_funding_extension_purchase` · `bbys_rapid_approval_evaluation` ·
`bbys_lead_routing_outcome` · `instant_equity_unlock_check` ·
`bbys_snapdocs_closing_order` (**SnapDocs** — a closing/notary vendor not previously in
[[tools-and-stack]]) · `bbys_client_soft_credit_check_frozen` ·
`bbys_heloc_1003_anomaly_detected` · `bbys_heloc_1003_lite_parser_completed`

Rake task: `update_bbys_eligible_zipcodes` — **there is also zip-code-level eligibility**,
maintained by a rake task. Not yet documented here.

## Open questions

1. **Which state list is authoritative?** Four exist. Reconcile against
   `marketplace_program_states`.
2. **Is VA actually served?** Live deals exist; the constants say no.
3. **Is `RAPID_YEAR_BUILT` ≤ 2021 intentional?** It excludes all new construction.
4. **8% vs 6% selling costs** — which is right, and should the calculator change?
5. **Which RAP table is live** — base, risk-on, or risk-off? Who flips it?
6. `"Coming"` / `"Soon"` as separate statuses — bug?
7. What populates `update_bbys_eligible_zipcodes`, and how often does it run?
