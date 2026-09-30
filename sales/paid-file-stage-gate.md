---
date: 2026-04-20
type: decision
status: active
tags: [comp, compensation, bbys, hubspot, stage]
source: HomeLight-Vault/decisions/2026-04-20-paid-file-stage-gate.md
imported: 2026-09-29
---

# Decision: Main-plan comp gated on IR Closed or DR Closed stage

## What We Decided
A deal only pays out on the main comp plans (LSM, LRM, Nick, and everything derived — pod shares, Brandi/Ian LRM pool, Brandi's 1.5bps override, Jake's 16% of pool) when **both** of the following are true:

1. `ir_closed_date` falls in the comp period (bucketing — unchanged), **and**
2. Current `dealstage` is either:
   - **IR Closed** — `998755447`
   - **DR Closed** — `998815822`

The accelerator quota math is **intentionally unaffected** — it continues to see the full `irClosedDeals` population.

## Why
`ir_closed_date` can get set on a deal that didn't actually close (e.g., via a workflow misfire, manual edit, or a stage rollback). Without a stage guard, those files would still generate comp rows. The stage gate ensures the date landing in a month isn't enough on its own — the deal must currently be in a stage that represents real closure.

DR-only files (closing on the DR side without an IR event) are also valid paid files, which is why both stage IDs qualify. Bucketing still uses `ir_closed_date` — a DR Closed file's `ir_closed_date` is populated by the HubSpot sync, so it lands in the correct period.

**Alternatives considered:**
- Filtering at the DB query in `getDealsByPeriodField` → rejected: would hide the deals from the accelerator too, which we don't want.
- Only gating on IR Closed → rejected: would leave DR-only files unpaid.
- Requiring stage = IR Closed OR DR Closed without the date guard → rejected: wouldn't protect against the "stage set but `ir_closed_date` missing" case.

## Impact
- **LSM / LRM / Nick** main comp rows: now only generated for stage-confirmed paid files.
- **Downstream roles** (pod shares, Brandi/Ian pool, Brandi override, Jake's 16%): auto-follow because they derive from the filtered LSM/LRM comp arrays. No separate filtering needed.
- **Accelerator**: unchanged — quota attainment still counts the full period population.
- **Simulator "Deal Count" summary** now reflects the paid subset (not the full bucket).
- **Gui (RevOps-GB)**: unrelated — he's scored by `createdate` volume, not paid files.

Implemented as a single helper `isPaidStage(deal)` in `server/compEngine.ts`, wired into 6 loops (3 actual + 3 simulation).

## Revisit Date
Revisit if HubSpot introduces a new stage ID that should count as "paid" (e.g., a new closure variant) — it's a one-line change to `PAID_STAGE_IDS`.

## Related Notes
- [[2026-03-28-comp-app]]
- [[2026-03-30-configurable-comp-system]]
