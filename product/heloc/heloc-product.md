---
last_updated: 2026-08-29 (+Nous archive-tail addendum — card/access-device mechanics,
secondary-market plan, and a broader compliance workstream (CA state exam, Continental Bank side
letter, TX 50(a)(6)) surfaced from a permission-restricted Nous collection, search-snippet only,
mid-May 2026 snapshot)
type: context
source: HELOC Playbook v1 (Wei Wang, 20260331) + Slack verification 2026-08-26 + Nous MCP
(read-only, search-snippet access only) 2026-08-29
source: HomeLight-Vault/context/heloc-product.md
imported: 2026-09-29
---

# HELOC / Equity Boost Product

> Comprehensive reference for the HELOC Equity Boost product. Source: HELOC Playbook v1.
>
> Fee/cost model lives in [[bbys-unit-economics]].

## Verified 2026-08-26 — what changed since the April playbook

### ✅ Standalone HELOC is now LIVE (this file previously said it was not)

The "not available at launch" note further down is **superseded**. As of ~June 2026:

- Live at **HomeLight.com/HELOC** — no BBYS transaction required
- **HomeLight Home Loans is the originator**
- **No 0% teaser period** on standalone (unlike the BBYS-attached version)
- **No origination fee and no annual fee** to the LO or client (confirmed by Marc Kaplan)
- An LO cannot originate their own HELOC, but HLHL can originate it for them

### 💰 HELOC economics — appears to be pure net interest margin

With no origination fee, no annual fee, and a 0% teaser for the first 6 months on BBYS-attached
lines — which is the window most BBYS HELOCs live in before the DR sells and the line returns to
$0 — **HomeLight earns effectively nothing on a BBYS-attached HELOC**.

**Working conclusion:** HELOC Boost is a *deal-enabler that costs money*, not a revenue line, and
should be modeled in the Channel P&L as a cost of winning the BBYS deal. ⚠️ Open question — see
[[bbys-unit-economics]] §"Open questions" #3.

### ✅ Rate sheet revalidated 2026-06-25

Marc Kaplan confirmed the full margin ladder below is still current. Two clarifications:
- **Margin is set by FICO only — not by CLTV.** A lower CLTV does not improve the rate.
- Estimated all-in variable rate in mid-2026 was running **~9.375%** (WSJ Prime + margin)

### ✅ Asset Equity Boost — 401(k) cap

401(k) usage is capped at the **lesser of 50% of the vested balance or $50,000**, and only if the
client is **actively employed**. Outstanding loans against the 401(k) can void eligibility
altogether. (Domonique Stubblefield, Aug 2026 — this cap is *in addition* to the 1.5x/2x credit
coverage ratios documented below.)

## Equity Boost Umbrella — Two Types

### Asset Equity Boost (Original)
- Secured against liquid financial assets: 401k, savings, IRA, brokerage, gift funds
- Credit 680+: 1.5x coverage. Credit 620-679: 2x coverage
- Caps at ~85% CLTV on DR
- Excludes: crypto, business assets, assets already allocated for purchase

### HELOC Equity Boost (Launched March 4, 2026)
- HELOC originated on the **Incoming Residence (IR)**, not DR
- Closes at **$0 drawn** simultaneously with purchase
- Pushes EU up to **90% CLTV** on DR (vs ~85% with assets)
- No liquidation penalties, no forfeited investment returns
- 0% teaser for 6 months, then variable (Prime + margin)
- Can combine BOTH asset EB and HELOC EB on same deal
- HELOC generally preferred: operationally simpler, avoids EPO/recast issues

## HELOC Pitch — Richie's Talk Track (v2)

> "Hey LO, as you know with regular BBYS we are going to end up somewhere between 70-75% CLTV. In your case, your borrower looks like they are going to need a little bit more than that."
>
> "What we can do is add a HELOC to the transaction, which will specifically go against the new house."
>
> "We are not going to draw on it upfront, you're not gonna have any LLPA hits to your rate, everything's going to be fine, but essentially we will use that HELOC to go up to 90% CLTV, essentially giving your client more money."
>
> "It's not going to cost anything different."
>
> "The best thing we can do for now is here is the list of docs that I'm going to need or list of items. Let's get them in, of course, nothing to lose, we will basically check to see how high we can actually go to make it work for your borrower."

### Buzz Words for Sales Cycle
- "This lets you unlock up to 90% CLTV."
- "No interest for six months."
- "Lower mortgage payment upfront."
- "No need to liquidate retirement funds."
- "Keeps liquidity available even after closing."
- "No early pay off penalties."

## How HELOC Funding Works (6 Steps)

| Step | What Happens | Why It Matters |
|------|-------------|----------------|
| 1 | HELOC established on IR at $0 drawn, closes with purchase | CLTV = LTV at close. No subordinate financing LLPA. No PIW impact. |
| 2 | HomeLight increases EU on DR — HELOC backs the difference | Client gets a larger Equity Unlock to cover down payment gap |
| 3 | Full EU amount (standard + boost) funds the purchase | Client closes on new home with needed funds |
| 4 | HELOC draws 7 days after IR close | Pays down the boosted portion of the BBYS lien on DR |
| 5 | Exposure transfers — DR carries only standard EU lien; boost sits on HELOC against IR | Risk moves from DR to IR |
| 6 | DR sells — proceeds repay mortgage + BBYS lien | If proceeds cover everything, HELOC goes to $0 and line stays open |

## HELOC Messaging

### For LOs to Position with Borrowers
Equity Boost is a powerful tool that allows you to have your biggest financial asset at your fingertips. Using BBYS with Equity Boost unlocks up to 90% CLTV on your existing home to use on the purchase of your next home.

With Equity Boost, equity is unlocked two ways:
- A HELOC on the incoming residence
- Proof of assets (savings, 401k, IRA, brokerage, gift funds)

Choose one or both. Get the mortgage and monthly payment you want now, without waiting for a refinance.

### Agent Messaging
With BBYS, HomeLight unlocks a portion of your client's equity in their existing home. Sometimes clients need more equity to make the deal work. Equity Boost helps clients reach their down payment goals or assist in listing prep at no extra cost. Unlocks up to 90% CLTV via HELOC on IR or proof of assets.

### Client Messaging
We believe you should have your biggest financial asset at your fingertips. Equity Boost unlocks up to 90% CLTV on your existing home. Two methods: HELOC on incoming residence, or proof of assets. Get the mortgage and payment you want now without waiting for refinance. Equity in hand even after moving in.

## Objection Handling

| Objection | Response |
|-----------|----------|
| "The HELOC closes at $0 — where do the funds come from?" | The HELOC backing allows HomeLight to increase the EU on the DR. That increased EU funds the purchase. 7 days after close, the HELOC draws and pays down the boosted portion of the BBYS lien — transferring exposure to the new home. |
| "Won't the HELOC hurt my client's purchase rate?" | No — closes at $0 drawn, so CLTV = LTV. Per Fannie Mae, only the drawn balance of a subordinate HELOC is included in CLTV. No subordinate financing LLPA applies. |
| "My client's DTI is already tight." | HELOC can help — use it to consolidate higher-rate credit cards or auto loans, potentially lowering monthly obligations. Qualified conservatively at fully drawn, post-teaser rate + 2%, on 20-year amortization — but actual payments are $0 for 6 months, then interest-only. |
| "What happens if the departing home sells short?" | Sale proceeds pay off mortgage first, then EU, then whatever's left pays down HELOC. Any remaining HELOC balance stays on the line under normal repayment terms — up to ~30 years. |
| "Is this available for FHA/VA?" | Currently conventional only (Fannie/Freddie). Not eligible in TX, NY, AK, MA, VA, VT, or IA. |

## Key Stats

| Talking Point | Detail |
|---------------|--------|
| Max CLTV on DR | Up to 90% (vs ~85% with assets) |
| Introductory rate (teaser) | 0% for 6 months |
| Purchase pricing impact | None (CLTV = LTV at close, $0 draw) |
| Min credit score | 620 |
| Max HCLTV on IR | 89.99% |
| Draw timing | 7 days after IR close |
| Term | 30 years (10-yr IO draw / 20-yr amortized) |
| Can combine with Asset Boost | Yes |

## Rate Sheet (Variable — Index + Margin)

Index = WSJ Prime Rate. Margin locked at origination. Rate adjusts monthly with Prime.

| Credit Score | Margin | Initial Variable Rate | Qualifying Rate (DTI) |
|---|---|---|---|
| 780+ | 2.625% | Prime + 2.625% | Initial + 2% |
| 760-779 | 2.625% | Prime + 2.625% | Initial + 2% |
| 740-759 | 2.875% | Prime + 2.875% | Initial + 2% |
| 720-739 | 3.125% | Prime + 3.125% | Initial + 2% |
| 700-719 | 3.500% | Prime + 3.500% | Initial + 2% |
| 680-699 | 3.750% | Prime + 3.750% | Initial + 2% |
| 660-679 | 4.500% | Prime + 4.500% | Initial + 2% |
| 640-659 | 5.125% | Prime + 5.125% | Initial + 2% |
| 620-639 | 5.625% | Prime + 5.625% | Initial + 2% |

Qualifying rate = Initial Variable Rate + 2%, on 20-year amortization assuming full draw. Actual payment: $0 for 6 months, then interest-only to year 10, then P&I for remaining 20 years.

## Underwriting & Qualification (as of 2/25/2026)

- **Min Credit Score**: 620 (lowest mid score; multiple borrowers = use lowest middle score)
- **DR CLTV**: 90% max
- **IR HCLTV**: 89.99% max (underwritten as if fully drawn)
- **DTI**: 45% max. Exception: 50% if FICO 740+ or 700-739 with >= $3,500 monthly residual income
- **Eligible loans**: Conventional (Fannie/Freddie) only
- **Not eligible**: VA, FHA, certain cash scenarios (unless fully underwritten)
- **HELOC borrowers must exactly mirror IR mortgage borrowers**

### HCLTV Calculation on IR
Formula: (First Mortgage + Full HELOC Credit Limit) / IR Value

Key nuance: because HELOC Boost is simultaneously part of the down payment (reducing the first mortgage) AND included as the HELOC credit limit, the effects cancel out. HCLTV effectively simplifies to:
- **(Purchase Price - Non-HELOC Down Payment) / IR Value**

**Increasing HELOC size does NOT change HCLTV** — it is neutral. HCLTV is driven entirely by how much the borrower puts down from non-HELOC sources.

Rule of thumb: **Standard EU + Borrower Own Funds >= ~10% of IR purchase price**

### If HCLTV Is the Binding Constraint
Since HELOC size doesn't affect HCLTV, the non-HELOC down payment is the only lever:
1. **Asset Boost** fills the gap — adds to down payment without creating a lien on IR, increases non-HELOC down payment, brings HCLTV into range
2. Once HCLTV threshold is cleared, **HELOC can be maximized up to DTI ceiling** — every dollar of HELOC reduces first mortgage dollar-for-dollar

Combined strategy:
- Asset Boost brings non-HELOC down payment to >= ~10% of IR value (satisfies HCLTV)
- HELOC Boost then maximized to DTI ceiling (drives first mortgage as low as possible)

### DTI Details

**HELOC payment for DTI**: Underwritten as fully drawn, at Initial Variable Rate + 2%, on 20-year amortization.

**LOS entry**: Enter qualifying HELOC payment in "Other Financing Information" from day one. If underwriting changes anything (income reduced, liabilities adjusted), the DTI impact must show up immediately. Notify HomeLight right away so HELOC can be resized if needed.

**HELOC funds CAN reduce DTI**: Proceeds can pay off credit cards, auto loans, other higher-rate debt.

**If borrower is at 49% DTI**, levers include:
- Pay off higher-rate debts with HELOC (free up DTI headroom)
- Reduce HELOC size (smaller qualifying payment)
- Lower first mortgage (larger down payment via Asset Boost)
- Buy down first mortgage rate
- LRMs: Use HELOC Optimizer in Sales App
- LOs: Model directly in LOS

## Document Requirements

### Asset Boost
- Most recent account statement for each qualifying asset, with 30 days of transaction history

### HELOC Boost — Prequalification
- 1003 (or prequalification information entered manually into portal)

### HELOC Boost — Final Approval (after IR under contract + first mortgage conditionally approved)
- Final 1008
- IDs + SSN cards for all borrowers
- Credit report
- Most recent mortgage statement for DR
- IR purchase contract
- Final CD for IR mortgage
- IR mortgage final approval
- IR Appraisal
- IR HOI (HomeLight to be added as mortgagee)
- Income documentation (paystubs, W2s, VVOE within 10 days of closing)
- HLHL may require additional docs (gap explanations, credit inquiries within 90 days, etc.)

## Process Flow

1. Submit through Lender Portal (standard BBYS application). No need to select a path — HomeLight determines which boost applies based on information provided.
2. HomeLight pre-qualifies borrower for HELOC Boost amount
3. Borrower goes under contract on new property
4. Purchase mortgage processed and receives clear to close
5. Full loan documentation submitted to HomeLight Home Loans
6. HELOC underwritten and approved (clear to close)
7. Purchase closes with HELOC established at $0 drawn
8. ~7 days after closing, HELOC draws approved Boost amount
9. When DR sells, proceeds repay remaining BBYS lien. Client may pay down or carry remaining HELOC balance (~30-year repayment)

## FAQ Quick Reference

### No Impact on Purchase
- **Purchase rate**: Not affected (CLTV = LTV at close, $0 draw, no subordinate financing LLPA)
- **Appraisal waiver**: Not affected (same reasoning)
- **Not a purchase-money HELOC**: Not drawn at closing, doesn't increase CLTV, avoids subordinate financing pricing hits

### HELOC Terms
- **Origination**: At IR closing, $0 draw
- **Draw timing**: 7 days after IR close (contractually controlled, future-dated draw agreement)
- **Payments**: 0% interest / no payments for 6 months, then interest-only begins
- **Term**: 30 years (10-yr IO draw period / 20-yr amortizing repayment)

### Sale Scenarios
- **Happy path (DR sells at full value)**: Mortgage paid off, EU paid off, HELOC paid off. Customer retains open HELOC at $0.
- **DR sells short**: Mortgage paid off, EU paid off, remaining proceeds pay down HELOC. Any remaining HELOC balance stays on IR, customer repays over time.
- **Past 120 days**: HELOC remains 0% for 6 months. After 6 months, interest begins. Extensions handled case-by-case.
- **Extension fee**: Applies to sale price (same as today). HELOC itself has no interest during first 6 months.

### Additional
- **Retroactive to existing files**: Possibly, case-by-case
- **HELOC can be closed after DR sells**: Yes, if outstanding balance is fully repaid
- **Standalone HELOC (no BBYS)**: ⚠️ **SUPERSEDED — now live.** See the 2026-08-26 verification section at the top of this file.
- **Full documentation required**: After first mortgage receives final approval. Cash buyers require full underwriting by HomeLight.
- **Title fees**: Possibly minor recording fees (TBD)
- **Survey required**: No
- **Appraisal on IR**: For financed purchases, first mortgage appraisal accepted. For all-cash, AVM may suffice (confirm case-by-case).
- **Servicer**: HomeLight Home Loans (actively working on subservicer partnership)

### State Eligibility
**Not available in**: Texas, New York, Alaska, Massachusetts, Virginia, Vermont, Iowa

Available in all other states where BBYS operates.

## When to Recommend HELOC vs Asset Boost

| Recommend HELOC When | Recommend Asset When |
|----------------------|---------------------|
| Client has sufficient IR equity | Client lacks IR equity (HCLTV > 89.99%) |
| Client wants to preserve retirement assets | Client has liquid assets to pledge |
| Client wants highest unlock possible (90% vs ~85%) | Client is in an ineligible HELOC state |
| FHA/VA not involved | Conventional not available |

Both can be combined on the same deal.

## Metrics (as of late March 2026)

| Metric | Current | Target |
|--------|---------|--------|
| Utilization rate | 7.05% | 30% by end of April |
| Prequal rate | 72% | ~85% |
| Avg prequal amount | $85,375 | -- |
| Deals in contract | 3 | -- |
| HELOC cards closed | 1 (employee test) | 100 before May 15 (Drew) |

## Edge Cases: When HELOC Does NOT Work

| Edge Case | Why |
|-----------|-----|
| Client putting exactly 10% down | Hits 90% CLTV cap -- every dollar of HELOC requires first mortgage to shrink equally |
| DTI already above 45-50% | HELOC qualifying payment pushes DTI over limit |
| FHA or VA loan on IR | HELOC only works with conventional (Fannie/Freddie) |
| Ineligible states | TX, NY, AK, MA, VA, VT, IA |
| Construction/renovation loan on IR | Property must be lendable under Fannie/Freddie guidelines |
| All-cash purchase >$250K with no appraisal | Lines over $250K need full appraisal (under $250K can use AVM) |
| DTI-drop leads | HELOC initially disabled; now fixed but LO must apply later from portal |

## Common LRM/LSM Confusion Points

1. **LRMs lack mortgage knowledge** -- LOs ask technical questions about IO payments, rate adjustments, amortization that LRMs can't answer
2. **Asset EB vs HELOC EB confusion** -- both under "Equity Boost" umbrella; LOs/LRMs conflate them
3. **Wrong document uploads** -- LOs upload asset statements where 1003 is required
4. **"Where do the funds come from?"** -- HELOC closes at $0, backing allows HL to increase EU on DR
5. **"Won't this hurt the purchase rate?"** -- No, CLTV = LTV at close because undrawn HELOC excluded

## HELOC Card (Next Phase)

- Standalone product -- does NOT require a BBYS transaction
- Marc Kaplan + Oliver building LO portal with HELOC card tab
- 75-80% advance rate initially, potentially 90%
- No interest until first missed full payment
- Drew target: close 100 before May 15
- Open legal question: whether card requires mortgage servicing license

## Open Legal Questions

- Mortgage servicing license for HELOC card (Aven has full servicing despite similar positioning)
- Purchase money classification -- Friedman pursuing with legal
- Right of rescission: 3-day rescission on HELOC (secured by primary residence)

## Related Notes

- [[bbys-unit-economics]]
- [[bbys-overview]]
- [[dti-drop]]
- [[q2-priorities]]

---

# System model (added 2026-08-28 from `homelight/hapi` @ `c14079c76c`)

The technical side of HELOC — how it is modelled, staged, and gated in code. Everything
above is the business/sales view; this is the system view.

## 🔑 The 8 HELOC stages — answering an open question

On 2026-08-21 Joel asked in `#homes-heloc-pod`: *"does anyone have a list of all the heloc
stages that are possible currently?"* **Nobody answered.** From `HelocLead::STAGES`:

| # | Stage | Label |
| --- | --- | --- |
| 0 | `new` | New |
| 1 | `in_review` | In Review |
| 2 | `needs_human_review` | Needs Human Review |
| 3 | `preliminary_prequalified` | Preliminary Prequalified |
| 4 | `prequalified` | Prequalified |
| 5 | `approved` | Approved |
| 6 | `denied` | Denied |
| 7 | `withdrawn` | Withdrawn |

Like BBYS stages, each is a **two-element array** (`["in_review", "In Review"]`) — both forms
are live.

> 🔑 **`STAGE_VALUES` gives HELOC a contiguous 0–7 numeric scale.** (Ordinal labelling
> retired 2026-08-29 — this was called "a fifth numeric scale"; the full inventory is
> **[[stage-vocabularies-master]]**, where HELOC is #11 of 39 and BI alone holds nine numeric
> scales.) ⚠️ HELOC's `new` / `in_review` / `approved` collide by name with BBYS's — same
> strings, different meaning, and both can land in `provider_leads.stage`. Unlike `new_stage_index`, it is contiguous and has no
> fractional entries — do not assume the BBYS index conventions apply here.

### Stage groupings

- **`STAGES_BEFORE_PREQUALIFIED`** — `new`, `in_review`, `needs_human_review`,
  `preliminary_prequalified`
- **`STAGES_THAT_ALLOW_PRELIMINARY_PREQUALIFICATION`** — `new`, `in_review`,
  `needs_human_review`, **and `denied`** ← a denied lead can still receive a preliminary
  prequalification
- **`TERMINAL_BBYS_DOCUMENT_COLLECTION_STAGES`** — `denied`, `prequalified`, `withdrawn`
  (document collection stops here)

### What moves `new` → `in_review`

`FIELDS_THAT_MOVE_NEW_TO_IN_REVIEW`: `estimated_home_value` ·
`target_down_payment_percentage` · `purchase_loan_interest_rate` ·
`hoa_association_dues_monthly` · `property_taxes_monthly` · `home_owner_insurance_monthly` ·
`mortgage_insurance_monthly` · `client_estimated_income_monthly` · `form_1003_uploaded` ·
`credit_report_uploaded` · `estimated_credit_score`

**Minimum to prequalify** (`PREQUALIFICATION_REQUIRED_FIELDS`): `external_id` ·
`estimated_home_value` · `target_down_payment_percentage` · `client_estimated_income_monthly`
— only four fields.

### Forced transitions

`force_prequalified_stage_transition` · `force_preliminary_prequalified_stage_transition` ·
`force_denied_stage_transition` · `skip_stage_transition` are `attr_accessor`s used to
override the state machine. Stage updates are recorded in `heloc_lead_stage_updates`.

## 🔑 Prequalification is an external API call

`HelocLead#prequalify!` does **not** compute anything locally. It calls
`LeadDataService::Clients::HomelightHomeloans` — **HomeLight Home Loans (HLHL) is the
underwriting authority**, and HAPI stores the answer.

Sequence:
1. Resolve the `bbys_lead` (`lead.order.bbys_providable`) — **exits early if there is no
   BBYS lead**, so HELOC prequal cannot happen standalone in this path
2. **State gate:** `MarketplaceProgram.enabled_for_state_id?("heloc", state.id)` — if the
   state is not enabled, set `denial_reasons` and return immediately
3. Ensure the BBYS deal folder exists (Google Drive)
4. Call HLHL `prequalifications`, capture `last_api_log`
5. Normalize the payload, decide preliminary vs full, set the forced stage transition
6. Write an economic model snapshot (when `heloc_funds_used` applies and it is not
   preliminary)
7. Sync the prequal letter file

**State-ineligible denial text** (fixed string):
> *"HomeLight's HELOC program is currently not available in this state."*

This is why **HELOC is unavailable in Hawaii** while Asset Equity Boost still proceeds —
the gate is per-state, per-program, in `marketplace_program_states`. See
[[bbys-priority-partners-settings]].

### The invariant

`pre_qualified_amount` and `denial_reasons` are **mutually exclusive** — setting either
clears the other:
- setting `pre_qualified_amount` clears `denial_reasons` and moves to `prequalified`
- setting `denial_reasons` clears `pre_qualified_amount` and moves to `denied`

`denial_reasons` is a **jsonb array**, rendered by `formatted_denial_reasons` (joined with
", ", `"N/A"` when blank).

## Credit score bands

`ESTIMATED_CREDIT_SCORES` is a Rails enum with **11 bands**:

`below_620` · `620_639` · `640_659` · `660_679` · `680_699` · `700_719` · `720_739` ·
`740_759` · `760_779` · `780_799` · `above_800`

**`DEFAULT_CREDIT_SCORE = "700_719"`**

> ⚠️ **The default is not "unknown" — it is an actual mid-high band.** A HELOC lead with no
> credit information is treated as 700–719 unless overridden. Any analysis of credit
> distribution will over-represent that bucket. This is finer-grained than the BBYS Equity
> Boost bands (Poor <620 / Fair 620–679 / Excellent 680+) in [[repo-hapi]] — **the two
> systems band credit differently.**

## `heloc_leads` schema

`external_id` (uuid) · `lead_id` · `stage` · `estimated_home_value` ·
`target_down_payment_percentage` · `purchase_loan_interest_rate` ·
`hoa_association_dues_monthly` · `property_taxes_monthly` · `home_owner_insurance_monthly` ·
`mortgage_insurance_monthly` · **`pre_qualified_amount`** · `client_estimated_income_monthly`
· `form_1003_uploaded` · `credit_report_uploaded` · `loan_number` · `estimated_credit_score`
· **`heloc_funds_used`** · `primary_1003_google_drive_file_id` ·
`primary_credit_report_google_drive_file_id` · `incoming_residence_state_code` ·
`document_drive_folder_id` · **`denial_reasons`** (jsonb)

Associations: `belongs_to :lead` · `has_one :provider_lead` (polymorphic `providable`) ·
`has_many :heloc_lead_liabilities` · `heloc_lead_clients` · `heloc_lead_stage_updates`.
Reached from BBYS via `BbysLead has_one :heloc_lead, through: :order`.

`heloc_funds_used` is one of the 25 `ECONOMIC_INPUT_FIELDS` on the BBYS econ model, and
`heloc_adjusted_home_value` is the **sole** `HELOC_CLTV_OVERRIDE_FIELDS` entry
([[repo-hapi]]) — HELOC directly overrides BBYS CLTV.

## Slack notifications

Two separate notifications on a prequalification decision:

1. **`notify_prequalification_decision_to_slack`** → the channel in
   `ENV["HELOC_PREQUALIFICATION_DECISIONS_SLACK_CHANNEL"]`. Only fires when
   `bbys_equity_unlock_approved?` — **HELOC decisions on non-approved BBYS leads are
   silent.**
2. **`notify_prequalification_decision_to_bbys_lead_slack_channel`** → the deal channel

Both are wrapped in `rescue StandardError` → Sentry + log. **Slack failures never fail the
prequalification.** Suppressible via `suppress_prequalification_decision_slack_notification`
and deferrable via `defer_prequalification_decision_slack_notification`.

> Brandi's 2026-06-04 feedback that *"HELOC only tags LRM — can we make that tag Marc, LRM
> and LOS"* applies to notification #2.

## The HELOC Card (a separate product)

A distinct model cluster: `heloc_card_account` · `heloc_card_application` ·
`heloc_card_account_application` · `heloc_card_ach_request` · `heloc_card_event`, served by
its own engine **`HelocCardService`** (`/api/heloc-card-service`) with
`HelocCardManager`, `CreateHelocCardEvent`, `CreateHelocCardAchRequest`,
`FetchHelocCardAchRequests`.

`homes-fe` has `VITE_HELOC_CARD_SUBMISSION_HOST` — a **separate submission host**. The
public page is `homelighthomeloans.com/heloc`.

> This is a different product from the BBYS HELOC Equity Boost. Don't conflate them in
> reporting.

## The 1003 pipeline

- `Heloc1003OptionsStep` and `Heloc1003ParsingStep` in [[repo-homes-fe]]
- Events: **`bbys_heloc_1003_lite_parser_completed`** and
  **`bbys_heloc_1003_anomaly_detected`** — there is a "lite" parser and an anomaly detector
- A `1003_anomaly_detection` Periscope dashboard exists ([[repo-bi-periscope]])
- Anirudh asked (2026-06-09) whether the extractor version is controllable; *"cursor didn't
  show anything in code"* — **still unanswered**
- `primary_1003_google_drive_file_id` — the parsed 1003 is retained in Google Drive

## County loan-limit gate

Shipped 2026-07-27 (Thiago, feature-flagged on): the applicant enters the **new home's zip
code**, the system fetches the **county loan limit**, and **hides the HELOC step entirely**
when `New Home value − Down Payment > county limit`.

> 🔑 **HELOC step suppression is silent to the user.** Funnel analysis showing "HELOC step
> skipped" may mean *ineligible by county limit*, not *declined by the client*.

## Flags and related

`bbys-application-for-lenders-with-heloc` (+ `-2`, `-2-a`, **`-ab-3`**, **`-ab-3-a`**) ·
`quiz_flow_heloc_intro` · `bbys_schedule_heloc_closing_signing` (task) ·
`bbys_heloc_eb_upload_task` · HELOC-specific closeout document title matching
(PR #20089, case-insensitive, handles legacy + current naming).

`homes-fe` features: `mortgage-coach` product `heloc-90` (**90% max CLTV**, the most
permissive of the three products) and `HelocOptimizerSubmissionDialog` /
`helocOptimizerSubmittedBy` in [[repo-sales-app]].

## 2026-08-29 addendum — where HLHL actually lives

The full `POST /prequalifications` request/response contract, the `/prequalifications/{id}/scenario-updates`
optimizer endpoint, and the MeridianLink LOS path (HELOC loan files are created from the
**`HELOC` template**; `heloc_leads.loan_number` is the MeridianLink number) are documented in
[[lending-mechanics]]. The HLHL entity itself — NMLS **1529229**, `trus.io`, the `eave-go` /
`eave-web` repos — is in [[repo-eave-hlhl]].

> 🔴 **The HELOC prequalification engine is not in either Eave repo.** `eave-web` has three
> case-insensitive matches for "heloc" and none is a feature; `eave-go`'s only HELOC references
> are MISMO enum values. `homelighthomeloans.com` (which serves `/prequalifications` and
> `/admin/heloc-optimizer`) is a **third, undocumented application on Porter**. Nobody in the
> vault has identified its repo.

> 🔑 **The prequal response returns `calculations[]` keyed by `partner`** — HLHL evaluates each
> borrower against multiple capital partners and returns per-partner eligibility and line
> range. HAPI reads only `partner == "default"` and discards the rest.

## 2026-08-29 addendum — Nous + Gamma mining pass

Source: Nous "Meetings - Tulli" collection (Granola-derived, read-only) + Gamma deck
`g_nir4bt3n8lwxstl` ("BBYS Growth Command Center — Leadership Strategy, June 2026," read-only).
Both are **stated, not code-observed** — treat accordingly, and note the dates: this is older
than the file's own 2026-08-26 verification pass above, so where they conflict, the
2026-08-26 section wins.

### 🔴 D2C HELOC card compliance gap — stated 2026-06, current status unverified

Two independent sources on the same date window both describe the same gap, which strengthens
confidence it was real at the time:

- **Nous, "Tulli & Gui <> Marc Weekly," 2026-06-12** (stated by Marc Kaplan): *"D2C HELOC
  rollout has compliance and operational exposure due to missing licensed follow-up, warehouse
  line, and adverse-action process."*
- **Gamma, "BBYS Growth Command Center," slide "HELOC / Equity Boost: Momentum + Risk," dated
  2026-06-22** (stated to Homes exec leadership): *"A D2C card application launched without a
  warehouse line or adverse-action process for low-FICO applicants. **Three states reportedly
  auditing.** Must close before any D2C scaling."* Listed as a top-5 leadership action item in
  the same deck's closing slide.

This is **not currently in [[risk-register]]** and is not mentioned in this file's own
2026-08-26 verification section above (which confirms standalone HELOC is live and covers fee
structure, but does not address licensing/adverse-action/warehouse-line status). ⚠️ **Unknown
whether this was resolved between 2026-06-22 and today** — no later vault source confirms
closure. Added to [[risk-register]] §2 as a new row pending verification; ask Marc Kaplan or
Drew Uher directly before citing "three states auditing" as current.

### Aven — competitor demo notes (Nous, "Aven TLS HELOC," 2026-06-24)

Aven (a TLS-adjacent HELOC platform) demoed to Nick. Durable competitive-intel points not
elsewhere in the vault:

- Aven **has full mortgage servicing licensing** for its HELOC card — this is the direct
  comparison point behind this file's own "Open Legal Questions" row above ("Aven has full
  servicing despite similar positioning").
- Aven's strongest LO-facing positioning, per the demo, is a **"financial checkup" + borrower-link
  marketing strategy** — not the HELOC mechanics themselves. i.e., Aven markets the tool LOs use
  to open a conversation with their client, not the product spec.
- Aven income verification stack: Work Number, IRS Form 4506, Plaid — relevant if HLHL's own
  verification path (see [[lending-mechanics]]) is ever benchmarked against a competitor.
- Aven was expanding its non-owner-occupied credit box and its max loan toward **$1M** as of
  the demo date.
- Post-close condition handling and hard-pull timing were flagged as comparison points worth
  auditing against the HLHL HELOC portal experience — not resolved in the meeting, no vault
  doc has done this comparison yet.

Add an Aven row/cross-reference to [[competitors-bbys]] (done, 2026-08-29) — this file did not
previously name Aven despite already discussing servicing-license comparison against it.

## 2026-08-29 — from the Data Bridge knowledge base (KB, source: `knowledge_documents`)

New fact from the compiled Data Bridge KB (see [[data-bridge-kb-index]] for the full corpus).
Stated (compiled ops answer), not code-verified here. The Hawaii state-gate fact this KB pass
also surfaced ("Hawaii Equity Boost: HELOC Unavailable but Asset Equity Boost Still Applies") is
**already documented above** (code-verified, §"State-ineligible denial text") — flagged here only
as independent corroboration from a second source, not a new finding.

- 🔑 **High-balance county limits, not just the standard conforming limit, gate HELOC Equity
  Boost eligibility.** A first-lien loan amount that exceeds the *standard* Fannie Mae conforming
  limit can still qualify if it's within that county's *high-balance* Fannie Mae limit — operators
  should check the county-specific figure before declining a file on loan-amount grounds. Files
  approved this way may sit near the high-balance ceiling and warrant closer review.
  ("HELOC Equity Boost: High-Balance Fannie Mae Limit Eligibility Guidance")

### 2026-08-29 addendum #2 (LOOP-2 KB sweep — remaining ~290 title-only rows read via the
compiler's own `summary` column, not full content_text; see [[data-bridge-kb-index]] for method)

- 🔴 **DTI cap: possible drift from the documented 45%/50% rule.** The "Underwriting &
  Qualification (as of 2/25/2026)" section above (line ~163) states **DTI 45% max, with a 50%
  exception only for FICO 740+ (or 700–739 with ≥$3,500 monthly residual income)** — i.e.
  credit-score-gated. A 2026-06-23 KB article ("HELOC Equity Boost & BBYS DTI and CLTV
  Qualification Guidelines") states the **current** guidance is a flat **50% DTI max regardless of
  credit score**, explicitly calling the credit-score-dependent 45% cap "outdated," and instructs
  teams to update all FAQ/field-sales materials. These cannot both be current: either the rate
  sheet section in this doc is stale (last dated Feb 2026, four months before the KB article), or
  the June KB article is itself already stale relative to a later reversal not yet surfaced. Not
  resolved this pass — **treat the 45%/50%-by-FICO rule in this doc as unconfirmed** until checked
  against the live Sales App/portal calculator (the KB article says that's the actual source of
  truth for current DTI logic). Logged in [[numbers-that-disagree]].
- **Standalone HELOC vs. HELOC Equity Boost are structurally different products**, not just
  different pricing: standalone HELOCs have no teaser/no-payment period (interest starts
  immediately), default to primary residences, and use a separate application path entirely.
  Second-home standalone HELOCs are exception-only, not standard guidance. ("HELOC: Standalone
  vs. HELOC Equity Boost – Key Distinctions")
- ⚠️ **HELOC Equity Boost is categorically incompatible with jumbo/non-conforming purchase
  loans** — no exception path. Standard BBYS (non-HELOC) may still be compatible with jumbo if the
  LO confirms no investor/lender DTI overlays outside Fannie/Freddie guidelines. Don't apply the
  same jumbo tolerance to both products. ("BBYS Jumbo Loan Guidance: Standard BBYS vs. HELOC
  Equity Boost")
- **Asset Equity Boost caps below the general ~85% CLTV ceiling for retirement-account-backed
  boosts** — in at least one documented case a TSP/401k-backed Asset EB capped at **$50k**
  regardless of the borrower's account balance. Verify plan-specific limits (and, separately, the
  plan's own loan-policy terms conflict rule already noted for standard 401k loans) before quoting
  a maximum EB amount. ("Asset Equity Boost (EB): TSP/401k-Backed Limits and Operator Guidance")
- ⚠️ **State-availability lists disagree across sources** (independent of the DB-driven gate
  described in "State-ineligible denial text" above): one KB FAQ snapshot lists HELOC Equity Boost
  as unavailable in **TX, NY, AK, MA, VA, VT, IA**; a separate 2026-06-15 article lists **HI, AK,
  TX, NY** as broader than standard BBYS's **AK, NY only**. Neither list matches the other exactly
  (MA/VA/VT/IA appear in one but not the other; HI appears only in the second). Since the real
  gate is the DB-driven `marketplace_program_states` check, both lists are likely point-in-time
  snapshots of that table rather than independent hardcoded rules — but don't quote either list as
  current without checking the live gate. ("HELOC Common Questions & Objections" [already in this
  doc]; "BBYS & HELOC Equity Boost: Compatibility and State Availability Guidelines")
- Texas 50A6 departing-residence files are capped at **~70% CLTV**, not the standard 90%
  Equity Boost/HELOC ceiling — don't quote the higher ceiling even where BBYS/HELOC is otherwise
  available in Texas. ("Texas 50A6 CLTV Limit for Departing Residence Files")

## 2026-08-29 — Nous archive tail (meetings_archive/threads/topics collections, read-only)

Loop-2 mining pass into two Nous collections a prior pass ([[projects/2026-08-29-nous-gamma-mining]])
found but did not open. **Hard access limit:** `meetings_archive`, `threads`, and a related
`topics` collection are indexed for Nous's semantic search but return `permission_denied` on
every direct fetch attempt (`query_collection` by ID and by name, `get_collection_items` by item
ID) — confirmed on both meaning search can surface fragments of these collections' records that
this identity cannot otherwise open. See [[projects/2026-08-29-nous-archive-tail]] for the full
methodology note. Everything below was reconstructed from search-result snippets only — no full
meeting transcript or thread body was read. The `topics` collection's own snapshot marker reads
`status: 05-14` (2026-05-14) on most entries; treat all of this as a **mid-May 2026 snapshot**,
not verified against anything more current.

- 🔑 **HELOC Card / Access Device mechanics (topic narrative, `heloc-card-access-device`,
  snapshot 2026-05-14):** the physical/digital card for HELOC funds disbursement runs through
  **Pesto**, drawing against a warehouse line from **Luminate** (see partner mechanics below) —
  "HomeLight manages debit cards via Pesto, draws against Luminate's line." Distribution
  strategy at the time was 20 individual top-performing LO teams; **target was 50 card
  applications by an April 27, 2026 board meeting** (unconfirmed whether hit — no later vault
  source addresses it). A card-rewards enhancement (2–3% cash back for refi-rate buydown) was
  **proposed**, not confirmed shipped. `(stated, single source, not code-verified)`
- 🔑 **Secondary-market plan for HELOC paper (topic narrative, `secondary-market`, snapshot
  2026-05-14):** volume projection **$90–180M in year one**. **MCT** was providing
  scratch-and-dent pricing at the time; a person referred to as "Bitkin" was evaluating
  retained-servicing options and Wall Street buyer connections. Named prospective buyers/targets
  in later meeting titles (Aug 17 NYC trip prep): MCT, Goldman, Planet, MIAC, Figure, TMC.
  **Flagged salability concern, stated in-meeting:** 30-year HELOC terms on 1–3-year draw periods
  "have no comparable products" in the secondary market — i.e. the paper may be hard to sell as
  structured. Not resolved in any source found. `(stated, not verified)`
- ⚠️ **Compliance workstream is older and broader than the single D2C-card gap already logged as
  [[risk-register]] row 38.** A `compliance` topic (snapshot 2026-05-14) names three concurrent
  threads: **Texas 50(a)(6) historical fee refund exposure** (open), a **California regulatory
  crisis** — "June 2026 on-site state exam, 110 servicing items" (open), and a **Continental Bank
  IT compliance side letter** with a stated deadline of "end of May / mid-June" 2026. Meeting
  titles tied to this same topic continue at least through **2026-08-21** ("HELOC Project Sync"),
  meaning this is a long-running, still-active workstream, not a one-time item — but no vault
  source states current status on any of the three sub-items. Added as [[risk-register]] row 47.
  `(stated, not code-verified — see risk-register row 47 for full citation)`
- Partner mechanics for the Luminate warehouse relationship (TPO distribution, $5M line, $1M
  restricted deposit) are documented in [[partners]] §"2026-08-29 — Nous archive tail" — do not
  duplicate here, only cross-reference. Nous's own `people` record for the Luminate contact
  (Jerry Kaplan) is marked `stale: true` inside Nous itself, i.e. even the source system flags
  this relationship as no longer current as tracked.

## Open questions (system side)

- Can HELOC prequalification ever run **without** a BBYS lead? `prequalify!` exits early if
  `bbys_lead` is blank, which conflicts with "standalone HELOC is live."
- Which states have `heloc` enabled in `marketplace_program_states`? That is the real
  service map.
- Is `DEFAULT_CREDIT_SCORE = 700_719` intended as a real default or a placeholder?
- Why can a `denied` lead re-enter preliminary prequalification?
- Who owns the 1003 extractor version?
- **Was the D2C card compliance gap (missing warehouse line / adverse-action process, three
  states auditing per the 2026-06-22 leadership deck) ever closed?** No vault source after
  2026-06-22 addresses it. Highest-priority follow-up from this addendum.
- **Did the California on-site state exam (June 2026, 110 servicing items) happen, and what was
  the outcome? Was the Continental Bank IT compliance side letter closed by its stated May/June
  deadline? Was the Texas 50(a)(6) fee-refund exposure resolved or quantified?** All three
  surfaced only as open items in a mid-May 2026 Nous snapshot; no later vault source addresses
  any of them. See [[risk-register]] row 47.
- Was the 50-card-application target (stated for an April 27, 2026 board meeting) hit?
