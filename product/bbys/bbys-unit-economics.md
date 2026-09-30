---
last_updated: 2026-08-29 (§11 added — 2026 management plan model, from Drive)
type: context
source: HomeLight-Vault/context/bbys-unit-economics.md
imported: 2026-09-29
---

# BBYS Unit Economics — Revenue, Fees & Cost Model

> Canonical model for how a BBYS deal makes (and loses) money. Claude reads this before any
> revenue, fee, pricing, comp, or P&L work.
>
> **Why this file exists:** fee mechanics used to be scattered across [[dti-drop]], [[bbys-edge-cases]],
> [[partners]] and [[lo-lifecycle]], so any blended-take-rate question required reassembling the model
> from memory. This file owns every revenue and cost line; the others link here.
>
> Related: [[bbys-overview]] · [[dti-drop]] · [[heloc-product]] · [[partners]] · [[bbys-edge-cases]] · [[projects/2026-08-17-channel-pnl-finance-readiness]]

## Provenance

Reconstructed from Slack (Aug 2026) and graded by Tulli in the same session. Items marked
⚠️ are **unverified** — do not treat as fact until confirmed. Items marked 🕰️ are known-stale.

---

## 1. Program Fee

### Standard rates

| Product | Fee | Minimum |
|---|---|---|
| Standard BBYS | **2.4%** of DR final sale price | **$9,000** |
| Florida BBYS | **2.9%** | $9,000 |
| DTI Drop / BYOC | **1%** of final sale price | **$5,000** |
| Florida DTI Drop | 1% | $5,000 |

**Minimum-fee history** (needed for any historical restatement):
- May 1 – Nov 1, 2023: **$11,000** minimum
- From Nov 2, 2023: **$9,000** minimum

⚠️ **Possible client-agreement error (stated, added 2026-08-29, unresolved):** per Nous
"Meetings - Tulli" (Granola, read-only), "LSM & LRM Team Meeting" (~2026-06-18): *"Agreement
says $9K minimum for properties under $375K, but Florida minimum applies under roughly
$310K."* The math checks out — $9,000 ÷ 2.9% (Florida BBYS rate) ≈ **$310,345**, the actual
price at which the flat $9K minimum overtakes the percentage fee. If the client-facing BBYS
Agreement document literally states "$375K" as that threshold, it is wrong by ~$65K and
potentially misrepresents the minimum-fee trigger point to Florida clients. The same meeting
notes "legal team is expected to create corrections within the next week" (i.e., ~w/o
2026-06-22) — **status unconfirmed**; no later vault source says whether the agreement
template was actually corrected. Verify the current BBYS Agreement template with
Legal/Karly Trota before relying on either number.

### The rule for 2.4% vs 1%

> *"Whenever we release funds at time of IR closing, it's BBYS. So 2.4%."* — Karly Trota

If HomeLight funds **any** equity, it is standard BBYS at 2.4%. If EU is $0 (client brings own
cash, or Asset/HELOC Boost is used only to bridge to $0), it stays DTI Drop at 1%. See
[[dti-drop]] for the HELOC interaction rule.

Changing the fee also moves the HELOC constraint math — one file's CLTV cap moved from 91% →
92.4% when the fee was switched from 1% → 2.4%.

### Fee is charged on final DR sale price, quoted off HL valuation

The agreement quotes off the econ model (HL valuation × fee %), but the fee actually charged is
a percentage of the **final sale price**. As of **Oct 2025**, marketing was directed to stop
publishing "2.4%" on marketing sites and the LO Portal, replacing it with
**"program fee (adjusted upon the sale of the departing residence)"**, and to show the dollar
amount without the percentage (e.g. `$15,000 (2.4%)` → `$15,000`).

---

## 2. Variable Pricing (VP) — now a product feature, not an exception

VP is a **systematized pricing model** in Sales App (internally "dynamic pricing"), not a
one-off Jake Vogel exception. It rewards a fast DR sale with a lower fee.

### Standard VP schedule

| Period | Fee |
|---|---|
| Days 0–29 | **1.5%** |
| Days 30–89 | **+0.015 percentage points per day** |
| Days 90–120 | **2.4%** (standard fee — the cap) |

Confirmed verbatim by two independent sources six months apart (Deborah Shutt, Mar 2026;
Tierney Izar, Aug 2026).

### Florida VP schedule ⚠️ may still be in build

| Period | Fee |
|---|---|
| Days 0–30 | 1.5% |
| Days 31–89 | **+0.0233 pp per day** |
| Day 90+ | **2.9%** (cap) |

Florida **starts at the same 1.5%** as standard — it ramps ~1.55× faster to reach the higher
cap. Source is Joel Shurtleff reviewing the model with Taylor Wong on 2026-08-10, who then
asked for the list of FL partners it should apply to — so this reads as *in design*, not
shipped. **Verify before quoting.**

### Flat-fee exception (the alternative offer)

Instead of VP, a client can be offered a flat-fee exception — typically **1.8%–1.9%**,
sometimes **2.0%**, sized off the approved equity unlock. LRMs present both options and let the
client choose.

### VP eligibility rules

- **Gate: DR value ≥ $425,000.** Below that the deal is flat 2.4% or the $9K minimum.
- Eligible partners: **default on, auto-included ≥$425k, blocked <$425k**
- **APM** was the first partner live on dynamic pricing
- **Envoy is automatic VP**
- Sales App field **"Pricing Model"** = `Flat-Fee` | `Variable-Fee`, auto-populating since
  **2026-01-26**; since May 2026 it toggles the VP verbiage on the agreement generator
- **Critical:** *"If neither the conditional or final agreement have VP on the addendum, we will
  not apply it at time of DR sale regardless of sales price."* (Karly Trota). VP is decided off
  **agent/lender estimated value at agreement time**, not final sale price.

**Known bug (open as of Jul 2026):** the Pricing Model field still reads `Variable` on sub-$425k
files for eligible partners, causing mismatches on the BUO/final-agreement review tool and
confusing CAs. Bruno patched the comparison logic (`if partner in list AND < $425k, consider VP
false`); the real fix — flipping the field to `Flat-Fee` when home value disqualifies — was
still pending.

---

## 3. The LO spread — gross program fee ≠ HomeLight revenue

**This is the single biggest trap in any BBYS revenue calculation.** LOs can ride their own
compensation on top of HomeLight's fee. The client sees one blended "program fee"; HomeLight
only keeps its portion.

Live examples:
- *"5% Fee For BBYS. HL will keep 2.4% LO keeps 2.6%."*
- *"Fee is 2.4% and He is collecting 1.1% in addition so total program fee is 3.5% with 1.1% going to pay the LO for running the all-cash file."*
- VP variant: client pays 2.5%; the LO earns the 1.5%–2.5% spread; post day 30 the LO's share
  decays ~0.015%/day as HL's VP eats into it, flooring at **0.1% on day 90**. *"None of this
  will be on the BBYSA and will only be noted internally in the notes section."*

**Implication:** the fee percentage on the BBYSA is not a reliable revenue input. Anything
reading a fee % off the agreement will overstate HomeLight revenue on LO-spread deals.

Separately, the **$1,500 LO benefit** (Elite / TLS deals) is paid to the LO after the DR closes,
or credited to the client on the settlement statement — Evan Jaquias-Johnson preferred the
credit-on-payoff route to eliminate billing steps.

---

## 4. Extensions

Standard program period is **120 days**; the loan note is due day 120.

| Extension | Days | Added fee | Cumulative fee |
|---|---|---|---|
| Program period | 0–120 | — | 2.4% |
| 1st extension | 121–180 | +1.2% | **3.6%** |
| 2nd extension | 181–240 | +1.2% | **4.8%** |
| Auto-renew | 241–300 | per amendment | — |

- The extension amendment **auto-rolls to 240 days** if payoff isn't completed by day 180; a
  300-day auto-renew exists beyond that. Files at day 360–420 have been observed.
- **DTI Drop extensions are 0.5% per 60 days**, not 1.2% (Evan Jaquias-Johnson).
- Each extension requires a **loan modification/refi + repaper + notary signing**, and must be
  aligned to the **warehouse line maturity** — extension expiry and warehouse aging have drifted
  out of sync before.
- **Per-diem proration:** `(sale price × 1.2%) / 60` = daily rate. Worked example: $500k ×
  1.2% = $6,000 / 60 = **$100/day**. Granted when the client goes under contract before the
  deadline but closes inside the extension window. Past day 180 the full 1.2% applies.
- **Extensions are almost always granted.** Christy Meek: the only real blocker is pricing — if
  the listing is well above HL valuation, mandatory price-drop guidance becomes a condition of
  extending. Extension approvals routinely carry a dated price-reduction schedule.
- **Orchard** files: extension terms can be locked before going IRUC.

### UGA — Upside Guarantee Agreement

After the program period, HomeLight buys the property at **LPV**, keeps it listed at market,
and remits net profit back to the client (`final sale price − LPV − fees and costs`). HomeLight
determines list price once it owns the home.

---

## 5. Other fee and cost lines on the payoff

Every DR payoff demand ("PO") carries these lines:

| Line | Typical | Notes |
|---|---|---|
| **CMF — Closing Management Fee** | **$1,450** | **Waived if the seller uses HLCS** (HomeLight Closing Services). Waived on nearly every observed payoff. Also used as a negotiating lever — extension-fee proration has been offered conditional on moving title/escrow to HLCS. |
| Late Listing / Listing Surcharge | $0 | Frequently waived |
| Inspection | $485–$970 | Vendor: Inspectify. **$900 is the econ-model default**, trued up to actual |
| Notary | **$150** | Rolled into the total loan amount |
| Recording | $41–$310 | Rolled into the total loan amount |
| Documentary & Intangible Taxes | ~$1,738 (FL example) | Florida only |
| HOA | $0–$75 | |
| **Maintenance Reserve Holdback** | $6.7k–$45k | Almost always released "NOT Utilized"; occasionally partially drawn |
| Solar lease holdback | **$10,000** | Credited off the EU loan if the lease transfers to the buyer within 120 days. If HL purchases on day 120, panels are removed and the home relisted; client owns the lease obligation. |

Sales App now has **dedicated HOA and Inspection Report cost fields** feeding the payoff demand
generator directly — these should no longer be rolled into the Econ Model.

---

## 6. Cost of capital & interest expense

The Equity Unlock is funded on a **warehouse line**, with a portion funded through **TLS**.

Interest expense formula as modeled in ECPAI (see §7), with its revision history:

```
2023 baseline : EUA × 0.0755 × days_EUA_outstanding / 365
Jan 2024      : EUA × 0.0800 × days_EUA_outstanding / 365 + (50% × ((EUA × 0.0025) + 250))
Jul 2024 doc  : EUA × 0.0807 × days_EUA_outstanding / 360 + (20% × ((EUA × 0.0025) + 250))
```

- The second term is the **TLS funding cost**. TLS charges **25 bps**; Sarah Coyne confirmed
  *"We don't pay TLS $250 but do pay the 25bps"* — the $250 in the formula is questionable.
- The 50% / 20% coefficient is the assumed **share of loans funded on TLS**. Actuals: FY2023
  12%, Aug–Dec 2023 20%, Jan 2024 trending 33%. Vanessa flagged that the 50% figure was never
  correct.
- Interest accrues on **funded EU**, not approved EU (changed Oct 2024). Where funded EU is
  missing in Sales App, the amount is pulled from Encompass.
- **Days EUA Outstanding = expected DOM + 20-day buffer**; where expected DOM is missing, the
  state-average expected DOM + 20 days. Actual average rose **75 → 85 days** across 2024.
- 2024 volume reference: approved EU ≈ **$242M**, funded EU ≈ **$250M**.

⚠️ **The 180-day program-period change (see §9) materially increases carry** — up to 60 extra
days of interest per deal before any extension fee is collected. At ~8% that is roughly 1.3% of
EUA. Confirm Finance sized this against the fee change.

> 🔴 **2026-08-28 (stated, Granola "Weekly Sync"):** the **Chase/JPM warehouse line — a $150M
> revolving credit facility for BBYS — was paused** after an auditor discrepancy: an auditor
> connected to Vicki reportedly sent Homelight and Chase different versions of an audit, and
> Chase's copy showed a risk score above their internal threshold. Marc Kaplan reported Sarah is
> pessimistic about getting the line reinstated; Derek is moving deals to other lines in the
> meantime (one deal required interim equity funding, then was repapered to TLS two days later).
> **Needham Bank (NY)** is in due diligence on a **$75M** line intended to cover roughly half the
> gap. Stated business framing: the team's monthly target is 615 deals; losing Chase creates a
> capacity gap but was not characterized as an immediate crisis. **Not yet resolved as of
> 2026-08-28** — see [[2026-08-29-granola-decisions-6mo]]. This is a capital-availability risk
> layered on top of the cost-of-capital modeling above, not a change to the interest-expense
> formula itself — flag before modeling forward BBYS volume against warehouse capacity.

---

## 7. CPAI — Contribution Profit After Interest 🕰️

**CPAI = Contribution Profit After Interest** (confirmed by Vanessa Famulener). It is *not*
cost-per-application. Finance/BI owns the lineage.

```
Expected CPAI = Homes Revenue − COGS − Selling & Marketing Expense − Interest Expense

  Homes Revenue = BBYS Revenue + Mortgage Revenue
  BBYS Revenue  = HL Valuation × BBYS Fee %
  COGS          = $300  →  $250 (Oct 2024)
  S&M           = $1,500 for TLS-sourced enterprise partner
                  ($1,000 for IR contracts received 5/1–7/31/2023)
  Interest      = see §6
```

**Risk Loss** = `BBYS Revenue − COGS − Selling Costs − Warehouse Fees − TLS Service Fees −
Interest Expense`. If negative, that deal booked a risk loss.
- Target: average risk loss **≤ $800 per transaction**
- $800/lead was incorporated as an additional cost in Oct 2024
- Vanessa: *"risk loss is very lumpy and will happen on only a handful of deals, but when it
  does it is a very large amount"*

### ⚠️ Known problems with this model — do not use it as-is

1. **Everything above is 2024-era.** No 2026 revision was found. No metric called "CPAI/L" exists.
2. **Finance disputed it on arrival.** Sarah Coyne, Jan 2024: *"Rates havent been 7.55% since May '23."*
3. **It was put on hold.** Sept 2024: *"The logic updates for the computation of expected CPAI is
   currently on hold."* Average ECPAI was hardcoded at **$9,800** from Ankur Jain's manual input.
4. **Lender referral fees were never integrated.** May 2025, Paulo: *"Our pod hasn't had the
   chance to revisit the ECPAI data model to integrate the referral fee logic."*
5. **Lender CPAI excludes HLCS revenue entirely.** *"Lender CPAI is based solely on BBYS revenue
   which is the product of DR Home Value (HL valuation) and BBYS fee rate."* Title/escrow
   contribution profit is only included for **agent** leads.
6. **Revenue uses HL valuation × fee %, not actual sale price × actual fee** — so ECPAI is blind
   to variable pricing, surcharges, discounts, the LO spread, and extension fees.

### Infrastructure

| Thing | Where |
|---|---|
| Table | `homes_closed_orders_cpai` (PK `order_id` / `closed_bbys_lead_id`) |
| Dashboard | Periscope — *Homes Order-Level Expected CPAI* |
| Dictionary | ECPAI Data Dictionary (Google Sheet) |
| Agent Sales columns | prefixed `as_` — `as_interest_expense`, `as_cogs`, `as_cpai`, `as_commission` |

**People:** Paulo Quilao (BI — built it) · Francis Magtibay (migrated to dbt) · Zach Stanko +
Jon Geraci (Finance actuals) · Vanessa Famulener (original definition) · Ankur Jain (manual inputs)

---

## 8. Source of truth — which field to trust

Homes Data Tape definition, validated by Paulo against the `sandbox.sa_backfill_5_8_26` backfill
in June 2026:

```
Total Fee = bbys_lead_fees.actual_program_fee_amount
          + SUM(bbys_extensions.actual_fee_amount)
          + bbys_leads.program_fee_surcharge
          + bbys_leads.program_fee_discount
```

- **Legacy TI+ leads have none of these fields populated** — the source tables are BBYS-specific.
  TI+ requires the static tables `ml_non_static_trade_in_loans` / `non_ml_static_loans`.
- Data Tape population: ~3.4k leads (original loans only, repapered loans excluded, `TEST` prefix
  removed); ~3.0k after excluding failed.
- Sales App now separates **estimated vs. actual** program fee fields, specifically to improve
  CPAI accuracy.

---

## 9. ⚠️ Pending pricing change — verify before relying on it

**Joel Shurtleff, 2026-08-11, `#homes-ops-and-efficiency-pod`:**

> *"based on discussion with Nick F yesterday we will very soon be making **DTI Drop flat fee of
> $2500** and **program period for BBYS 180 days**. Might be good idea to create a new dti drop
> template to account for this that we can replace current standard template with when we go live
> with it, should be before End of month"*

Two changes, targeted before end of **August 2026**:
1. **DTI Drop → flat $2,500** (from 1% / $5,000 min)
2. **BBYS program period → 180 days** (from 120)

Corroborating: Brandi Cirell had a **180-day sell period with no extension fee** approved as a
**pilot** in June 2026.

**Economic read:** a flat $2,500 halves DTI Drop revenue on a $500k DR and cuts it 75% on a $1M
DR. Combined with DTI Drop generating no revenue at IR close, this repositions DTI Drop from a
revenue product to a volume/funnel product — likely a response to FlyHomes' $2,500 flat fee
(already noted as competitive pressure in [[dti-drop]]).

**Status: unconfirmed as shipped.** Verify with Joel / Nick Friedman / Karly.

### 9a. Where the 180-day / variable-pricing idea came from (added 2026-08-29, Nous mining pass)

Source: Nous "Meetings - Tulli" (Granola, read-only) — "Strategy Session," 2026-06-15, and
"LSM & LRM Team Meeting," 2026-06-15. This is **origin-story context for §9's pending change**,
not a new fact about its ship status — it explains the economics *why*, which the Slack quote
above does not.

- **Why 120 days was the cap in the first place:** at the flat 2.4% standard BBYS fee, the
  program is **unprofitable past ~110 days** of carry. The 120-day cap exists specifically to
  stop losing money before that threshold, not as an arbitrary round number.
- **Why 180-day pricing is variable (1.5%→3.6%), not flat:** a flat 2.4% extended to 180 days
  would push the loss further out, not solve it. Scaling the fee **1.5% up to 3.6% across the
  full 180-day period** was the proposed fix — it was explicitly framed as solving three
  problems at once: the economics (fee scales with actual carry cost), the motivation problem
  (a flat long period removes urgency to sell), and transparency (the client sees the fee grow
  with time, rather than a hidden loss to HomeLight).
- **Pilot results (Anirudh Bhutani, ~3 weeks, ending mid-June 2026, partner Richie):** rule was
  "LSMs pitch 180 only when a competitor offers it." **12 competing deals** were won on the
  180-day offer; only **2 fell through**; several are under contract. Read as evidence these
  deals would not have come to HomeLight without the extended offer.
- **The hard constraint, not yet resolved as of the meeting:** HomeLight's **warehouse lines
  currently max out at 180 days with no safety margin** — this is why 180 (not 240+) was
  proposed as the ceiling, and it is the same warehouse-line system flagged as **paused** in
  [[risk-register]] row 12 (Chase/JPM, stated 2026-08-28). No vault source connects these two
  facts explicitly before this addendum, but they are the same underlying capital-markets
  constraint two months apart.
- **Corroboration:** the Gamma leadership deck `g_nir4bt3n8lwxstl` (2026-06-22, read-only,
  via Gamma MCP) independently states dynamic-pricing deals **close ~29% faster** than flat-fee
  deals — same claim, second source, one week later.
- 🔴 **This directly informs the Jul 30, 2026 decision already in
  [[projects/2026-08-29-granola-decisions-6mo]]** ("Stop loan modifications in North Carolina;
  repaper as new loans instead, move to 180-day variable pricing") — that entry records *what*
  was decided; this section is the *why* that preceded it by six weeks.

---

## 10. Partner economics (cost side)

| Partner | Terms |
|---|---|
| **TLS** | Rev share per closing (eff. Mar 2026): <75/mo **$500** · 75–125/mo **$600** · 125+/mo **$750**. Paid to TLS, not individual AEs. Separately charges **25 bps** as a funding cost. |
| **Orchard** | **$400/closing** for 2026. Monthly, Thursday after first Monday. Orchard will **not** do BYOC/DTI Drop — assume max EU on every file. |
| **Elite Lender Program** | **$1,500 LO benefit per deal**, variable pricing with no minimum fee, 1 free extension |
| **Builders** (NVR, Lennar, DR Horton, Pulte) | Selective subsidy / rate credits. NVR framing: "NVR special price – 1% off". Lennar has run a **1.9% promo for 75 days reverting to 2.4%**. |
| **UWM** | ❓ Terms not documented — see open questions |

Channel P&L uses an **$8,500/month notional rep-cost proxy** allocated by application share.
❓ Provenance unknown — no Slack source found. Treat as placeholder.

---

## 11. The 2026 management plan model (added 2026-08-29, from Drive)

**Source:** `Wei's 550 model - Homes 2026 Planning_20260116_v3.xlsx` (owner nicholas.santulli@,
Drive modified **2026-01-16**). Sheet cites an upstream Google Sheet `1-WtnFSCf88PZrRQ...` as the
Base Mgmt Plan feed. This is the first **absolute revenue plan** in the vault — everything prior
was rate mechanics without a denominator.

⚠️ **This is a PLAN as of 2026-01-16, not actuals.** Eight months have run against it. Do not
quote any figure here as performance. [[numbers-that-disagree]] carries the pre-flight rule.

### Headline

| Line | 2026 plan |
|---|---|
| **BBYS program revenue** | **$55,493,698.50** |
| Applications | 17,833 |
| Approvals | 11,948 |
| IR Contracts | 5,716 |
| IR Closings | 4,463 |
| Assumed **avg DR price** | **$575,000** (flat all 12 months) |
| Assumed **revenue per deal** | **$14,000** |

Monthly revenue runs $1.94M (Jan) → $6.70M (Oct) → $4.88M (Dec).

### 🔑 The planned pricing-mix inversion — this is the big one

The model assumes the fee mix **flips in July 2026**:

| Product | Jan | Jun | **Jul–Dec** |
|---|---|---|---|
| Standard BBYS @ 2.4% | 85% | 75% | **20%** |
| Variable Pricing @ 2.2% | 5% | 15% | **70%** |
| BBYS DTI Drop @ 1% | 10% | 10% | 10% |

🔑 **The plan is for Variable Pricing to become the default product in H2 2026, not the
exception.** §2 of this file still frames VP as "now a product feature, not an exception" —
the plan goes further: 70% of volume. Blended headline take-rate under the H2 mix is
**~2.14%** (0.20×2.4 + 0.70×2.2 + 0.10×1.0), down from ~2.31% in January. `(inferred)` from
the mix rows.

⚠️ **Whether this inversion actually happened is unverified.** It is now late August — two
months into the planned 70%-VP regime. Nothing in the vault confirms or refutes it. This is
the single highest-value thing to check against actuals.

### Funnel conversion assumptions

| Step | Rate |
|---|---|
| Application → Approval | **67%** (flat, all months) |
| Approvals **at Target Equity** | 30% (Jan) ramping to **80%** (May onward) |
| Approvals **below TE** | 70% (Jan) → 20% (May onward) — *"HELOC will support this (to launch 2/1)"* |
| Approved-at-TE → IR Contract | 47% (Jan) → **60%** (Mar onward) |
| Approved-below-TE → IR Contract | **30%** (flat) |
| IR Contract → IR Close | **80%** (flat) |
| Application → IRUC (blended) | 23% (Jan) → **33–34%** (H2) |

🔑 **The below-TE conversion penalty is 2:1** — 60% vs 30%. The entire economic case for HELOC
Equity Boost sits in this one row: moving a file from below-TE to at-TE **doubles** its
contract conversion. [[heloc-product]] and [[bbys-equity-boost]] should carry this number.

⚠️ A second, larger IR-closing series ("Wei's buffer," +0% Jan–Mar rising to +45% Sep) sits
alongside the base — 4,463 base closings vs a buffered path reaching 761 in September alone.
The $55.5M revenue line reconciles to the **buffered** series, not the base. Naming which
series you're quoting matters.

### House accounts — 32% of volume is not sold by an LSM

| Source | 2025 actual apps | Plan share |
|---|---|---|
| Orchard | 1,468 | 15% |
| Builder | 1,468 | 15% |
| SRM & others | 196 | 2% |
| **Total house accounts** | **3,131** | **32%** |

Quota methodology, stated verbatim in the sheet: *"Calculated total apps needed with a 25–45%
(seasonal bumps) increase on team IR closing goal"* → *"Less 32% volume from all house accounts
(Orchard, builder, SRM & others)"* → historic pod/IC split → manual adjustment.

🔴 **The plan bakes in 15% Orchard volume. The team no longer works Orchard.** This model was
built 2026-01-16; Orchard, Lennar and D.R. Horton have since left the team's scope. **~30% of
the 2026 plan's application base (Orchard + Builder) rests on relationships that changed after
the plan was written.** Any variance analysis that doesn't back this out will misattribute the
gap to sales execution. Also assumes *"TT will leave to the builders team soon"* — a staffing
assumption, now stale.

### 2025 actuals baked in as the baseline

| Metric | 2025 actual (per this model) |
|---|---|
| LSM-attributed revenue | **$16,768,080** |
| LSM-attributed apps | 6,654 (7,218 incl. pod-4) |
| LSM-attributed contracts | 1,197.72 (modeled at 18% app→contract) |
| Apps→contract conversion assumption | **18%** (2025) → **22%** (2026 quota sheet) |

⚠️ The 2025 contract figures are **fractional** (e.g. 234.90) — they are apps × 18%, not
counted contracts. Do not cite them as observed contract counts.

### Activity model

Daily guidance: **30 calls, 4 meetings** per LSM. Efficiency constants: **0.34 apps per call**,
**4.31 apps per meeting**. Team-level: 270 calls/day → 92 apps/day → 1,832 apps/month; 36
meetings/day → 155 apps/day → 3,106 apps/month.

🔑 **A meeting is worth ~12.7 calls.** The plan's own numbers say the meeting motion is an order
of magnitude more productive per unit, yet the daily guidance weights calls 7.5:1. Stated intent
in the sheet softens this: *"Goal is not to micromanage… Eyes on the prize — IRUC matters the
most. Activities are more of a guidance."*

### 2026 quota shape

Total quota (ex-house-accounts) ramps **703 apps (Jan) → 2,552 (Sep) → 1,636 (Dec)**; 18,796
for the year. Per-pod 2026 distribution and per-LSM monthly targets are in the sheet. Peak
individual monthly quota is ~338 apps (Oct). This is the denominator the LSM accelerator in
[[comp-plans]] §3 measures against — and it varies ~3x across the year.

---

## Open questions

1. **Did the $2,500 DTI Drop / 180-day BBYS change ship?** (§9)
2. **Is Florida variable pricing live or still in build?** (§2)
3. **Is HELOC revenue purely net interest margin?** No origination fee, no annual fee, 0% teaser
   for 6 months on BBYS-attached lines — see [[heloc-product]]. If so, HELOC Boost is a
   deal-enabler cost, not a revenue line, and should be modeled that way in the Channel P&L.
4. **What are UWM's economics?** No rev-share or referral-fee terms found in public Slack.
5. **What is the real fully-loaded cost of an LSM / LRM / CA?** The $8,500/mo proxy has no source.
6. ~~**LSM comp** — is "BIPs of DR volume + IRAX bonus (threshold 200)" still live post-reorg?~~
   **ANSWERED 2026-08-29 → [[comp-plans]].** No: superseded 2026-04-01 by bps-per-Nth-LO-close
   (6/10/13/2) plus a quota accelerator, on a derived "comp-able value" basis. LRMs get 3 bps
   flat. **CAs, Builder reps and SRMs remain unanswered** — no plan document exists for them.
7. **Does HubSpot `bbys_revenue` reconcile** to `bbys_lead_fees.actual_program_fee_amount`?
8. **Is there a current (2026) CPAI formula and named owner?** See
   [[projects/2026-08-17-channel-pnl-finance-readiness]].
9. **Did the planned July 2026 pricing-mix inversion happen?** (§11) The plan says Variable
   Pricing @ 2.2% goes from 15% to **70%** of volume in July. Unverified against actuals. This
   moves blended take-rate ~17 bps and is the largest single unverified assumption in the model.
10. **Has anyone rebuilt the 2026 plan without Orchard/builder volume?** (§11) ~30% of the
   planned application base sits on relationships the team no longer works.
11. **Which IR-closing series does Finance use** — the 4,463 base or the "Wei's buffer" path the
   $55.5M revenue line actually reconciles to? (§11)

## Related Notes

- [[bbys-overview]]
- [[dti-drop]]
- [[heloc-product]]
- [[partners]]
- [[bbys-edge-cases]]
- [[projects/2026-08-17-channel-pnl-finance-readiness]]
