---
date: 2026-05-14
type: decision
status: active
tags: [bbys, priority-queue, lrm, scope, funding-team]
source: HomeLight-Vault/decisions/2026-05-14-pm-lrm-drop-ctf-funding-team-owns.md
imported: 2026-09-29
---

# Decision: LRM scope ends AT IR In Escrow (drops Clear to Fund)

## What We Decided
LRM ownership window shrinks again — **6 stages → 5**. Clear to Fund (`998755446`) drops out of `LRM_ACTIVE_STAGE_IDS`.

LRM scope is now: **New, In Review, Approved, Agreement Signed, IR In Escrow** (5 stages).

LSM scope unchanged at 4 pre-UC stages.

This is the **second** LRM scope shrink today, after the morning's [[2026-05-14-lrm-handoff-at-ir-closed]] (which dropped IR Closed, taking LRM from 7 → 6).

## Why
Operator (Tulli) opened a deal popup for Stacy Brosius in **Clear to Fund** stage and flagged: *"Clear to fund needs to be excluded for the LRMs and LSMs as well."*

The principle: **once a loan is approved-to-fund, the funding/closing team owns the deal — not LRM**. CTF doesn't mean "still working it"; it means "loan is good, waiting on funding mechanics" — which is a different team's domain.

This is the same operator pattern that drove the morning's IR-Closed drop: *"if [some-other-team] now owns the deal, it shouldn't be in [LSM/LRM]'s priority queue."* Worth flagging because there may be more shrink coming as ownership boundaries get clarified through actual usage.

**Alternatives considered:**
- *Keep LRM at 6 stages, mark CTF deals visually as "funding-team-owned but still LRM-tracked".* Rejected — visual demotion in a priority queue still costs cognitive load for the LRM. Operator's rule is binary: either the role owns the work or it doesn't.
- *Add a sub-state for "in funding".* Rejected — CTF already IS the sub-state. Stacking another layer would be over-engineering.

## Impact

**Code (PR #193, merged `11c367d`):**
- `src/utils/priorityScore.ts` — `LRM_ACTIVE_STAGE_IDS` shrinks 6 → 5 (drop `STAGE_CTF`).
- `supabase/migrations/20260514_priority_score_orphan_delete_lrm_drop_ctf.sql` — updates the role-aware orphan-delete RPC `delete_orphan_priority_score_rows()` to match.
- `admin/src/pages/QueuePage.tsx` — Stage filter dropdown shows 5 options for LRM/ALL (was 6), 4 for LSM (unchanged).

**Prod cleanup (immediate, not waiting on Railway):**
- Migration applied via Supabase MCP.
- Orphan-delete RPC run once: **29 stale CTF rows removed**.
- Verified the originally-reported deal (Stacy Brosius / 6508 W AVENIDA DEL SOL) is gone from `deal_priority_scores`.

**Cumulative LRM scope shrink today:**

| When | Scope | Change |
|---|---|---|
| 2026-05-13 | 7 stages | LSM 4 + IRUC + CTF + IR Closed |
| 2026-05-14 AM (PR #188) | 6 stages | dropped IR Closed (handoff happens AT the boundary) |
| 2026-05-14 PM (PR #193) | **5 stages** | dropped CTF (funding team owns from CTF onward) |

**Downstream:**
- Operator SOP (`sops/2026-05-13-priority-queue-operator-guide.md`) updated to reflect 5-stage LRM scope.
- `signal_conversion_probability` was already gated on PRE_UC stages — no scoring change needed.
- Conv-badge hide-list unchanged (CTF was already not in it; the badge is a separate concern from queue scope).

## Revisit Date
**2026-08-14** (3 months) — paired with the AM IR-Closed decision review. Re-check whether the team's mental model has stabilized at "LRM scope = pre-UC + IRUC only" or whether further shrink is needed (e.g., would IRUC also leave LRM scope at some point?).

## Related Notes
- [[2026-05-14-lrm-handoff-at-ir-closed]] — sibling AM decision (LRM 7 → 6, dropped IR Closed)
- [[2026-05-12-deal-priority-queue]] — main project doc, Day 2 sprint section now reflects both shrinks
- [[2026-05-13-priority-queue-operator-guide]] — operator SOP, updated for 5-stage LRM scope
- [[2026-05-13-hub-deals-as-active-set-source]] — sibling decision: "is this deal currently active?" reads `hub_deals` not `partner_report_deals` — orphan-delete RPC inherits this same source-of-truth choice
