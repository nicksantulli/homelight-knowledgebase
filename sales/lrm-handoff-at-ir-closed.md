---
date: 2026-05-14
type: decision
status: active
tags: [bbys, priority-queue, lrm, ca-handoff, scope]
source: HomeLight-Vault/decisions/2026-05-14-lrm-handoff-at-ir-closed.md
imported: 2026-09-29
---

# Decision: LRM scope ends AT IR Closed (not after)

## What We Decided
The Priority Queue's LRM ownership window shrinks from **7 stages → 6**. IR Closed (`998755447`) is no longer in `LRM_ACTIVE_STAGE_IDS`. LRM scope is now: New, In Review, Approved, Agreement Signed, IR In Escrow, Clear to Fund.

LSM scope unchanged at 4 pre-UC stages.

Both roles are now silent on IR Closed, DR Closed, Failed, and Nurture.

## Why
Operator (Tulli) opened the queue and saw a P0 row labeled "Clear to Fund" that, when clicked, was actually IR Closed in HubSpot. Two layers stacked into one bug:

- **Layer 1 (data staleness):** the score row's `stage_id` said CTF but `hub_deals.deal_stage` said IR Closed (deal moved at 6:23 AM, recompute hadn't refreshed at 10:02 AM).
- **Layer 2 (scope rule):** the existing rule INCLUDED IR Closed in LRM scope. The earlier comment said *"LRM owns through IR Closed boundary — handoff to CA at the boundary."* Operator clarified the handoff happens **AT the transition, not after** — so a deal in IR Closed has already been handed off and shouldn't be in the LRM queue.

Operator rule, verbatim: *"if it's closed then it's not of priority to LRMs or LSMs."*

Layer 2 fix makes Layer 1 moot — the orphan-delete sweep removes IR Closed rows entirely instead of trying to refresh stale `stage_id`s. Cleaner outcome than chasing stale stage refreshes.

**Alternatives considered:**
- *Keep LRM at 7 stages, fix the staleness with a tighter recompute SLA.* Rejected — the deal IS handed off, the score row is misleading regardless of staleness.
- *Add a sub-state ("IR Closed → in handoff" vs "IR Closed → handed off").* Rejected — overengineered. The transition IS the handoff per operator spec.

## Impact

**Code (PR #188, merged `8a2d481`):**
- `src/utils/priorityScore.ts` — `LRM_ACTIVE_STAGE_IDS` shrinks 7 → 6 (drop `998755447`).
- `supabase/migrations/20260514_priority_score_orphan_delete_lrm_drop_ir_closed.sql` — updates the role-aware orphan-delete RPC `delete_orphan_priority_score_rows()` to match.
- `admin/src/pages/QueuePage.tsx` — Stage filter dropdown shows 6 options for LRM/ALL (was 7), 4 for LSM (unchanged).

**Prod cleanup (immediate, not waiting on Railway):**
- Migration applied via Supabase MCP.
- Orphan-delete RPC run once: **435 stale score rows deleted** (430 LRM rows for IR-Closed deals + 5 LSM rows that were already stale from earlier transitions the orphan-delete had missed).
- Verified the originally-reported deal (Doctor Wylie / 56 South Wilson Avenue) is gone from `deal_priority_scores`.

**Downstream:**
- LRM Monday DM digests no longer include IR Closed deals.
- Stalled-deals widget already filters by PRE_UC stages only (PR #162 from previous session) — no change needed there.
- `signal_conversion_probability` was already gated on PRE_UC stages — no scoring change needed.
- Operator SOP (`sops/2026-05-13-priority-queue-operator-guide.md`) updated to reflect the 6-stage LRM scope.

## Revisit Date
**2026-08-14** (3 months) — re-check whether the LRM still wants zero IR Closed visibility, or whether a separate "recently handed off, watch for handoff failure" view is worth building.

## Related Notes
- [[2026-05-12-deal-priority-queue]] — main project doc
- [[2026-05-13-hub-deals-as-active-set-source]] — sibling decision: "is this deal currently active?" reads `hub_deals` not `partner_report_deals`
- [[hub-mirror-gotchas]] — Gotcha 5: prefer canonical state column over derived/cached fields (this fix is an applied case)
- [[2026-05-13-priority-queue-operator-guide]] — operator SOP, updated for 6-stage LRM scope
