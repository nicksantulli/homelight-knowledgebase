---
created: 2026-06-22
type: sop
status: draft — pending Jake/Friedman sign-off
owner: Nick Santulli (RevOps)
approvers: Jake Vogel, Nick Friedman
source_ticket: DBD-53 (Wave 4 — Deal Desk & Discount Governance)
related: [[2026-06-22-revops-gap-roadmap]] · [[bbys-overview]] · [[hubspot]]
source: HomeLight-Vault/sops/2026-06-22-discount-exception-policy.md
imported: 2026-09-29
---

# BBYS Discount & Exception Policy (Draft v1)

> **Purpose:** codify today's ad-hoc discount and exception rules into one written
> policy so pricing decisions are consistent, auditable, and not dependent on any
> single person's memory. This is the **first milestone of Wave 4** — the policy
> comes *before* any approval-workflow tooling.
>
> **Status:** DRAFT. Dollar/bps thresholds and the margin floor are marked
> `[DECISION NEEDED — Jake/Friedman]`. The one-pager exists to ratify those.

## 1. Standard pricing baseline

BBYS comp/fee is driven primarily by the deal's **`bbys_opportunity_source`**
property (see [[hubspot]]):

| `bbys_opportunity_source` | Treatment | Rationale |
|---------------------------|-----------|-----------|
| **Lender** | Full comp / standard fee | Core LO-sourced channel |
| **Agent** | **25% comp discount** | Agent-sourced; lower LO involvement |
| **Client (D2C)** | **$0 comp** | No partner to compensate |

The standard BBYS fee percentage lives on the deal (`bbys_fee_percentage`, e.g.
2.5%). Any deviation from the baseline above is a **discount** and is governed by
this policy.

## 2. Known discount types in force today

| Discount | Current rule | Owner | Notes |
|----------|--------------|-------|-------|
| **Agent-source discount** | 25% off comp | Automatic (property-driven) | Not an exception — baseline |
| **D2C zero-comp** | $0 comp | Automatic (property-driven) | Not an exception — baseline |
| **UWM conflict split** | **50/50 commission split on the first 3 conflicted deals** between AE relationship and existing LO assignment | Jake / LSM | AE relationship trumps existing LO assignment (key 2026 decision) |
| **Promotional bps discount** | Time-boxed promos (e.g., **50 bps** through ~Apr 14) | Marketing + Jake | Must have explicit start/end dates and a tracked promo code/segment |
| **One-off deal exception** | Discretionary | **Jake (today)** | The case this policy is meant to formalize |

## 3. Approval matrix `[DECISION NEEDED — Jake/Friedman]`

The discount magnitude determines who must approve. **Proposed** tiers (confirm at sign-off):

| Discount magnitude (off standard comp/fee) | Approver | Logging |
|--------------------------------------------|----------|---------|
| Baseline (Agent 25% / D2C $0) | None — automatic | Property-driven |
| Up to `[X]%` or `[X] bps` | LSM | Logged on deal |
| `[X]%`–`[Y]%` | Jake (Head of Lender Relations) | Logged on deal + Slack channel |
| Above `[Y]%` / below margin floor | Friedman (GM/VP) | Logged + written justification |

> Today, Jake holds blanket exception authority. The matrix's job is to delegate
> the small/routine cases downward and escalate only the material ones — so Jake
> isn't the bottleneck and the big ones get the right eyes.

## 4. Margin guardrail `[DECISION NEEDED — Jake/Friedman]`

- **Margin floor:** no deal may be discounted below `[$____ / ____% effective fee]`
  without GM sign-off.
- Below the floor, approval **escalates one tier** regardless of who initiated it.
- Rationale: as Builder/UWM volume scales, undocumented stacked discounts are the
  most likely place revenue leaks quietly.

## 5. Exception logging (interim, pre-tooling)

Until the Wave 4 approval-workflow tooling ships (later DBD-48 child), log every
non-baseline discount with:

- Deal ID + LO/partner
- Discount type + magnitude (% or bps)
- Reason / justification
- Approver + date
- Where: the deal's Slack channel **and** a deal property/note (interim:
  free-text note; the tooling ticket will formalize a structured field).

## 6. Out of scope for this doc (handled in later Wave 4 tickets)

- The **approval workflow** (request → approve → log via Slack + Helper command bar).
- The **discount-leakage report** (margin guardrail monitoring).
- Structured HubSpot fields for discount/exception capture.

## 7. Open items for sign-off

1. Fill the approval-matrix thresholds (§3).
2. Set the margin floor (§4).
3. Confirm whether any **LO-tier exceptions** exist today (none documented — §reserved).
4. Confirm the interim logging location before tooling exists (§5).
