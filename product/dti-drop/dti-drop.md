---
last_updated: 2026-09-24 (comp status: deployed but switched off; 9 September closes flagged, all under $750K; waiting on Jake's start date). Previous: 2026-09-16 ~14:25 MT (Elite qualifying YTD HubSpot calcs LIVE — [[2026-09-16-elite-qualifying-ytd-hubspot-calcs]]. Previous: first HubSpot flat stamp + Jake LIVE announce.)
type: context
source: HomeLight-Vault/context/dti-drop.md
imported: 2026-09-29
---

# DTI Drop Program

> Context for the DTI Drop variant of BBYS. Claude reads this for DTI Drop-related work.
>
> Fee/cost model lives in [[bbys-unit-economics]].

## ⚠️ DTI Drop was formerly called BYOC ("Bring Your Own Cash")

**Same product, two names.** Confirmed by Tulli 2026-08-26. BYOC is the legacy internal name and
still appears throughout Sales App, agreements, email templates, and older Slack threads.

Where you will still see "BYOC":
- **Sales App** carries a BYOC true/false flag that determines which agreement template generates
  (*"for BYOC please ensure that you mark True in the SalesApp so the correct agreement is generated"*)
- Email templates — e.g. `bbys_day_0_all_status_byoc`
- Notary/signing packages defaulted to the name "BYOC" in SignatureSync; a request was raised to
  relabel these as "DTI Drop"
- Staff shorthand **"DTID"** is also used
- ECPAI has a dedicated "BYOC Fee Rate" input in the Finance model

Jake Vogel's canonical description: *"if the borrower is doing BYOC which is contingency removal
only… Since we don't place a lien, we avoid all of the above."*

### BYOC/DTI Drop fee history

| Effective | Fee | Minimum |
|---|---|---|
| Through 2025-05-15 | 1.5% of final sale price | $7,000 |
| **2025-05-16 →** | **1% of final sale price** | **$5,000** |
| **2026-09-16 → (NEW apps, Jake announce)** | **flat $2,500** under $750K home value; **flat $5,000** at $750K+ | n/a (flat) |

Announced internally 2025-05-16, to LOs on 2025-05-22. Flat NEW-app pricing announced LIVE by Jake Vogel in `#lab-rats-lp-sales` **2026-09-16** — full detail [[2026-09-16-jake-dti-drop-pricing-live-announcement]]. ✅ First NEW create stamped `bbys_pricing_basis=dollar_amount` + $2500 on deal `65107199603` (~1:50 PM MT). Morning legacy `65060986500` can stay `$5050`/`percentage` until restamp.

> 🔑 **Source of the rename, dated:** `#proj-dti-drop` (Slack), created 2025-05-28 by Annie
> Dreshfield specifically to coordinate the BYOC→"DTI Drop" rebrand campaign, bundled with the
> 1.5%→1% pricing update above. Rollout: Iterable brand-announcement email, Canva-designed
> illustration/one-pager, updated lender- and agent-center copy, MRC one-pager updates for all
> partners. Fairway and TLS had already received the pricing change the prior week and were
> deliberately excluded from the rebrand announcement timing. The channel has zero activity
> after 2025-05-28 — a one-shot campaign, not an ongoing workstream. See
> [[2026-08-29-slack-product-pods-6mo]] for the full channel read.

### Other BYOC/DTI Drop specifics

- **No lien is placed on the DR** — this is the structural difference from standard BBYS, and why
  second-lien payoff rules do not apply
- From **2025-06-01**: a **$150 notary fee** and a recording fee are added. HomeLight fronts both
  and recovers them on the payoff when the DR sells
- **CMF / HLCS closing management fee is automatically waived** on BYOC
- **No fee exceptions on the minimum fee** — Jake: *"No fee exceptions on min fee. They can do BYOC for 7K."*
- **Extensions are 0.5% per 60 days**, not the 1.2% standard BBYS rate (Evan Jaquias-Johnson)
- **Orchard will not do BYOC** — *"assume max EU on every file"*
- Opened in **NY and MA** in April 2025 ahead of full licensure

## What Is DTI Drop?

A BBYS variant for clients who **don't need equity** to buy their next home. Instead of providing an Equity Unlock (bridge loan), HomeLight places a backup purchase offer on the departing residence, which per Fannie Mae B3-6-06 and Freddie Mac 5401.2 allows the existing mortgage PITIA to be **excluded from the borrower's qualifying DTI**.

## How It Works

1. LO submits client's DR through normal BBYS portal
2. Econ model runs — if EU comes back at $0 (or negative), flagged as DTI Drop candidate
3. HomeLight places guaranteed backup purchase contract on DR (no financing contingency)
4. Client uses own cash for new home's down payment (HomeLight does NOT fund any EU)
5. Client makes offers without home sale contingency
6. After closing on new home, old home sells at full market value
7. HomeLight's backup offer = safety net if home doesn't sell within 120 days

## DTI Drop vs Standard BBYS

| | Standard BBYS | DTI Drop |
|---|---|---|
| Equity Unlock | Yes (bridge loan funded) | No ($0 EU) |
| Program Fee | 2.4% of DR sale price ($9K min) | 1% of DR sale price ($5K min) |
| Florida Fee | 2.9% | 1% |
| Contingency Removal | Yes | Yes |
| DTI Relief | Yes | Yes |
| Revenue at IR Close | Yes | No |

## Key Rule: DTI Drop + HELOC Interaction

- If HELOC only bridges to $0 EU (no actual equity funds): fee stays at **1%**, no HELOC draw
- If HELOC funds actual EU for down payment: flips to standard BBYS at **2.4%**
- Jake Vogel: "As long as we are not funding any equity it will stay 1%"

## Negative EU Paths to $0

- **Mortgage paydown** — client pays down balance ($6K-$26K common)
- **Asset Equity Boost** — pledge 401k, savings, investments (not liquidated)
- **HELOC Equity Boost** — open $0-draw HELOC on IR to bridge gap

## Campaigns

- **DTI Drop Nurture** — historically drove 16-18% click conversion (best email theme)
- Campaigns paused for 4-5 months, dropped to ~7% — identified as root cause for SQL-to-App conversion decline
- Andrew Soss relaunching (target: end of next week)
- HubSpot list: "DTI Drop Nurture Clicked on Lead Submission Link" (list ID 19140)

## HubSpot Field

🔑 **Internal name: `bbys_contingency_removal_only`.** Label is "DTI Drop". The internal name
describes the *mechanism* (contingency removal without a lien), not the product — **anyone
grepping HubSpot for "dti" will miss it.**

- Type: single line text (not a boolean, not an enum). Values: `"true"`, `"false"`, `"unknown"`
- Created 2025-06-12; syncs from Sales App whenever workflow nodes run
- Gui built separate EUA follow-up workflows to avoid sending equity content to DTI Drop leads

### It is the only DTI Drop field — verified exhaustively (2026-09-09)

Searched **all 918 deal properties, including hidden**. Exactly one match:

| name | label | verdict |
|---|---|---|
| `bbys_contingency_removal_only` | **DTI Drop** | ✅ the one |
| `heloc_dti_percent` | DTI percent | ✗ HELOC debt-to-income ratio, unrelated |
| `bbys_ir_loan_contingency_end_date` | BBYS IR Loan Contingency End Date | ✗ unrelated despite the name |
| `bbys_expected_revenue` | BBYS Expected Revenue | ✗ mentions DTI Drop in its description only |

⚠️ **Do NOT use `deals_bbys_euc_dropout_type`** to identify DTI Drop. Despite the name it is
labeled "BBYS Opportunity Type" and its options are `Lender Opportunity / Agent Opportunity /
Builder / Integration / D2C / Lennar NHC / Contact Info Only / Pre-EUC Dropout / EUC Dropout /
Agent EUC Dropout / Other EUC Dropout / eng-staging`. **There is no DTI Drop option.**

### 🔑 The flag was backfilled — good news for anything needing history

The property was created 2025-06-12, so the natural assumption is that earlier deals are blank.
They are not. Sampled closed deals, 100 per month:

| Close month | `true` | `false` | `null` |
|---|---|---|---|
| 2024-06 | 4 | 96 | **0** |
| 2025-06 | 6 | 94 | **0** |
| 2026-08 | 16 | 84 | **0** |

**Zero nulls anywhere**, including a full year before the field existed. The rising true-rate
(4% → 6% → 16%) tracks DTI Drop growing as a product, which is evidence the backfill carries real
signal rather than a blanket `"false"` default.

This mattered for [[2026-09-09-dti-drop-flat-comp]]: the comp change counts an LO's BBYS closes
across all time, so the flag has to be right on *all* history, not just post-cutover.

### ⚠️ Nothing in HubSpot can cross-check the flag

DTI Drop bills **1% / $5K min** vs standard BBYS at **2.4% / $9K min** — a 2.4× spread that should
separate trivially on price. It doesn't, because **`bbys_expected_revenue` frequently stores the DR
value itself rather than a fee**:

```
dti=false  expected_revenue 325,000.00   lo_est_dr_value 325,000.00   ← revenue == value
dti=true   expected_revenue 482,777.00   lo_est_dr_value 482,777.00   ← revenue == value
dti=true   expected_revenue   9,000.00   lo_est_dr_value 314,359.00   ← the $9K STANDARD-BBYS
                                                                        minimum, on a deal
                                                                        flagged DTI Drop
```

The field's own description says it switches basis for DTI Drop ("uses the authoritative BBYS Fee
Value synced from the HAPI pricing engine… Standard BBYS retains the existing 2.4% / 9000-min
calculation"), so it is mid-migration. Same problem [[bbys-unit-economics]] §8 tracks: multiple
revenue-adjacent bases that don't tie. See [[numbers-that-disagree]] row 34.

**Practical consequence:** if a rep ever disputes a flat DTI Drop comp payment, HubSpot alone
cannot adjudicate it. The references are **Sales App** (which carries the upstream BYOC boolean
driving agreement generation) and **program fee actually collected** in Finance/ECPAI.

## Comp treatment (2026-09, pending cutover)

DTI Drop closes move off bps onto a flat **$100 under $750K comp-able value / $200 at or above**,
for AEs, AMs and Builder reps — and they stop consuming a slot in the LO's escalating
6 / 10 / 13 / 2 bps close ladder.

**Built but not published; no cutover date set.** Full detail, impact numbers and open questions
in [[2026-09-09-dti-drop-flat-comp]]; plan mechanics in [[comp-plans]] §9.

**Status 2026-09-24:** the code has been deployed since 2026-09-21 (PR #7) but stays switched off
until a Comp Rules rule set carries `dti_drop_comp`. **The comp app can already tell the two fee
types apart**: `deals.dti_drop` is synced from HubSpot `bbys_contingency_removal_only`. 9 of the 86
September 2026 closes are flagged, all under $750K, so they would pay $100 each. Until the rule set
is added, they are still paid at bps. **Blocked on Jake's start date** (1st of a month, anchored on
close date). Jake asked about it in Slack on 2026-09-24. Note that product pricing went live on
2026-09-16, which is not the same as the comp cutover.

## Known Issues

- Agreement file name was hardcoded wrong — should be "Version 3.0, Updated 1-2026"
- Addendum showed raw template code when no exceptions present
- FHA loans: no DTI benefits applicable — DTI Drop useless for FHA
- Competitive pressure: FlyHomes at $2,500 flat fee (homes under $500K)

## Ownership

- Product: Jake Vogel, Jason Smith (UW/approvals)
- Builder DTI Drop: Karly Trota, Tiffany Traxler, Patricia Pinckard
- Campaigns: Andrew Soss
- Workflows: Gui Batista

## Call Scoring Classification (2026-04-16)

Added as a `call_types` row so the scoring pipeline can tag calls that discuss this variant.

- **UUID:** `357f27e2-f510-4e1c-a51c-c57b32d58482`
- **Description in DB:** *"Discussed DTI 9debt-to-income) Drop, a Buy Before You Sell variant that puts an offer on the departing residence but no equity unlock loan"* (typo "9" should be "(" — safe to fix later)

### Backfill: 62 calls tagged over last 7 days

- **52 calls** auto-tagged via SQL tight-match — explicit phrase "DTI drop" or "DTI draft" in the transcript (high precision)
- **10 calls** tagged via AI verification pass — mentioned DTI generically but described the specific product variant (no equity unlock, offer on departing residence, DTI-only benefit). Used `gpt-4o-mini` against 156 medium-confidence candidates. Zero errors.
- **146 generic DTI mentions correctly skipped** — reps describing DTI as one of several BBYS benefits, customer context, etc.

Going forward, new calls auto-tag via the live classifier (reads all active `call_types` dynamically). No code changes needed.

If we ever want to tighten or re-classify, the pattern is: targeted SQL pre-filter on keywords → AI verification against the type description → UPSERT into `call_call_types` (composite unique on `call_id, call_type_id`).


## NEW-app flat pricing LIVE (Jake 2026-09-16 announce)

Source: Jake Vogel in `#lab-rats-lp-sales` (C06D60XFXFA / ts 1789587035.338019). Full note: [[2026-09-16-jake-dti-drop-pricing-live-announcement]].

| Rule | Value |
|------|-------|
| Under $750K | **$2,500** flat |
| $750K+ | **$5,000** flat |
| LO paid | Only if fee charged **above** base |
| Apply timing | Before IRUC (Elite: can at IRUC; post-IRUC case-by-case) |
| DTI Drop program period | **180 days** (BBYS stays **120**) — 🔴 supersedes earlier 120-for-DTI-Drop enablement notes |
| Elite | Retention by EOY to keep **pricing**; ladder volume: only **1 new-flat** DTI IR close/year qualifies (old-pricing DTI uncapped) — HubSpot [[2026-09-16-elite-qualifying-ytd-hubspot-calcs]]. Else back to **1%** pricing per Jake announce. |

**CRM confirmation:** first true flat stamp landed ~1:50 PM MT — deal `65107199603` (`bbys_pricing_basis=dollar_amount`, fee **$2500**, CRO true, ~$289k). Blank HubSpot prop named `dollar_amount` is unrelated noise. Legacy morning CRO `65060986500` still $5050/percentage. Do **not** change type of `bbys_contingency_removal_only`.


## Elite qualifying YTD (HubSpot calcs LIVE 2026-09-16)

Nick-locked: old-pricing DTI uncapped; **new flat** DTI (`bbys_pricing_basis=dollar_amount`) only **first IR close/year** counts toward Elite ladder. Full write-up: [[2026-09-16-elite-qualifying-ytd-hubspot-calcs]].

| Object | Property | Role |
|--------|----------|------|
| Deal | `dti_new_flat_ir_closed_flag` | 1 if CRO + dollar_amount + `ir_closed_date` in year(NOW) |
| Contact | `dti_new_flat_ir_closes_ytd` | Sum of flag on LO-attached Deal (14) |
| Contact | `elite_qualifying_ir_closes_ytd` | DTI-capped IR qualifying + COALESCE(`elite_manual_credit_ytd`, 0) — [[2026-09-17-elite-manual-credit]] |
| Contact | `elite_lender_status` | Ladder uses qualifying (raw `ir_closings_ytd` kept for reporting) |

Separate from all-time LO DTI count `dti_drop_deals_bbys` / Jake EOY pricing-retention rules.

## HubSpot LO contact rollup (2026-09-15, LIVE)

Portal 4744876. Full write-up: [[2026-09-15-dti-drop-hubspot-contact-rollup]].

| Object | Label | Internal name | Type |
|---|---|---|---|
| Deal | DTI Drop Flag | `dti_drop_flag` | calc equation → 1 if `bbys_contingency_removal_only` string equals `'true'`, else 0 |
| Contact | DTI Drop Deals (BBYS) | `dti_drop_deals_bbys` | rollup Sum of `dti_drop_flag` on **LO-attached Deal** (type 14) only |

Do **not** change text SoT `bbys_contingency_removal_only` (Crunchy → n8n XQ6). Rollup conditions need number/enum — hence the companion.

Also: Deal Info section **DTI Drop & Pricing** live on BBYS Opportunities & Leads view ([[2026-09-15-dti-drop-deal-info-view]]); XQ6 still must map `bbys_pricing_basis` ([[2026-09-15-xq6-bbys-pricing-basis-gap]]); fee parity [[2026-09-15-dti-drop-hubspot-db-parity]]; launch pack index [[2026-09-15-dti-drop-launch-pack]].

## Related Notes

- [[2026-09-16-elite-qualifying-ytd-hubspot-calcs]]
- [[2026-09-16-jake-dti-drop-pricing-live-announcement]]
- [[bbys-unit-economics]]
- [[bbys-overview]]
- [[heloc-product]]
- [[aircall-system]]
