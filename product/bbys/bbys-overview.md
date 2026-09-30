---
last_updated: 2026-09-29
type: context
source: HomeLight-Vault/context/bbys-overview.md
imported: 2026-09-29
---

# Buy Before You Sell (BBYS) — Product Overview

> High-level overview of the BBYS product for Claude context. Fill in details as you go.
>
> Related: [[homelight-homes]] · [[team]] · [[hubspot]] · [[partners]] · [[heloc-product]] · [[dti-drop]] · [[lo-lifecycle]] · [[systems-map]] · [[bbys-edge-cases]] · **[[bbys-unit-economics]]** (all fee, revenue and cost mechanics)

## What Is BBYS?
HomeLight's Buy Before You Sell program allows homeowners to purchase their next home before selling their current one. This eliminates the contingency problem and makes offers more competitive.

## How It Works (High Level)
1. Homeowner applies through a Loan Officer partner
2. HomeLight evaluates the current home
3. HomeLight provides financing to purchase the new home
4. Homeowner moves into the new home
5. The old home is listed and sold on the open market
6. Transaction settles

## Distribution Channel
- **Loan Officers (LOs)** are the primary referral source
- **Lending Sales Managers (LSMs)** manage LO relationships
- LOs are organized under **mortgage companies**

## Key Metrics to Track
| Metric | Current Value | As Of | Notes |
|--------|---------------|-------|-------|
| April IRUC target | 330 | 2026-04-09 | Up from 300 Q2 baseline. Wei pushing hard |
| April IRUC pace | 59 (pacing 195) | 2026-04-09 | Well behind 330 target. Slower start expected; strong finish planned |
| April apps | 317 (pacing 951) | 2026-04-10 | Jake's Friday dashboard. Target: 1,450 total across channels |
| April contracts | 69 (pacing 200) | 2026-04-10 | |
| Weekly meetings | 85+ (first time since Jan) | 2026-04-10 | Kara & Brian leading. Mostly self-generated from outbound calls |
| LSM daily target | 30+ dials + Zoom on every call | 2026-04-09 | Credit score workaround: full Credit Karma reports showing 620+ |
| App cutoff | April 22-ish | 2026-04-06 | Apps after the 28th won't convert in time for April IRUCs |
| Team weekly calls | 1,450 | Post-quota (started 2026-01-21) | Up from 717 pre-quota (~80% increase) |

## LO Funnel (from Qualification System analysis)
| Stage | Count | Notes |
|-------|-------|-------|
| Total leads in 6-month window | 6,322 | All LO contacts entering pipeline |
| LOs who never close a deal | ~71% | Current state — no pre-qualification |
| Projected SQLs (with qualification) | ~2,213 | 35% pass rate after scoring |
| SQL→App rate (base case) | 20% | Breakeven at 19% |
| Current yield (leads→closings) | 1.6% | |
| Projected yield (with system) | 5.0% | |

## Terminology
| Term               | Meaning                                                               |
| ------------------ | --------------------------------------------------------------------- |
| BBYS               | Buy Before You Sell                                                   |
| LO                 | Loan Officer                                                          |
| LSM                | ⚠️ Lending Sales Manager (as written here) — but three expansions are observed vault-wide; canonical is **Lender Success Manager** (the roster/`job_title` value). See [[glossary]] collision section. |
| LRM                | Lender Relationship Manager                                           |
| IR | Incoming Residence — the home being *purchased* |
| DR | Departure Residence — the home being *sold* |
| EU | Equity Unlock — equity from DR advanced to fund IR purchase |
| EUC | Early Use of Cash — clients who drop out early |
| IRUC | IR Under Contract (stage 998755445, also "IR In Escrow") |
| IRX | IR Closed (stage 998755447) — the primary close milestone |
| DRX | DR Closed (stage 998815822) — the final close |
| SQL | Sales Qualified Lead (LO who passes qualification scoring) |
| MQL | Marketing Qualified Lead (engagement score ≥ 50) |
| TPD | Transactions Per Day (cohort performance metric) |
| SRM | 🔴 **Corrected 2026-08-29** — this row previously read "Suppression list — contacts not to be reassigned," which is **wrong**. **SRM = Strategic Relationship Manager**, a real role that owns AGENT relationships (small number of LO contacts too), appearing in the `srm` owner field on deals. "SRM Suppression" is a HubSpot list *named after the role* (contacts an SRM has claimed, protected from auto-reassignment) — not a second meaning of the acronym. See [[team]] §"Strategic Relationship Managers (SRMs)" and [[glossary]] for the full correction and source trail. |
| EB | Equity Boost — additional lending beyond standard EU |
| HELOC | Home Equity Line of Credit — new product launched 2026-03-04 |
| Lender Direct Line | Aircall phone number for inbound LO calls |
| BIPs | Basis points of DR volume — LSM comp metric |
| IRAX | IR Close bonus threshold — LSM comp metric |
| BYOC | "Bring Your Own Cash" — **legacy name for [[dti-drop]]**. Still the flag name in Sales App, agreements, and email templates. Same product. |
| TI+ | Legacy "Trade-In+" — the pre-BBYS product, retired when BBYS deployed in early 2024. Separate `trade_in_leads` table, `user_type = cc_trade_in`, none of the BBYS fee fields. A recurring source of report count mismatches. |
| VP | Variable Pricing — fee scales with days-to-DR-sale. Now a Sales App "Pricing Model" field, not just an exception. See [[bbys-unit-economics]] §2. |
| CMF | Closing Management Fee — $1,450, waived if the seller uses HLCS |
| HLCS | HomeLight Closing Services — title/escrow arm |
| HLHL | HomeLight Home Loans — originates the HELOC |
| UGA | Upside Guarantee Agreement — post-program-period, HL buys at LPV, relists, remits net profit to client |
| LPV | Loan Payoff Value — the figure HL purchases at under the UGA; appears in the econ model |
| PO | Payoff (demand) — the itemized payoff issued at DR close |
| DOM | Days on Market — drives the interest-expense day count in CPAI |
| CPAI | Contribution Profit After Interest (**not** cost per application) — Finance/BI owned |

## Deal Pipeline Stages (BBYS Applications — ID 681595694)

| Stage | Stage ID | Milestone? | Notes |
|-------|----------|-----------|-------|
| New | 998755441 | | Application submitted |
| In Review | 998755442 | | Being reviewed by ops |
| Approved | 998755443 | Key | Valuation approved, agreement pending |
| Agreement Signed | 998755444 | Key | Client signed BBYS agreement |
| IR In Escrow | 998755445 | Key | IR under contract. **Operational note:** LRMs/LSMs author the [[2026-05-12-ca-handoff-slash-command|CA Notes]] handoff message FIRST and then move the stage to IRUC afterward — so `/ca-handoff` regularly fires on deals still at Approved (998755443) or Agreement Signed (998755444) with no IR contract uploaded yet. See [[2026-05-14-ca-handoff-pre-iruc-workflow]]. |
| Clear to Fund | 998755446 | | Ready for funding |
| IR Closed | 998755447 | Close | 1st milestone close — `ir_closed_date` |
| DR Closed | 998815822 | Final | DR transaction closed |
| Nurture | 998815823 | | On hold / future conversion |
| Failed | 998815824 | Terminal | Deal fell through |

## Key Deal Properties

| Property | Purpose |
|----------|---------|
| `ir_closed_date` | IR closed date — comp period trigger. Presence = "contract" for quota |
| `lo_bbys_irx_number` | DEPRECATED — **0% populated** as of 2026-09-21. The Comp App derives IRX from deal history. `lo_s_irx_count` is also dead (0.2%). Neither field is reliable for IRX counts. See [[projects/2026-09-21-bbys-hubspot-field-description-audit]] Finding 5. |
| `bbys_opportunity_source` | "Agent" → 25% comp discount, "Client (D2C)" → $0 comp, "Lender" → full comp |
| `hl_deals_lp_bbys_lo_est_dr_value` | LO estimated DR value — primary compable value source |
| `homelight_value` | HL value — fallback compable value (× 1.05) |
| `est_compable_amount` | Estimated comparable comp amount |
| `bbys_revenue` | Revenue generated from this deal |
| `bbys_approval_velocity` | Speed from submission to approval |
| `lsm_collaborator` | Owner ID of assigned LSM |
| `lender_relations_manager` | Owner ID of assigned LRM |
| `partner_slug` | Partner slug — used for deterministic partner matching |
| `wholesale_lender` | Wholesale lender on the deal |

## Q1 2026 Results (Record-Breaking)

- **March: 216 IRUCs** — First time ever breaking 200 in Q1, up 21% YoY
- 35 contracts in final 2 days (16 Monday, 19 Tuesday)
- ~1,050 apps in March (up from ~700 in February)
- 2025 total: 701 transactions

## Q2 2026 Targets

- **1,450 total apps** across all channels
- **330 IRUCs** in April (raised from 300 baseline)
- **200+ contracts/month**
- Conversion rate concerns: SQL-to-App 32% (down from 36%), Approval-to-Agreement 30% (down from 41%)
- Risk: LSM KPIs shifted to emphasize app volume over lead quality vetting — historical precedent (Orchard partnership) dropped conversion from 50% to high 30s

## Q2 2026 Results (actuals, as of 2026-06-02)

| Month | Apps | IRUCs | Target | Attainment |
|-------|------|-------|--------|------------|
| Apr | 1,145 | 278 | 381 | 73% |
| May | 1,123 | 267 | 525 | 51% |

- Attainment running 51–73% of target all year; the gap is *widening* because quotas ramp far faster than app volume (May target nearly 2× April's while apps were flat).
- Forecast model (Data Bridge `/reports/bbys-forecasting`) is accurate (13-mo backtest MAPE ~17.6%) but was biased ~20 IRUCs/month LOW; PR #320 bias-corrects it, fixes the month-end intra-month under-read, persists daily snapshots, and adds operator forward-demand overrides. See [[2026-06-02-bbys-forecast-may-review-and-accuracy]] + [[2026-06-02-bbys-forecast-bias-correction]].

## Growth Channels

### UWM Partnership ("Pot of Gold" for 2026)
- Generating ~20 apps/day (pre-April baseline)
- Weekly Thursday trainings
- Each LSM getting 70-80 UWM AEs assigned; need to assign 200+ more AEs (~20-30 per LSM) — overdue as of Apr 11
- AE relationship trumps existing LO assignments (key decision)
- 50/50 commission split for first 3 conflicted deals
- **April status:** 3 contracts, targeting 30 with 15 business days left. Need 1-2 apps/day → 3-4 to hit +25 IRC bridge goal
- **50 bps discount promotion** running through ~Apr 14 (next Tuesday)
- **UWM Forum (Apr 7-8):** Jake attended. Night 1: strong turnout. Night 2: poor (no confirmation process). Generated at least one live lead. Impressed by UWM's internal all-hands energy
- **Marketing:** 6 billboards near UWM office highways proposed. Social/Google/LinkedIn campaigns. Early May event post-board meeting. Jake skeptical on billboard ROI tracking
- **UWM AE distribution SOP v3** needs finalizing

### Builder Channel (Explosive Growth)
- 2-3x more leads in one month than entire period since Nov 2023
- DR Horton expanding to 32 communities in mid-Atlantic
- Pulte deal under contract
- NVR contract nearing final turn
- Nick Plamondon started as Head of Builder Relations (~Apr 13)
- **April estimate:** Conservative +20-25 IRUCs from builder channel
- **Builder client welcome email:** Separate SendGrid email being built with photo upload URL. HubSpot filter needed to exclude builder leads from standard welcome emails. Target: completed by Apr 10
- **Top Flite → Flite Financial rename:** Update needed in Sales App and HubSpot (flagged by Brad Barton Apr 10, Tejas coordinating)

### HELOC Equity Boost (Launched March 4)
- First funded deal: client received extra $65K that made deal possible
- **Prequalification rate: ~92%** (as of Apr 10, excluding TX new homes and ≤10% down payments)
- **Average prequalified amount: $82,794** (up from ~$80K)
- **Utilization rate: ~7.6%** (stuck, barely moved from prior week — top priority to increase)
- CLTV for prequalified+approved: 81.62% vs 82.79% LO target — good indicator product meets equity needs
- Drew's target: Close 100 HELOC cards before May 15
- **HELOC Playbook** released (Google Doc) + AI chatbots on Gemini and ChatGPT — well received by sales team
- **HELOC Optimizer tool** being integrated into LO Portal (Oliver building frontend, Pavel building backend). Target: before end-of-month webinar
- **HELOC payment calculator** added to LRM tool
- **Deb error (Apr 8):** Double-counted HELOC in equity boost calc — pitched client $850K vs actual $127K capacity. Flagged as training gap
- Webinar planned end of April with Mette to feature HELOC Equity Boost
- Marc visiting LRMs in person next week to help them pitch the product

## Underwriting & Buy Box (July 2026)

Canonical DR underwriting rules — internal only:

- [[2026-07-01-bbys-dr-underwriting-buy-box-source-of-truth]] — national buy box, CLTV limits, eligible/ineligible property types, state list
- [[2026-07-01-florida-bbys-full-reopening]] — Florida full relaunch, county CLTV zones (70%/75%), coastal and property-type restrictions, 2.9% fee
- [[2026-07-01-florida-zip-county-cltv-reference]] — ZIP-level CLTV lookup (1,495 FL ZIPs)

## Related Notes
- [[bbys-unit-economics]] — fee structure, variable pricing, extensions, cost of capital, CPAI
- [[hubspot]]
- [[team]]
- [[partners]]
- [[q2-priorities]]

---

## 2026-08-28 — code-verified product facts

Added from the six-repo documentation pass. Where this section conflicts with anything
above, **this section is newer**. Full detail: [[repo-hapi]], [[repo-homes-fe]],
[[bi-metrics-definitions]], [[bbys-integration-map]].

### The three products and their CLTV ceilings

| Product ID | Max CLTV |
| --- | --- |
| `base-unlock-70` (default) | **70%** |
| `equity-boost-85` | **85%** |
| `heloc-90` | **90%** |

`maxCltvForProduct()` in `homes-fe` **defaults unknown product IDs to 90%** — the most
permissive ceiling.

Plus **DTI Drop** (contingency-removal-only), which is priced and papered separately —
`contingency_removal_only` on `bbys_leads`, `DTIDA` agreement.

### Pricing constants (authoritative, from `bbys_lead.rb`)

| | Value |
| --- | --- |
| Program fee | **2.4%** |
| Florida program fee | **2.9%** |
| Contingency program fee | **1.0%** |
| Minimum fee | **$9,000** |
| Contingency minimum fee | **$5,000** |
| Maintenance reserve | **3%** |

⚠️ The `homes-fe` calculator uses **2.4% with no Florida variant** — FL quotes from the
calculator understate the fee.

⚠️ **DTI Drop is being repriced right now** to a **$2,500 flat fee** with a **180-day
program period** (decided 2026-08-11), but $2,500, $3,500, and $5,000 are all being applied
in live deals as per-deal exceptions. A **2.25%** tier also appears in dynamic-pricing
planning but is not in the code. See [[2026-08-28-vault-refresh-slack-findings]].

### Calculator assumptions (the numbers behind the pitch)

| Input | Default |
| --- | --- |
| **BBYS selling premium** | **4%** — what a BBYS seller gets above market |
| **Contingent purchase premium** | **2%** — what a contingent buyer pays |
| Selling costs | 6% |
| Target max DTI | 45% |
| Interest rate | 6.99% |
| Loan term | 30 years |
| Closing costs | $10,000 |

### The two-channel model

| Channel | Route | Pre-lead endpoint? |
| --- | --- | --- |
| **`provider`** (LO- or partner-initiated) | `/application-flow` | ❌ |
| **`client`** (client-initiated) | `/bbys-intake` | ✅ **yes** |

Client-direct splits into four variants, each with its own HubSpot source label:
**`Builder`** · **`Client Direct`** · **`Lender Direct`** · **`Bank`**. Only `builder` gets
Slack notifications, its own comms journey, and a client confirmation template.

> 🔑 **Client-channel leads exist as pre-leads before they become `bbys_leads`.** Counting
> only `bbys_leads` undercounts client-direct demand.

### The credit box (authoritative)

`hapi/config/bbys_deal_failure_reasons.yml` is **the most precise statement of the BBYS
credit box that exists anywhere** — more current than any deck or SOP.

**Ineligible property types:** mobile/manufactured · non-residential · vacant land ·
high-rise condo · multi-family.
**Florida-specific:** out of service area · within 2 miles of coast · townhome or condo.
**Also ineligible:** flood zone · lot > 5 acres · under 700 SF · over 5,500 SF ·
age-restricted community · other deed restrictions · unpermitted addition · property
condition · HOA fees too high · under minimum pricepoint.
**Marketability declines:** lack of similar comps · active DOM too high · expected DOM too
high · new build saturation.

Every one of these sends an automated LO failure email.

**Borrower credit minimum: 620.** Equity Boost bands: Poor (<620) · Fair (620–679) ·
Excellent (680+).

**Massachusetts is blocked from extension emails** for regulatory reasons
(`EXTENSION_EMAIL_REGULATORY_BLOCKED_STATES = ["MA"]`).

**New York:** DTI Drop is offered, but there is **no 0% bridge**.
**Hawaii:** HELOC unavailable; Asset Equity Boost still proceeds.

### The operational competitor set

From the `lost_to_competitor` picklist — narrower and more current than
[[competitors-bbys]]:

**Knock · UpEquity · Homeward · Calque · Lendsure · FlyHomes · Zavvie · Cash for Keys ·
Smart Move Guarantee · QuickBuy · Traditional Bridge Loan**

Plus **"Loan officer lost client to different LO"** — losing the LO is tracked as a
competitive loss. That is a distribution-channel signal, not a product one.

*(The legacy sales-app list also carries `lennar`, `orchard`, `lower`, `tls`, `ultra`,
`regular_mortgage` — these do **not** appear in the new taxonomy.)*

### ⚠️ Partnerships the team no longer works with

As of 2026-08-21: **Orchard, Lennar, and D.R. Horton** are being removed from Round Robin.
Orchard still has active post–July 2025 leads with special email CC rules.

### ⚠️ Reporting cautions

- **Two failure taxonomies are live at once** — the legacy flat list and the v2 YAML tree.
  Any failure trend spanning the v2 rollout compares two schemas.
- **The failure *category* is never stored** — only reason and sub-reason.
- **`nurture` counts as failed**, not active.
- **`agreement_signed` is often skipped** — low counts there are a data artifact.
- **Production dates are partly synthetic** (proxy dates, fallbacks, hardcoded overrides).
- **`equity_boost = true` may be system-set**, not LO-chosen — unconfirmed.
- Always name the **stage scale**, **channel filter**, **product filter**, and **snippet
  version** when quoting a number. See [[bi-metrics-definitions]].

---

## 2026-09-25 — Public lender-facing claims (lenders.homelight.com)

What HomeLight **publicly** tells lenders today. Safe to reuse in external marketing, because it is already
published. Captured while building the [[projects/2026-09-25-bbys-lender-ad/README|lender ad video]].

- Headline: "Eliminate your client's home sale contingency, reduce their DTI ratio, and unlock up to **90% CLTV**"
- "Unlock up to **$2M** of your client's equity with **0% interest**"
- "Pre-qualify your client in **24 hours or less**" (no fee or commitment)
- Four steps: pre-qualify → non-contingent offer → close new home + mortgage (equity unlock = down payment) → sell old home for full market value
- **Coverage:** BBYS is available nationwide **except Alaska, Massachusetts, and New York**; those states get DTI Drop instead. **DTI Drop is available in all 50 states.**
- Social proof: **22k+ top loan officers · 28k+ top real estate agents · $884M+ total equity unlocked**
- Lender CTAs: `lenders.homelight.com` (register form asks for NMLS ID, role, employer, licensed states), a submit-client link (`equity.homelight.com/bbys/new?flow=lenderSubmission`), and a demo-booking link

⚠️ The **620 credit floor**, **no appraisal**, and the **2.4% fee** are *not* on the public lender page. Treat them as sales-conversation facts, not ad copy, until Compliance says otherwise.

---

## 2026-09-28 — Drivers study (regression, Jan 2025 → Sep 2026)

From [[projects/2026-09-28-bbys-regression-analysis]] (19,639 apps; core = ex-Orchard/Lennar/DHI). Report: https://claude.ai/artifact/WnqqsjGVgMqXXngkWXuHKF

| Metric | Value | Notes |
|---|---|---|
| Core apps per active LO per month | ~1.2–1.3, flat 20 months | Volume growth = more LOs, not more per LO |
| YoY IRUC change, core cohorts Jan-25→May-26 | +463 = +589 volume, −104 conversion | Shift-share |
| App→IRUC within 180d (KM) | W/B 28% · Retail 26% · Builder/Client 22% · Orchard 11% | Orchard approves most (78%), stalls at agreement |
| Median App→Approval / Approval→IRUC / IRUC→IRX | ≤0.4d / ~6–7d / ~24d | Flat six quarters; faster approval does not convert better |
| LO repeat rate (days 45–180) | 32.7% if first deal hits IRUC ≤45d vs 20.5% if approved-but-stalled | +69% apps next year; denial no worse than a stall |
| LO concentration (TTM) | 62% of active LOs sent 1 app; top 10% = 38% of apps | |
| Rep conversion spread after case-mix | 0–1.6 pp (none significant) | Reps differ in reach (22–144 apps/mo), not conversion |
| ~~Core approval rate~~ | ~~~54% → ~71% (Q1-25 → 2026)~~ | ⚠️ **Superseded by v2:** trough-vs-peak comparison; same-season approval moved only 0 to +3 pp and marginal approvals do not explain the Approval→IRUC softening (0 to −6 pp YoY). See v2 section below. |

## 2026-09-28 (late) — Drivers study v2: tribunal-verified numbers

Supersedes the v1 table above where they differ. Source: [[projects/2026-09-28-bbys-regression-analysis]] ·
`exports/2026-09-28-bbys-regression/deep/tribunal/signal_register.csv` (232 claims; only SIGNAL shown unless noted).

| Metric | Value | Tier / note |
|---|---|---|
| Source of 2026 YTD core growth (+1,463 LO-attributed apps, +27%) | ~86% from LOs on file at Jan 1 (more submitting + dormant returning); new recruits ~11%; per-LO depth ~4% | SIGNAL (w3_lo_stock_flow#0). v1's r=0.954 "reach" stat is an artifact by construction |
| Post-approval leak (core) | 58–60% never reach IRUC, flat 9 quarters (~375–390 deals/mo) | SIGNAL; rep follow-up recovers ~1–2% (PROBABLE) |
| Requested CLTV | Above 70: ≈ −8 pp approval, ≈ −6 pp App→IRUC per +10 pts | SIGNAL |
| DTI Drop at CLTV > 70 | ~~Approval cliff, confirmed out of sample (Sep-2026)~~ | ~~SIGNAL → route as an intake rule~~ **Corrected 2026-09-29 (v3):** fully explained by no borrowable equity at intake; once equity is known the DTI Drop label adds nothing (−2 pts). Fix = day-one equity-gap call + LO confirmation before DTI Drop is auto-selected, not a routing rule. |
| LO value vs zip ZHVI | Far above local value → −5 to −10 pp approval | SIGNAL |
| Agent on deal at intake | +7–12 pp contract rate, within LO (readiness marker) | SIGNAL |
| First deal to IRUC ≤ 45 d | +11 pp LO repeat (days 45–180); ~24% of first deals hit it | SIGNAL (association) |
| IRUC → IR close | 78% within 90 d, median 24 d; nothing on contract day predicts fall-out | SIGNAL / NULL_CONFIRMED |
| Gross fee per core IR close | ~$14.0k (≈ $10.9k per core IRUC) | SIGNAL |
| Orchard / Lennar / DHI exit | ~22% of apps but ~11–13% of IRUC/fee: ≈ 34 IRUC & ~$293k gross fee/mo | SIGNAL |
| CrossCountry | Approval→IRUC −10 to −16 pp since 2025Q3, within LO | SIGNAL |
| UWM | ~65–105 core apps/mo gross; net-new share unidentified | SIGNAL (gross) / UNVERIFIABLE (net) |
| Whole lever portfolio | ≈ 35 IRUC / ~$380k gross fee a month central (overlap-adjusted), none proven causal | Sizing: `deep/tribunal/LEVER_SIZING.md` |

**Withdrawn v1 claims:** DTI/EB post-approval "rescue" (+29/+21/+20 pp) — post-outcome writes; approval 54→71%; event
effects (none beats placebo); "37% uncategorized failures" (integration blanking); "DR close booking stalled" (live since 7/2025).


## v3 verified metrics (2026-09-29)

Supersedes the v1/v2 tables above where they differ. Source: [[projects/2026-09-28-bbys-regression-analysis]] ·
`exports/2026-09-28-bbys-regression/deep/v3/REPORT_V3.md` · report https://claude.ai/artifact/WnqqsjGVgMqXXngkWXuHKF (v5).
Core = LO-channel deals excluding Orchard and the exited builders Lennar and D.R. Horton. Data through 2026-09-27.

**Funnel per month (core; counts by the month the step happened):**

| Step | Jan–Aug 2026 / mo | Jun–Aug 2026 / mo | Conversion to next |
|---|---|---|---|
| 1. Application | 794 core (1,036 all-BBYS) | 943 core (1,184 all) | 68% of Mar–May 2026 core apps approved within 30 days; 9% denied; ~23% withdrawn or quiet |
| 2. Approval | 542 | 651 | 40% of Mar–May 2026 approvals reached a new-home contract within 120 days (biggest leak) |
| Denial | 83 | 106 | ~11 of every 100 apps (all channels) |
| 3. Agreement signed | 224 | 262 | Agreement and contract happen together (median 0 days apart); 44% of approved clients never get the agreement |
| 4. IRUC | 208 | 246 | 80% of Jan–May 2026 core contracts reached IR close within 90 days (planning value 0.78) |
| 5. IR close | 157 (~$2.2M gross, ~$1.1M net) | 193 (~$2.7M gross, ~$1.35M net) | ~22% of old homes still unsold 120 days after IR close |
| 6. DR close | 132 | 154 | Median 76 days IR close → old-home sale (2025 IR closes) |

| Metric | v3 value | Note |
|---|---|---|
| Fee per core IR close | **~$14.0k gross**; net ≈ 50% of gross (44–55%) | ≈ $10.9k gross per contract; ≈ $2.9k gross per core app. Not Finance-recognised revenue |
| Growth split of the +1,463 LO-attributed core apps (Jan 1–Sep 27, +27% YoY) | **57 / 29 / 11 / 4** = more already-active LOs submitting / dormant LOs returning / more brand-new LOs / more apps per LO | Flow split. The cohort split of the same +1,463 is 86% from LOs on file at Jan 1 vs 14% first seen in 2026 — see [[numbers-that-disagree]] row 44. ~17 pts of the 27% was a one-off 2024-cohort comeback wave |
| 2027 central case (no new partners or programs) | **~+12% → ~10,400 core apps**; ~229 core contracts/mo; ~$15.0M net fee | North Star (500 contracts/mo) is ~270/mo above this |
| CrossCountry contract rate (approved → contract within 90 days) | **32 per 100** since Jul 2025 vs **47 per 100** before; rest of LO ~40 | CrossCountry-specific, first 30 days after approval; costs $35–63k gross ($18–32k net)/mo |
| Capital constraint | A 615-contract month needs **~$205M** of equity outstanding at the January peak vs **$131M** today | Largest credit line ($150M JPM) paused after 2026-08-24 |
| HLCS title attach (service states outside Orchard) | **46% → 32%** (Jul 2024–Mar 2026 vs Apr–Sep 2026) | ~half aligned with listing-specialist mix; Finance revenue fell too |
| Post-approval leak (core) | 59.7% of approvals (2,941 of 4,926, Jun 2025–May 2026) never contract in 120 days | ~245/mo across that cohort, ~280–300/mo over the last 12 months, 390–420/mo at the summer peak; decided in the first 1–2 weeks |
