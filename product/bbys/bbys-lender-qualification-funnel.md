---
last_updated: 2026-05-19
type: context
source: lead_funnel_and_lead_scoring repo + Q2 2026 executive deck + HubSpot rollout notes
status: active rollout — MQL gate live Apr 28; priority/re-MQL/win-back/unresponsive workflows live or in QA as noted
source: HomeLight-Vault/context/bbys-lender-qualification-funnel.md
imported: 2026-09-29
---

# BBYS Lender Qualification Funnel — MQL / SAL / SQL Revamp

> Operating-model reference for the new BBYS Lender Qualification funnel. Claude reads this for any work that touches MQL/SAL/SQL routing, lead scoring, the `[BBYS] Lender Qualification` pipeline, Sales Responsiveness, P-tier priority, re-MQL, win-back, or any HubSpot/Data Bridge automation that creates, qualifies, pauses, or disqualifies a Lead.
>
> Related: [[data-bridge]] · [[hubspot]] · [[hubspot-workflows]] · [[lo-lifecycle]] · [[bbys-overview]] · [[partners]] · [[modex-scrapers]] · [[systems-map]] · [[2026-03-26-lo-qualification-system-justification]] · [[2026-04-23-data-bridge-improvements]] · [[2026-05-15-bbys-lead-funnel-revops-sop]] · [[2026-05-15-bbys-lead-funnel-lsm-sop]]

## TL;DR

The old "any engaged LO = MQL, any actioned engagement = SQL" model has been replaced with a clean three-stage funnel — **MQL → SAL → SQL** — built on a single `[BBYS] Lender Qualification` Lead pipeline, a 60-point Contact-level MKT Score gate, lifecycle guardrails, a Lead-level P0-P5 priority queue, and a separate mechanical Sales Responsiveness label (Hot / Warm / Cold / Unresponsive). Fewer MQLs, much higher trust, every Lead has a priority/SLA expectation, and current customers / active BBYS Deals route to expansion or deal follow-up instead of net-new qualification.

**Current rule:** the hourly SLA proposal from the original deck has been superseded. The live operating language is daily: **P0 = drop everything, P1 = same day, P2 = next day, P3 = end of week, P4 = best effort, P5 = nurture only.** P0 sits above the standard matrix and is reserved for leadership-flagged or highest-intent escalations.

**Source-of-truth hierarchy:** when this note conflicts with older slide commentary later in the file, use the newer repo-backed sections immediately below. The slide transcript remains useful for executive framing, but the live operating model is governed by the `lead_funnel_and_lead_scoring` repo and HubSpot workflow rollout notes.

---

## Current Operating State — May 15, 2026

| Component | State | What to know |
|-----------|-------|--------------|
| Bowtie classification | Live | W-A / W-B / W-C keep `lender_journey_stage` and `lender_lifecycle_phase` aligned to ICP, Lead state, deal activity, and dormancy. `lifecyclestage` is not trusted for BBYS funnel routing. |
| Contact MKT Score | Live | HubSpot Score property `[Lenders] Marketing Score` / `lenders_marketing_score_total`. Runs for Lender contacts in Pre-Qualification, Qualification, and At Risk; Land/Expand customers are excluded. |
| MQL gate | Live since 2026-04-28 | Score >= 60 plus guardrails creates a Lead in `[BBYS] Lender Qualification` (`866607668`). Launch produced ~1,771 net-new MQLs. |
| Unified Lead pipeline | Live | Pipeline `866607668`; New stage `1297783695`. Legacy pipeline `735605687` and archived pipelines remain read-only / historical. |
| Data Bridge Lead Funnel mirror | Shipped 2026-05-19 | HubSpot Leads object `0-136` for pipeline `866607668` now mirrors into `hub_leads`. Admin CRM has `/hub/leads`; Priority Queue Loan Officers mode reads `/queue/loan-officer-leads` and ranks by `lo_lead_priority`. See [[2026-05-19-data-bridge-lead-funnel-mirror-and-queue]]. |
| MQL Trigger | Live since 2026-04-28 | Real-time score-crossing workflow. Creates net-new Leads only when no active Lead, no active BBYS Deal, no recent archived Lead, and no first BBYS app date. |
| Priority Matrix Writer | Live 2026-05-13 | Writes `lo_lead_priority` once at Lead creation. Sales sees the P-tier, not the raw score. |
| Re-MQL Sweep | Built / go-live 2026-05-13 | Daily scheduled path for contacts still score-qualified after the 90-day cooldown. Creates `hs_lead_type = "[Lender] Re-qualification"`. |
| Win-Back MQL Trigger | Built / go-live 2026-05-13 | Parallel score-crossing path for At Risk / Churned contacts, including contacts with prior applications. Creates `hs_lead_type = "[Lender] Re-engagement"` and writes Re-engaging classification. |
| Sales Responsiveness Label | Live (confirmed 2026-08-10 portal sweep) | Custom Code computes `lo_sales_responsiveness_points`, writes `hs_lead_label`, stamps SAL fields, and handles Unresponsive Auto-DQ. Previously documented as test mode; 10 Aug sweep confirmed live enrollment. |
| Unresponsive Sweep | Live 2026-05-13 | Daily backstop that moves eligible Cold Leads to Unresponsive / Auto-DQ when age and outreach thresholds are met. |
| Event score | Live via n8n | Daily 02:00 UTC event decay writes `lead_scoring_calculation_event_score` and `lo_event_score`; scoring rules read `lead_scoring_calculation_event_score`. |
| Dedicated event bypass | Target, not fully built | Conference / booth / raffle / dinner should bypass score where appropriate, but the dedicated event-trigger workflow is still future work; today events mostly flow through event score unless manually handled. |
| Numeric Sales Score | Deferred | Do not build until categorical Sales Responsiveness has at least 30+ days of data and preferably the 60-day checkpoint. |
| Primary tuning | Scheduled ~2026-07-28 | Threshold and weight tuning wait for 90-day post-launch outcome data. Earlier checkpoints are plumbing / distribution checks only. |

## Core Architecture

The system has five layers:

1. **Bowtie lifecycle** on the Contact — one `lender_lifecycle_phase` at a time: Pre-Qualification, Qualification, Land, Expand, At Risk. It describes where the LO is in the BBYS relationship, not whether a single Lead is active.
2. **MKT Score** on the Contact — persistent, population-level score used to decide who deserves a sales conversation right now.
3. **MQL Trigger / guardrail layer** — score >= 60 or approved exception signal enters the Lead-creation decision tree.
4. **Unified Lead pipeline** on the Lead object — one active queue for all BBYS lender qualification work.
5. **Sales Responsiveness** on the Lead — per-cycle Hot / Warm / Cold / Unresponsive signal after sales engagement starts.

The most important principle: **Contact properties describe the LO; Lead properties describe this qualification cycle.** A Contact can produce many Leads over time. The MKT Score persists across cycles; the Sales Responsiveness label resets each cycle.

## Contact-Level Model

| Purpose | Canonical field | Notes |
|---------|-----------------|-------|
| Scope | `hl_contact_type = "Lender"` | Non-Lender contacts are out of scope. |
| Owner | `hubspot_owner_id` | Lead inherits this. Fallback owner is Nick Santulli (`656430767`) if blank. |
| Bowtie phase | `lender_lifecycle_phase` | Canonical phase truth. Do not use `lifecyclestage` for this funnel. |
| Granular state | `lender_journey_stage` | 17-value stage enum: No MODEX Data through Re-engaged. |
| Persona / ICP | `lender_icp` | Challenger, Entrepreneur, Steady Eddy, Learner, Enterprise - Poor Fit, Excluded. |
| MKT Score | `lenders_marketing_score_total` | Display label `[Lenders] Marketing Score`. Score >= 60 triggers MQL evaluation. |
| Event score | `lead_scoring_calculation_event_score` | Written by n8n. This is the property the HubSpot score rules read. |
| First app guardrail | `first_bbys_application_date` | Net-new MQL Trigger skips contacts with a known first app. Win-Back handles At Risk post-app population. |
| Reach-out count | `lo_reach_out_count` or Lead-native counters depending workflow | Used in responsiveness / Auto-DQ logic. Verify which native counter the specific workflow reads before debugging. |

## Lead-Level Model

| Purpose | Canonical field | Notes |
|---------|-----------------|-------|
| Pipeline | `hs_pipeline = "866607668"` | `[BBYS] Lender Qualification`. |
| New stage | `hs_pipeline_stage = "1297783695"` | First stage at creation. Other stage IDs should be pulled from HubSpot when needed. |
| Sales responsiveness | `hs_lead_label` | Blank at MQL creation. Set at SAL transition based on activity signals. |
| Priority | `lo_lead_priority` | P0-P5. This is the only number sales needs to work the queue. |
| Priority timestamp | `lo_priority_last_computed_at` | Audit field written by Priority Matrix Writer. |
| MQL attribution | `lo_qualification_funnel_mql_channel`, `..._activity`, `..._source_detail` | Last-click within prior 30 days. `Fit-Only` means the score moved on non-behavioral fit changes. |
| MQL origin | `hs_lead_type` and/or `lo_mql_origin` | Live builds can use `hs_lead_type` for Re-qualification / Re-engagement markers, but production population is sparse (3/867 Leads on 2026-05-19). Do not use Lead Type as a primary operator filter/UI column; use priority, stage, label, MQL attribution, and contact `lender_icp` instead. |
| SAL snapshot | `lo_sal_date`, `lo_mkt_score_at_sal` | Stamped the first time a non-blank responsiveness label is written. |
| SQL date | `lo_sql_date` | SQL is the manual Qualified milestone after credible fit/path to revenue is confirmed. |
| DQ reason | `lender_dq_reason`, `dq_date` | Required on Disqualified Leads. `NO_RESPONSE` can be auto-written by Unresponsive logic. |
| Responsiveness points | `lo_sales_responsiveness_points` | Per-cycle point total computed from logged activity and decay. |

## Qualification Definitions

| Stage | Plain English | Operational rule |
|-------|---------------|------------------|
| **MQL** | Qualified demand that deserves LSM review | Contact score >= 60 OR approved exception signal; eligible lifecycle phase; no active Lead; no active BBYS Deal; 90-day cooldown clear; Lead created in `866607668`. |
| **SAL** | Sales accepted the Lead and a real conversation is happening | First valid Hot/Warm/Cold signal writes `hs_lead_label`, stamps `lo_sal_date`, snapshots MKT Score. Outreach attempt alone should not be treated as SAL unless the responsiveness workflow's signal rules support it. |
| **SQL** | Confirmed fit with credible path to revenue | LO understands BBYS, sees value, and has a ready client or credible deal path. LSM manually moves Lead to Qualified / SQL milestone. |
| **Disqualified** | Lead failed this qualification cycle | Use one of the 8 canonical DQ reasons. DQ starts cooldown rules for future re-entry. |
| **Paused** | Timing issue, not a true DQ | Bad Timing = 30 days; Competitor Considering = 90 days. Auto-requeues after timer. Do not pollute DQ rates with timing pauses. |

## Scoring Model

The Contact-level MKT Score has eight groups:

| Group | Cap | Why it matters |
|-------|-----|----------------|
| Persona / ICP | +/-50 | Core fit. Challenger +50, Entrepreneur +40, Steady Eddy +30, Learner +20. Steady Eddy outranks Learner. |
| Elite Lender Tier | +/-50 | Prior BBYS success / tier status is a strong fit signal, especially for At Risk win-back. |
| Email | +30 / -85 | Opens, clicks, unsubscribe, bounce. Opens remain included for now despite Apple Mail Privacy inflation. |
| Website Activities | +/-30 | Conversion and evaluation URLs. Strong page activity can push an otherwise qualified LO over the gate. |
| Events | +/-30 | n8n-decayed event score with 90-day half-life. Executive Dinner attended carries the strongest raw event signal. |
| Demographic | +10 | Phone populated and Modex channel type. Pure Broker +8, Pure Banker +5, Hybrid +4. |
| Activity Decay | -30 | Demotes stale contacts after 60/90 days. |
| Milestones | +/-50 | Education, onboarding, first app, first close. Most useful for At Risk prioritization. |

The >= 60 gate was chosen because the April 2026 backtest showed a 4.0x rate ratio at 60: 52% Won recall vs 13% DQ leak. At 50, the model produced too much noise; at 70, it missed too many historical Wons.

## Priority Model

Priority is written to the Lead as `lo_lead_priority`. Sales should work the queue by priority, not by raw MKT Score.

| Tier | SLA language | Treatment |
|------|--------------|-----------|
| P0 | Drop everything | Leadership-flagged or highest-intent escalation. Rare target: ~1-3% of volume. |
| P1 | Same day | Top priority. Personalized outreach the day the Lead lands. |
| P2 | Next day | Work next business day. Still owned by LSM, not nurture. |
| P3 | End of week | Standard sales work. |
| P4 | Best effort | Lower priority. Work when bandwidth allows. |
| P5 | Nurture only | No proactive sales work unless new intent changes the picture. |

The Priority Matrix Writer runs once at Lead creation. Pre-SAL, `hs_lead_label` is blank, so the workflow approximates the matrix column using MKT Score band:

| MKT Score | Pre-SAL routing proxy |
|-----------|-----------------------|
| 90+ | Hot |
| 60-89 | Warm |
| < 60 | Not MQL-eligible |

Persona rules:

| Persona | Hot | Warm | Notes |
|---------|-----|------|-------|
| Challenger | P1 | P2 | Challengers never fall below P3 in the broader matrix. |
| Entrepreneur | P1 | P2 | Hot Entrepreneurs are P1 because speed matters most for this persona. |
| Steady Eddy | P2 | P3 | Moderate production / adaptable; not nurture-only. |
| Learner | P3 | P4 | Lower active-sales urgency. |
| Enterprise - Poor Fit | P4 | P4 | Structural mismatch overrides engagement. |
| Excluded | P5 | P5 | Nurture/self-serve unless separately approved. |

## Sales Responsiveness Model

Sales Responsiveness is Phase 2. It is per-Lead, not per-Contact. It should start only after sales activity creates a real signal.

| Signal | Points |
|--------|--------|
| Inbound reply (email / SMS) | +3 |
| Meeting booked | +5 |
| Meeting attended | +5 |
| Inbound call answered | +4 |
| Outbound connected >= 60s | +2 |
| Outbound attempt with no connect | -1 |
| Days since last inbound activity | -0.1 / day |

Label resolution:

| Label | Rule |
|-------|------|
| Hot | points >= +6 |
| Warm | points +1 to +5 |
| Cold | points -3 to 0 |
| Unresponsive | points <= -4 plus outreach / age gates; Auto-DQ to `NO_RESPONSE` |

Important operating rule: **do not manually set a Lead to Hot/Warm/Cold as a shortcut.** The LSM should log the call, meeting, reply, or outreach. The workflow should translate that activity into the label.

## Guardrails

Guardrails are more important than score. A high score is not enough to create a Lead if the LO is already in another active motion.

| Guardrail | Behavior |
|-----------|----------|
| Land/Expand customer | Excluded from MKT scoring. Route to expansion, not MQL. |
| Active BBYS Deal | Skip Lead creation; follow the Deal. |
| Active Lead in any lender pipeline | Skip Lead creation; avoid duplicate queue cards. |
| Archived Lead < 90 days ago | Skip; cooldown prevents ping-pong. |
| First BBYS app date known | Net-new MQL Trigger skips; Win-Back path handles At Risk post-app contacts. |
| Owner missing | Lead can still be created; owner falls back to Nick Santulli. High fallback rate is an owner-data issue. |
| Orchard Mortgage / orchard domain | Excluded from scoring and funnel enrollment. |

The active Lead check must scan the new unified pipeline plus all legacy Lead pipelines. Do not simplify it to `866607668` only during debugging; that reintroduces duplicate-active-Lead risk.

## Workflow Map

| Workflow | Object | Trigger | Writes | Status / notes |
|----------|--------|---------|--------|----------------|
| MQL Trigger | Contact | Score crosses >= 60 | Creates Lead, attribution, owner | Live since 2026-04-28. Re-enrollment off. |
| Priority Matrix Writer | Lead | Lead created in New | `lo_lead_priority`, `lo_priority_last_computed_at` | Live 2026-05-13. Runs once. |
| Re-MQL Sweep | Contact | Daily scheduled | Creates Re-qualification Lead | Covers contacts still >= 60 after 90-day cooldown. |
| Win-Back MQL Trigger | Contact | At Risk / Churned score crosses >= 60 | Creates Re-engagement Lead; writes Re-engaging classification | Handles prior-app / prior-close population skipped by net-new MQL Trigger. |
| Sales Responsiveness Label | Lead | Activity / property change | `lo_sales_responsiveness_points`, `hs_lead_label`, SAL snapshot, Auto-DQ | Live (confirmed 2026-08-10 portal sweep). |
| Unresponsive Sweep | Contact/Lead association | Daily scheduled | Cold -> Unresponsive / Auto-DQ | Live 2026-05-13. Backstop for Cold Leads that stop re-enrolling. |
| W-A Property Update | Contact | Lifecycle input changes | `lender_journey_stage`, `lender_lifecycle_phase` | Core bowtie classifier. |
| W-B Re-engaging | Contact/Deal | Re-engagement override | Re-engaging state | Deal-triggered override path. |
| W-C Dormancy Sweep | Contact | Daily scheduled | Dormancy and At Risk classifications | Backstop for time-based state changes. |
| n8n Event Scoring | Contact property/event | Daily 02:00 UTC | `lead_scoring_calculation_event_score`, `lo_event_score` | Event decay engine. |

## Operating Ownership

| Owner / team | Owns |
|--------------|------|
| Nick Santulli / RevOps | HubSpot architecture, workflow QA, property design, guardrails, dashboard definitions, operational monitoring. |
| Gui Batista | Lead scoring / ICP build, score and workflow implementation support, sales deck / rollout materials. |
| Marketing | Marketing signal strategy, campaign alignment, MQL quality feedback, score calibration partnership. Mette Adams was the prior marketing stakeholder and is no longer with HomeLight as of Friday 2026-06-12; Javy now covers Mette's external-events / Elite / partner-conference scope. |
| Sales leadership | SLA expectations, LSM adoption, funnel conversion targets, approval of sales operating rules. Wei Wang was the prior sales leadership stakeholder and is no longer with HomeLight as of Friday 2026-06-12; current owner TBD. |
| LSMs | Work the queue, log activity, move Leads through SAL/SQL/DQ/Paused correctly, own the LO relationship end-to-end. |

## Daily / Weekly Operating Checks

Daily RevOps checks:

- MQL Trigger execution count: investigate drop > 50% vs prior day.
- Any workflow errors in MQL Trigger, Priority Matrix Writer, Sales Responsiveness, Win-Back, Re-MQL, or Unresponsive Sweep.
- New Leads with blank `lo_lead_priority`: should be near zero after writer is live.
- Leads assigned to Nick fallback: should be small; high volume means upstream owner hygiene issue.
- New MQLs with `lo_qualification_funnel_mql_activity = Fit-Only`: > 50% suggests the score is moving mostly on property changes, not behavior.
- Sales Responsiveness distribution: Hot > 80% of labeled Leads is a red flag.
- Auto-DQ `NO_RESPONSE` spikes: check whether reps are under-working Leads or the Unresponsive logic is too aggressive.

Weekly checks:

- Persona distribution of MQLs: expected launch mix was ~85-95% Challenger / Entrepreneur.
- MQL -> SAL conversion by P-tier and LSM.
- P0/P1/P2 SLA attainment.
- DQ reason distribution.
- Active Leads with reach-out count zero.
- Legacy pipeline backlog clearing separately from new-model performance.

## Calibration Cadence

The MQL gate launched 2026-04-28. Calibration is anchored to launch and BBYS deal velocity, not arbitrary monthly dates.

| Target date | Checkpoint | Scope |
|-------------|------------|-------|
| ~2026-05-12 | 2-week health check | Plumbing only: workflow volume, attribution, owner inheritance, errors, duplicates. |
| ~2026-05-28 | 30-day operational read | Volume, persona mix, channel distribution, score distribution, early MQL->SAL trend. |
| ~2026-06-27 | 60-day engagement check | Email/web/event signals settling; consider whether numeric Sales Score is worth building. |
| ~2026-07-28 | 90-day primary tuning | First meaningful outcome signal from launch cohort; threshold and weight recalibration decision. |
| ~2026-10-28 | 180-day Equity Boost review | Product/messaging impact on EU Too Low and HELOC utilization. |

Do not tune the MQL threshold before the 90-day checkpoint unless the system is operationally broken. Earlier checkpoints can prove the plumbing works; they cannot prove win-rate quality.

## Known Gaps / Watch Items

- ~~Sales Responsiveness master workflow is built in test mode~~ — **Corrected 2026-08-10:** the 10 Aug portal sweep confirmed Sales Responsiveness is live with active enrollment, not test mode. Labels can be treated as production data.
- Dedicated event bypass workflow is target behavior, not fully implemented. Until then, event signals mostly influence MKT Score through n8n.
- Some docs still reference `lo_mql_origin`; live build can write `hs_lead_type` for Re-qualification / Re-engagement markers, but it is too sparse for primary reporting or Data Bridge operator filters. Verify field usage before reporting.
- Stage IDs other than Lead New should be pulled from HubSpot live rather than copied from old notes.
- Owner fallback ID is documented as Nick Santulli (`656430767`) in this project. If that conflicts with current team ownership, verify against live HubSpot before changing code.
- `lifecyclestage` is stale/noisy for BBYS Lender work. Use `lender_lifecycle_phase` and `lender_journey_stage`.
- Legacy pipeline `735605687` remains important for duplicate prevention and historical reporting even though new Leads should not be created there.
- Hourly SLA language in the original executive deck is stale. Daily P0-P5 framing is the current rule.

---

## Top-Level Numbers (from the deck)

| Number | What it means |
|--------|---------------|
| **~1,771** | Modeled net-new MQL candidates under the new model |
| **91%** | Of modeled MQL candidates are Challenger or Entrepreneur lenders (growth potential, not mature producers) |
| **60** | MKT Score threshold selected from backtest — the MQL gate |
| **~85%** | Long-term retention at "Magic 3" (3+ IR Closes). North-star retention proxy |
| **~66,487** | Total scored lender contacts in the database |
| **4,072** | Contacts that cross the gate at score ≥ 60 (~6.1%) |
| **76%** | Of the database has zero engagement events |
| **21,928** | Contacts in the negative-fit, low-engagement corner — excluded from routing entirely |
| **242 /mo** | Current IR Contracts pace (average) |
| **500 /mo** | Aspirational IR Contracts target (+107% lift) |

> Commentary: the 66,487 → 4,072 → 1,771 funnel is the math underlying the gate. The revamp does **not** promise to double IR Contracts on its own — the deck explicitly frames this as "one lever toward 500/mo" with realistic +10/15/20/25% post-ramp scenarios.

---

## Part One — Repairing the Leaky Funnel

### Slide 1 · Cover

- **Title:** From lead scoring to revenue control. A cleaner MQL/SAL/SQL operating model for prioritizing lender revenue, accelerating deal creation, and building sales intelligence.
- **Audience:** Executive
- **Status:** Proposed — pending approval
- **Decision needed:** Sales SLA tiers (P1–P5)
- **Timeframe:** BBYS · Q2 2026

### Slide 3 · Executive Answer

> "We changed the model because it can't reliably tell LSMs where the next deal is."

Four problem→fix pillars:

| # | Pillar | What changes |
|---|--------|--------------|
| 01 | A bigger addressable universe | Surfaces under-activated lender demand that the old model missed or buried |
| 02 | Cleaner prioritization | One unified queue with priority tiers — LSM knows what to work first, and why |
| 03 | Better rep actionability | Score, lifecycle, and engagement are defined so the next action is obvious |
| 04 | Better executive measurement | Funnel definitions tie qualification directly to IR Contracts and revenue |

**Speaker note (verbatim):** "We all know the issues with the current model… the new model is built around identifying the top opportunities for LSMs, handing off these opportunities at the most impactful moment with more insight than ever before while also making reporting substantially easier to understand."

### Slide 4 · The Tradeoff We Are Choosing

> "Fewer MQLs are better — if they convert. We are not optimizing for more MQLs. We are optimizing for more Contracts."

| OLD MODEL | NEW MODEL |
|-----------|-----------|
| More MQLs, lower trust | Cleaner MQLs, higher trust |
| Inflated by current customers and duplicates | Customers and active Deals filtered out |
| Mixed lifecycle signals on one score | MQL → SAL conversion becomes meaningful |
| Hard to defend in C-level reporting | SQL → IR Contract is the proof point |

**Speaker note:** "Level-set that there will be substantially less MQLs — but that's alright! We expect these to be much more vetted and impactful, allowing us to focus on the right people and reduce 'noise' that bogs down sales' time and capacity."

> Commentary: this is the framing the rest of the deck is built on — volume↓, trust↑, SAL becomes a real conversion gate.

### Slide 5 · Where the Revenue Comes From

The **Activation Ladder**:

```
Top of funnel  →  New gate            →  Deal motion         →  Retention
Aware → Engaged   MQL → SAL → SQL        IR Submit → Close     Magic 3 → Retained
```

- **Magic 3** = 3+ IR Closes. Correlates with **~85% long-term retention.**
- 91% of modeled MQL candidates are Challenger or Entrepreneur — growth potential, not mature producers.
- The goal is not just more MQLs. The goal is **more lenders moving toward repeatable close behavior.**

**Speaker note:** "The key to this model is not just getting any 'engaged' LO onboarded or re-onboarded; its focus is on identifying the right LOs — through engagement and fit — to target, and then getting them to elite, diamond, and obsidian."

> Commentary: this connects to [[lo-lifecycle]]'s funnel — Magic 3 is the same activation/retention pattern (1st close → 63% retention, 3+ → 85%) already in that file. Keep them in sync.

### Slide 6 · Clean Definitions

| Stage | Plain English | Rule |
|-------|--------------|------|
| **MQL** — Marketing Qualified | Qualified demand that deserves LSM review | MKT Score ≥ 60, high-value persona, eligible lifecycle phase, no known first BBYS app, passes guard checks before a Lead is created |
| **SAL** — Sales Accepted | LSM has accepted the Lead and a real conversation is happening | Real engagement signal exists — call answered ≥ 2 min, meeting attended, discovery held, or SPICED in progress. MKT Score is snapshotted; Phase 2 Sales Responsiveness starts |
| **SQL** — Sales Qualified | Confirmed fit with a credible path to revenue | LO understands BBYS, sees value, and has a ready client or a Deal created within the 120-day Lead window. Lead moves to Qualified; onboarding handoff begins |

**What this fixes:** MQL is no longer "any engaged person." SAL is no longer assumed at Lead creation. SQL is no longer just a high score — it requires confirmed fit and a credible path to revenue.

**Speaker note:** "It makes literally no sense [today]. In the new world, we now have a clean funnel pathway from MQL to SAL to SQL. It's modelled on the standardized lead funnel so no one gets confused — but we've tailored it entirely around lenders and BBYS."

> Commentary: these become the canonical definitions for `hs_lead_label`, `mql_date`, `sal_date`, `sql_date` (or equivalent) — any Data Bridge workflow or HubSpot Workflow that stamps qualification dates should follow these rules exactly.

### Slide 7 · From MQL to SAL

Five-step flow:

1. **MQL created** — Lead enters unified `[BBYS] Lender Qualification` pipeline.
2. **Priority queue** — assigned P-tier; `hs_lead_label` intentionally blank.
3. **Sales works by SLA** — P1 first, then P2, then P3 — by tier, owner context, recency.
4. **Real engagement** — call ≥ 2 min, meeting, discovery, or SPICED in progress.
5. **SAL recorded** — Sales Responsiveness starts: Hot / Warm / Cold / Unresponsive.

**SAL trigger signals (any of):**
- Call answered for ≥ 2 minutes
- Meeting attended
- Discovery call held
- SPICED conversation in progress

**What NOT to do:**
- Auto-convert every MQL to SAL on the gate.
- Treat first outreach attempt as SAL.
- Let Sales cherry-pick easy names without the queue governing SLA.

**Why:** If every MQL auto-moves to SAL, MQL→SAL conversion becomes meaningless.

> Commentary: the "intentionally blank `hs_lead_label`" is important — it means the MQL flag lives on the Lead state itself (P-tier + pipeline membership), not on a label that already implies acceptance. Any Data Bridge endpoint that creates Leads should leave `hs_lead_label` null until SAL criteria fire.

### Slide 8 · Exception Scenarios

Some motions bypass the score gate — but **lifecycle guardrails always apply.**

| Scenario (eligible non-customers) | Default action | Why |
|-----------------------------------|----------------|-----|
| Sales manually creates an outbound Lead | **MANUAL LEAD** | Sales can identify qualified demand before marketing score catches it. Source = Sales Outbound; reportable separately |
| Attended conference | **AUTO-MQL** | Event attendance is enough intent for Sales review |
| Visited booth | **AUTO-SAL** | Direct sales interaction or explicit engagement happened |
| Entered raffle | **AUTO-SAL** | Captures direct event engagement and consented follow-up |
| Attended dinner | **AUTO-SQL** | High-intent, high-touch engagement with enough context to treat as sales-qualified |

**Non-negotiable rule:** If the person is already a current customer or Land–Expand contact, the same activity is treated as **expansion** — not a new MQL/SAL/SQL.

**Speaker note:** "There are some scenarios where we will want to push past the regular steps of the funnel… a lender who enters a raffle (and engages at an in-person event) will automatically move to SAL. If they attend a dinner, SQL."

> Commentary: this exception matrix needs to live in code somewhere — likely a Data Bridge tool or n8n workflow that consumes event signals (booth scan / raffle entry / dinner attendance) and routes to the right starting stage. The customer-status check is the same guardrail used in [[lo-lifecycle]] and [[hubspot]] deactivated-LO logic.

### Slide 9 · Day One for Sales — One Queue, One Priority Rule, One SLA Clock

**Operating changes on launch:**
- Sales works from the unified `[BBYS] Lender Qualification` queue.
- The only number Sales needs to act on is **P-tier priority.**
- P1 → P2 → P3. **MQL ≠ SAL; engagement still required.**
- Current customers route to expansion, not net-new Leads.
- Manual outbound Leads need source, duplicate, and customer guardrails.

**What changes for the rep:**
- From "qualified vs not" → to "what to work, in what order, by when"
- From a binary score signal that didn't tell reps what to work first → to a ranked queue with a P-tier and an SLA on every Lead.

**SLA TIERS — DECISION NEEDED** *(proposed durations — these are the only operating rule needing formal Sales leadership approval)*

| Tier | Proposed SLA | Meaning |
|------|--------------|---------|
| **P1** | 1 hr | Drop everything; highest intent and fit |
| **P2** | 24 hr | Same-day priority |
| **P3** | 48 hr | Standard sales work |
| **P4** | Best effort | Work when bandwidth allows |
| **P5** | Nurture | No proactive Sales SLA |

**Speaker note:** "P1 will be like a top modex LO who's clicking links from marketing emails every day, attends our webinars consistently, etc. From there it cascades down to a no-volume LO who probably doesn't even sign into his email."

> Commentary: P1–P5 here are the **SLA tiers** — distinct from the P1–P4 Effort × Deal-Velocity quadrant on Slide 32 (see "Naming Note" there). Don't conflate them in HubSpot property naming. Suggested property: `lead_sla_tier` (P1–P5) separate from any deal-effort framework.

---

## Part Two — Sales Experience & Measurement

### Slide 11 · The Sales Intelligence Layer

A ranked work queue with context — not a vague score. **For every Lead, Sales sees:**

| What the rep sees | What it answers |
|-------------------|-----------------|
| Name, company, **P-tier + SLA** badge | Who to work — unified queue, priority logic |
| **Persona** (e.g. Challenger · Land) | What segment — lifecycle and persona context |
| **MKT Score** (e.g. 73 / 100) | Why now — score drivers and recent engagement |
| **Lifecycle phase** (e.g. Reactivation eligible) | Where in the lifecycle |
| **Last engagement** (e.g. Booth visit · 2 hr ago) | Why now |
| **Active Lead / Active BBYS Deal** flags | What NOT to work — active Lead, active Deal, cooldown guardrails |
| **Top reasons** (pricing page sessions, calculator submitted, booth visit) | Why now — score drivers |
| **Recommended next action** (e.g. "Call within 1 hr → discovery on Magic 3 path") | How to allocate effort — effort × deal-velocity lens |

**Example Lead card from the deck:**
> Maria Chen — Crosswind Lending · **P1 · 1 HR** · Challenger · Land · 73/100 · Reactivation eligible · Booth visit 2 hr ago · pricing page (3 sessions / 7d) · Calculator submitted (est. unlock $145K) · Booth visit at Western LO Summit · **Recommended: Call within 1 hr → discovery on Magic 3 path**

**Speaker note:** "Day one isn't going to look like what's on the right — that's part of our eventual sales copilot concept — but we will mirror it in HubSpot/Slack as much as possible."

> Commentary: this is exactly what the **Copilot / HubSpot Helper / Hub CRM mirror** stack already does for deals (see [[2026-04-21-hub-crm-mirror]] and [[2026-04-22-copilot]]). The Lead-level version of this view is the natural extension — `/hub/leads` mirror + agent-lead-snapshot recipe. Phase 1 will be mirrored in HubSpot lead views and Slack DMs; full intelligence card lives in the Copilot.

### Slide 12 · North Star and Supporting Metrics

> Measure fewer things. Make every metric actionable.

| Metric | Definition |
|--------|------------|
| **NORTH STAR — IR Contracts per month** | Closed apps as the single revenue signal |
| **LEVER 1 — P1 / P2 SLA attainment** | Are reps working the best opportunities fast enough? |
| **LEVER 2 — MQL → SAL within SLA** | Are prioritized MQLs becoming accepted conversations? |
| **LEVER 3 — SQL → IR Contract** | Are qualified LOs producing closed apps? |

**Realistic post-ramp scenarios:**

| Scenario lift | IR Contracts / month | Incremental |
|---------------|----------------------|-------------|
| Current pace | 242 | — |
| +10% | 266 | +24 |
| +15% | 278 | +36 |
| +20% | 290 | +48 |
| +25% | 303 | +61 |
| **Aspirational (500/mo)** | — | **+107%** lift required |

> "The revamp is one lever toward 500/mo — not a promise this project alone doubles production."

**Speaker note:** "Simplicity is the key here… an emphasis on acceptance: are reps accepting the leads and working them? This is two-fold: it helps us understand if they're doing the work while also providing a feedback loop to help continually tailor the funnel."

> Commentary: these four metrics should land in `/dashboard/...` endpoints in Data Bridge alongside the existing pipeline-breakdown / new-contacts dashboards. SLA attainment requires capturing the *first meaningful sales activity timestamp* against the *Lead-created timestamp* per P-tier.

### Slide 13 · Guardrails

| Bucket | Guardrails |
|--------|-----------|
| **Customer Protection** | Land/Expand customers excluded from scoring · Customer event actions route to expansion — not new MQLs · Contacts with a known first BBYS app are skipped (already past MQL) |
| **Duplicate Prevention** | No new Lead if an active Lead already exists · No new Lead if an active BBYS Deal exists · Manual & exception Leads still run customer / Deal / Lead checks |
| **Stage Discipline** | Normal MQLs do not auto-convert to SAL · Priority/SLA governs work order; engagement governs SAL · MQL enrollment doesn't depend on owner — fallback owner logic exists |
| **Reactivation Control** | Re-MQL Sweep planned as a daily workflow — *not live yet* · Re-MQL needs score ≥ 60, no active Lead/Deal, eligible phase, last archived Lead ≥ 90 days · Dormancy: 30 days onboarded-no-app · 180 days post-close |

**Speaker note:** "Standardization on stages, reduction of noise, no duplicates ever, and more automated efficiency for sales."

> Commentary: most of these guardrails map cleanly to existing Data Bridge logic — the customer-status check is similar to the `active_lender` / `hl_contact_type = 'Deactivated'` / PLACEHOLDER-company patterns in [[hubspot]]. The duplicate-Lead check needs a **cross-pipeline scan** (unified + all legacy Lead pipelines — see Slide 25 / Appendix A3).

### Slide 14 · Disqualification Reasons

Eight reasons to close a Lead — plus a sub-reason for "why."

| # | Reason | Mode | Re-entry | Sub-reasons |
|---|--------|------|----------|-------------|
| 01 | **No Response** | Auto | 90d | All channels exhausted — zero engagement; Voicemail wall — calls reach VM, never connect |
| 02 | **Not Interested** | Manual | 180d, inbound only | Happy with current solution; Not the decision-maker on tools |
| 03 | **Not a Good Fit** | Manual | Manual review | FHA-heavy book (≥ 40%) — BBYS economics break; Production too low to support program |
| 04 | **Already a User** | Auto / Manual | Merge with record | Active BBYS user — duplicate MQL; Duplicate Contact / Lead — merge required |
| 05 | **No Longer an LO** | Manual | None | Left the industry / changed careers; License inactive or revoked |
| 06 | **Lead Expired** | Auto | On intent | Pre-SAL expiry — never accepted by sales; Stale signal — no fresh trigger in 120d |
| 07 | **Competitor (Exclusive)** | Manual | 180d · hard DQ | Exclusive contract with competitor; Captive at a lender that prohibits BBYS |
| 08 | **Bad Data** | Auto / Manual | On data correction | Invalid email or disconnected phone; Test, spam, or wrong person on record |

**Pauses ≠ DQs** — two pause states sit in a **Paused** stage with a timer that auto-requeues the Lead back to MQL:
- **Bad Timing** · 30d
- **Competitor Considering** · 90d

**Speaker note:** "We standardized disqualification reasons to 8 — these give us sufficient insights without causing too much confusion. These also don't just help refine the model but help to determine how that LO is approached in the future (or if they're approached)."

> Commentary: this is a HubSpot property design task — likely `lead_dq_reason` (picklist, 8 values) + `lead_dq_subreason` (dependent picklist) + `lead_paused_until` (date, used by the auto-requeue cron). The "On intent" re-entry for #06 is interesting — should trigger a re-MQL evaluation when a fresh score event fires, not on a fixed timer.

### Slide 15 · Sales Responsiveness Score

Same four labels (**Hot / Warm / Cold / Unresponsive**) — but mechanical thresholds instead of rep judgment.

**Panel A — Activity weights** *(tunable in config, not in rep training):*

| Activity | Weight | Notes |
|----------|--------|-------|
| Inbound reply (email / SMS) | +3 | Substantive, not auto-responder |
| Meeting booked | +5 | Discovery or demo on calendar |
| Meeting attended | +5 | Stacks with booked → +10 total |
| Inbound call answered | +4 | Lead initiated |
| Outbound connected (≥60s) | +2 | Two-way exchange |
| Outbound attempt — no connect | −1 | Caps "tried hard, no response" |
| Days since last inbound activity | −0.1/d | Decay so old engagement fades |

**Panel B — Label thresholds** *(mechanical bucket assignment from the Panel A sum):*

| Sum | Label | Meaning |
|-----|-------|---------|
| ≥ +6 | **HOT** | Active two-way conversation, recent meeting or replies |
| +1 to +5 | **WARM** | Some response, slow but not silent |
| −3 to 0 | **COLD** | Multiple attempts, no substantive response |
| ≤ −4 **AND** reach-outs ≥ 5 **AND** age ≥ 21d | **UNRESPONSIVE** | Auto-DQ trigger — only label that closes the Lead |

**Panel C — Why this is better:**
1. **Deterministic** — two reps looking at the same Lead get the same label.
2. **Auditable** — replay the activity log and verify the bucket end-to-end.
3. **Foundation for numeric Sales Score** — same weights surface as a number when we un-defer it. Weights live in config — not in rep training decks.

**Worked example:**
> 1 reply (+3) + 1 meeting booked (+5) + 2 outbound attempts (−2) = **+6 → Hot**

**No HubSpot schema change required.** The four label values don't change — only how reps and automation arrive at them.

**Speaker note:** "We've set preliminary scoring on various engagements using standard scores that studies and our own data reflect high conversions. These aren't set in stone — we can adjust them now and we will adjust them as we move forward."

> Commentary: this is a natural Data Bridge service — `sales-responsiveness/recompute` endpoint that reads engagements from `hub_engagements` + `hub_contacts` and writes the label back to HubSpot. Weights belong in a config table (e.g. `sales_responsiveness_weights` in Supabase) so they can be retuned without code deploy. The auto-DQ trigger (unresponsive criteria) should fire on a daily cron — similar pattern to the `self-improvement` cron in [[data-bridge]].

---

## Part Three — Why This Threshold, Why Now

### Slide 17 · Marketing Score Snapshot — Where the 66,487 Sit

**Fit × Engagement distribution** (snapshot grid):

```
                         E1 (≤0)   E2 (1-4)   E3 (5-14)   E4 (15-29)   E5 (30+)
F5 (60+ · Strong)            14      1,085         543         238          58    ← 57%-100% MQL
F4 (40-59)                   81     14,843       2,180         611          76    ← 0%-100% MQL
F3 (20-39)                  126      8,589       1,277         261          33    ← 0%-61% MQL
F2 (0-19)                   137     11,091         969         265          38    ← 0% MQL
F1 (<0 · Negative)          406     21,928       1,422         190          26    ← 0% MQL
```

Key reads:
- **F5 (Fit ≥ 60) = 1,938 contacts total** — the entire ceiling of qualified pipeline. Every cell in that row crosses the MQL gate at near-100%.
- **F1 · E2 = 21,928 contacts (33% of database)** — negative fit, low engagement. **Excluded from routing entirely.**
- **Footer:** 66,487 total scored / 4,072 cross the gate (6.1%) / 76% have zero engagement.

**Cell color legend:**
- Green — ≥ 80% MQL-rich
- Amber — 20–79% mixed
- Gray — < 20% almost no MQLs

**Speaker note (verbatim):** "This is the lay of the land. We scored 66,487 lender contacts on two axes: Fit — how well the contact matches our buyer profile — and Engagement — how much they have actually interacted with our marketing. Each cell shows the count, and the color shows what share of those contacts cross the MQL gate of 60. The green band in the upper right is where the qualified leads live; everything gray is essentially dead weight."

### Slide 18 · Why the Gate Is at 60

The data has a **natural cliff** at 60 — population drops 4× and quality jumps 4×.

**Population by 10-point score bin:**

| Bin | Count |
|-----|-------|
| −80 → −10 | 16,713 |
| −10 → 0 | 8,805 |
| 0 → 10 | 5,820 |
| 10 → 20 | 5,668 |
| 20 → 30 | 4,019 |
| 30 → 40 | 6,846 |
| 40 → 50 | 3,699 |
| **50 → 60** | **10,845** |
| **60 → 70** | **2,479** |
| 70 → 80 | 869 |
| 80+ | 724 |

> The cliff: 10,845 contacts sit in 50–60. Just above it, only 2,479 sit in 60–70. Population drops 4.4× across one 10-point step.

**Gate comparison:**

| Threshold | MQL pool | % of DB | Won deals captured | MQLs that ended DQ'd | Win rate above vs below | Verdict |
|-----------|---------:|--------:|-------------------:|---------------------:|------------------------:|---------|
| Gate at 50 | 14,917 | 22% | 71% | 32% (1 in 3) | 2.2× | Too noisy — sales drowns |
| **Gate at 60** ✓ | **4,072** | **6%** | **52%** | **13% (1 in 8)** | **4.0×** | **Sweet spot — quality jumps 4×, volume stays workable** |
| Gate at 70 | 1,593 | 2% | 38% | 7% (1 in 14) | 5.4× | Too restrictive — we'd miss half our wins |

**Speaker note:** "60 is the sweet spot: the pool is workable at about 4,000, the disqualification rate falls to one in eight, and the win rate above the gate runs four times the win rate below."

> Commentary: this is the empirical justification — keep this table handy if Sales pushes back on the score threshold. The fact that the 4× quality jump *doubles* between 50→60 then barely moves from 60→70 is the strongest argument for the 60 line.

### Slide 19 · Calibration Cadence

> Set, measure, tune — against real funnel behavior.

| When | Checkpoint | What we look at |
|------|-----------|----------------|
| **2 weeks** | Health check | Workflows, attribution, owner fallback, guardrail skips, no execution errors |
| **30 days** | Operational read | MQL volume, persona mix, channel attribution, score distribution, MQL→SAL trend, DQ reasons, reach-out activity |
| **60 days** | Engagement check | Email, web, event, and Sales Responsiveness signals settling into usable patterns |
| **90 days** | Primary tuning | First meaningful outcome signal — decide whether threshold, weights, or deferred items move |
| **Quarterly** | Standing calibration | Drift, threshold sensitivity, persona performance, excluded logic, deferred enhancements |

**Important distinction:** 30 days is for **operational calibration** — signal sanity and sales adoption. Full threshold/weight recalibration waits for meaningful downstream outcomes around **90 days.**

> Commentary: these checkpoints are reportable deliverables — natural fit for the Friday weekly review cycle. A scheduled task at T+14 / T+30 / T+60 / T+90 generating each checkpoint's data is straightforward to build using existing Hub CRM mirror queries.

---

## Migration — Appendix A1–A5

### Slide 23 (A1) · Legacy Pipeline Status

| Pipeline | Future role | New Lead creation? |
|----------|-------------|--------------------|
| `[BBYS] Lender Qualification` | Active pipeline for all new / requalified demand | **Yes — only here** |
| `SQL-Lender Sales` (legacy) | Read-only for new creation; active records finish in place | No — frozen for creation |
| Archived legacy pipelines | Historical reporting and cooldown reference | No — historical only |

**Operating principle:** Legacy records remain valid historical records. They should not be deleted or rewritten to make launch reporting look cleaner.

### Slide 24 (A2) · Active Legacy Leads

Worked **in place** during the clearing window. **No bulk migration.**

**DO:**
- Let active legacy Leads close naturally.
- Use them for ongoing reporting and audit trail.
- Allow cooldown-based re-entry into the new model.

**DON'T:**
- Bulk-migrate active legacy Leads.
- Recreate active legacy Leads in the new pipeline.
- Use legacy outcomes to judge new-model performance.

### Slide 25 (A3) · Duplicate Prevention

Trigger → 3 checks → create only if clear:

1. **Trigger:** Score / exception signal — MKT score ≥ 60 *or* eligible event exception fires.
2. **Check 1 — Customer status:** Land / Expand customer? → expansion route.
3. **Check 2 — Active BBYS Deal:** Deal exists? → deal follow-up, no new Lead.
4. **Check 3 — Active Lead (current + legacy):** Active Lead anywhere? → skip creation.
5. **Result:** New Lead enters `[BBYS] Lender Qualification`.

**Cross-pipeline check:** The active-Lead check spans the unified pipeline AND all legacy Lead pipelines — so an old active Lead cannot generate a duplicate new one.

> Commentary: the cross-pipeline scan is the critical implementation detail — a Data Bridge tool that queries Lead-bearing pipelines (unified + every legacy ID) and returns "active anywhere?" before allowing creation. Belongs alongside the existing `lo-enrichment/webhook` guardrail logic.

### Slide 26 (A4 + A5) · Re-MQL Handling & Reporting Separation

**A4 — Re-MQL · 90-day cooldown**

```
T-0                       T+90D                  RESULT
Archived legacy Lead  →   Cooldown clears   →    Unified Lead
```
- Most recent archive date is the cooldown anchor.
- Score / eligibility recheck can fire at T+90.
- Re-MQL requires score ≥ 60, no active Lead/Deal, eligible lifecycle phase.
- Cooldown anchored on the most recent archived Lead.
- **Requalified demand never re-enters legacy pipelines.**

**A5 — Reporting separation**

| Lane | Reports on |
|------|-----------|
| **New-model lane** | SLA behavior, MQL→SAL→SQL conversion, IR Contracts |
| **Legacy clearing lane** | Backlog resolution, historical outcomes, audit trail |

**Why this matters:**
- New-model metrics start from Leads created in the unified pipeline post-launch.
- Legacy active Leads still produce outcomes — but don't judge new-model quality.
- Prevents launch metrics from being polluted by stage drift or backlog cleanup.

---

## Appendix — Additional Detail Slides

### Slide 27 · What Was Broken Before

Three layers of pain → concrete failure modes:

| Pain layer | Concrete failure modes |
|------------|------------------------|
| **Data problem** — Conflicting signals on one score | Customer inflation (Land–Expand contacts could count as MQLs, padding top-of-funnel). Stage drift (existing apps and active BBYS Deals not cleanly separated from net-new qualification) |
| **Sales problem** — Unclear priority, fragmented pipelines | Duplicate active Leads (over-contacting / misrouting). Fragmented pipelines (multiple qualification motions across separate lead pipelines). Reps could see who was "qualified" but not *why now*, *what to do next*, or *who to work first* |
| **Executive problem** — Inflated MQL counts leadership couldn't fully trust | One score, four jobs (marketing intent, fit, lifecycle, rep follow-up all blended). Win-back blind spot (re-engagement of dormant lenders was not controlled or measurable). Current customers / Land–Expand could trigger qualification logic |

### Slide 28 · Before / After Example

Same signal, **different motion — lifecycle decides.**

| Signal | Old model risk | New model treatment |
|--------|---------------|---------------------|
| Current customer attends dinner | Could be counted as new MQL/SAL/SQL, inflating funnel volume | **Expansion follow-up.** Owner activity, account note, campaign attribution. No new qualification Lead |
| Non-customer attends dinner | Buried as generic event engagement or lost to manual interpretation | **Auto-SQL exception** if guardrails pass. Direct, high-intent sales motion |
| Non-customer crosses score ≥ 60 from engagement | High score but unclear next action; risk of duplicate pipeline state | **MQL in priority queue.** Assigned a P-tier; SAL only after real engagement — never automatic |

> Lifecycle context — not the activity itself — decides whether a signal is acquisition, reactivation, or expansion.

### Slide 30 · The New Operating Model — Five Layers

| Layer | What it is | Description |
|-------|-----------|-------------|
| **Layer 1** | Contact MKT Score | Lender-level marketing & business intent |
| **Layer 2** | MQL Trigger | Score, eligibility, and guardrail checks |
| **Layer 3** | Unified Lead pipeline | `[BBYS] Lender Qualification` — one queue, one SLA clock |
| **Layer 4** | Sales Responsiveness | Hot / Warm / Cold / Unresponsive — separate from MKT score |
| **Layer 5** | Deal · archive · re-MQL | Bowtie lifecycle: activation, retention, dormancy, reactivation |

**What this gives each stakeholder:**
- **Marketing:** a defensible MQL gate that reflects fit + intent, not raw activity.
- **Sales:** one queue, one priority rule, one engagement signal — and exception paths for outbound.
- **RevOps:** bowtie visibility from acquisition through retention, dormancy, and reactivation.
- **Leadership:** funnel metrics that map to IR Contracts and revenue, not just activity.

### Slide 31 · The Old-Model Transition (Visual Summary)

| Lane | State |
|------|-------|
| Historical archived Leads | **READ-ONLY** — preserved for reporting & history; no new automated creation in legacy |
| Active legacy Leads | **FINISH IN PLACE** — worked in place until close, qualify, or DQ; naturally clear; no forced migration |
| New / requalified demand | `[BBYS] LENDER QUALIFICATION` — created only in unified pipeline; one active pipeline, one SLA clock |
| Customers / active Deals | **EXPANSION PLAYBOOKS** — route to expansion or deal follow-up; owner follow-up; campaign attribution |

**Transition rule:** The new MQL Trigger checks current and legacy Lead pipelines before creating anything. No duplicate active Leads across old and new model.

### Slide 32 · Effort × Deal Velocity (Sales-Intelligence Lens)

A 2×2 lens for *what kind of effort the opportunity deserves* — NOT the SLA matrix.

|  | Low effort | High effort |
|--|------------|-------------|
| **Fast velocity** | **P2 · Slam Dunk** — fast turnaround, minimal friction, close the loop | **P1 · House on Fire** — align expectations, solve property issues, protect urgent conversion |
| **Slow velocity** | **P3 · Nurture** — automated follow-up until a trigger event changes the picture | **P4 · Do Not Pursue** — deprioritize unless new signal changes the economics |

**Naming note (important to preserve):** *P1–P4 here are a deal-effort framework. The Lead queue uses P1–P5 SLA tiers. This picture is a sales-intelligence lens — it does not replace the formal SLA matrix.*

**Mapping back to the model:**
- MQL/SAL/SQL says **where** the LO is in the qualification funnel.
- Priority / SLA says **how fast** Sales should act.
- This 2×2 says **what kind of effort** the opportunity deserves.

> Commentary: keep this lens conceptually separate in any HubSpot property design. Suggest `lead_sla_tier` (P1–P5, the queue) vs. `deal_effort_quadrant` (P1–P4, the lens) — different properties, different scales, different uses.

---

## How This Maps to Existing Data Bridge Systems

Cross-reference for implementation planning. None of these are built yet — but each new-model concept has an obvious home in the existing stack.

| New-model concept | Likely Data Bridge surface | Notes |
|-------------------|---------------------------|-------|
| MQL Trigger + 3-check guardrails | New endpoint, e.g. `/lead-qualification/mql-trigger` | Same shape as `/lo-enrichment/webhook` — webhook in from HubSpot, runs guardrail chain, decides create / skip / route-to-expansion |
| Cross-pipeline active-Lead check | Shared util in `src/utils/` | Queries unified + legacy pipeline IDs; reused by manual-Lead creation, exception scenarios, re-MQL sweep |
| P1–P5 SLA tier assignment | Tool inside MQL Trigger | Score band + persona + lifecycle → P-tier; written to `lead_sla_tier` |
| Sales Responsiveness recompute | New cron, e.g. `cron-sales-responsiveness` | Daily; sums weighted engagements (from `hub_engagements`); writes label; fires auto-DQ when unresponsive criteria met |
| Re-MQL sweep | New cron, e.g. `cron-re-mql-sweep` | Daily; queries archived Leads where most-recent-archive ≥ 90d AND score ≥ 60 AND no active Lead/Deal AND eligible phase |
| Exception scenarios (booth / raffle / dinner) | n8n workflow or `/events/webhook` | Consumes event-source signals; applies customer guardrail; routes to auto-MQL/SAL/SQL |
| Lead intelligence card | `/hub/leads/:id` (mirror) + Copilot recipe | Lead-level analog of the existing `/hub/deals/:id` mirror; `agent-lead-snapshot` recipe in HubSpot Helper |
| North-star + 3 lever metrics | New `/dashboard/lead-funnel` endpoint | Adds SLA attainment, MQL→SAL conversion, SQL→IR Contract conversion to the existing dashboard suite |
| Calibration cadence reports | Scheduled tasks at T+14 / T+30 / T+60 / T+90 | Auto-generated weekly-review-style summary for each checkpoint |
| DQ reasons + paused states | HubSpot property design + Workflow templates | `lead_dq_reason` (8-value picklist) + `lead_dq_subreason` (dependent) + `lead_paused_until` (date); auto-requeue cron drives the pause |
| Effort × Velocity quadrant | Optional `deal_effort_quadrant` property | Sales-intelligence lens only — NOT the SLA tier. Keep naming distinct |

---

## Open Items / Decisions Pending

- [ ] **SLA tier durations (P1–P5)** formally approved by Sales leadership — the single open decision flagged in the deck.
- [ ] **Custom HubSpot property design** finalized (`lead_sla_tier`, `lead_dq_reason`, `lead_dq_subreason`, `lead_paused_until`, lifecycle markers, MQL/SAL/SQL date stamps).
- [ ] **Legacy pipeline IDs** captured for the cross-pipeline active-Lead check (which legacy pipelines, by ID, must be scanned).
- [ ] **Re-MQL Sweep cron** built and scheduled — explicitly called out as *not live yet* on Slide 13.
- [ ] **Sales Responsiveness weights config table** (Supabase) created so weights can be retuned without deploy.
- [ ] **Lead intelligence card** scope decided — HubSpot Lead view + Slack DM (Phase 1) vs. full Copilot Lead snapshot (Phase 2).
- [ ] **Event-signal sources** (booth scans, raffle entries, dinner attendance) — who captures, what format, which endpoint receives.
- [ ] **Score recompute frequency / latency budget** — how fresh must MKT Score be when MQL Trigger fires?
- [ ] **30/60/90 calibration checkpoint owners** assigned.

---

## Related Files

- [[2026-03-26-lo-qualification-system-justification]] — original business-case memo (~$292K/yr direct, $4.18M w/ attribution, 2,054 LSM hours saved over 6 months)
- [[lo-lifecycle]] — keep the Magic 3 / activation / retention numbers consistent between the two files
- [[hubspot]] — pipeline IDs, stage IDs, property catalog
- [[data-bridge]] — the Mastra integration platform that will own the new endpoints, crons, and dashboards
- [[bbys-overview]] — product context
- [[partners]] — Land/Expand customer definitions for the customer-protection guardrail
