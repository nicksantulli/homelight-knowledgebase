---
date: 2026-08-26
type: project
status: active
owner: Tulli
tags: [bbys, hubspot, n8n, reporting, sales-app]
source: HomeLight-Vault/projects/2026-08-26-bbys-failure-taxonomy.md
imported: 2026-09-29
---

# BBYS Deal Failure Taxonomy → HubSpot

## Goal
Every BBYS deal failure recorded in Sales App lands in HubSpot as structured, reportable
fields — category, reason, sub-reason, notes, timestamp — without breaking the reports that
already run off the legacy `reason_for_fail` property.

## Background
Dave Spivey announced the new deal-failure picker in Slack on 2026-08-26 14:00 ET: selecting
Disposition → "Failed" now shows a tiered left-to-right picker (category → reason → sub-reason),
and an envelope icon on a sub-reason means it auto-sends an "Application Declined" email to the
LO quoting the reason plus any notes. This closes the long-running Mette/Marc workstream to
align Sales App dispositions with HubSpot failure reasons (was tracked as "Q2 2026 launch" in
[[bbys-edge-cases]] and [[q2-priorities]]).

Marc supplied the full slug/label list by DM. Shape:
- **6 categories → 13 reasons → 56 sub-reasons.** No slug collisions anywhere.
- Persisted in Crunchy `bbys_deal_failures`: `bbys_lead_id`, `reason_slug`, `reason_label`,
  `sub_reason_slug`, `sub_reason_label`, `notes`, `schema_version`, `recorded_by_id`,
  `superseded_at`, `created_at`, `updated_at`.
- **Category is NOT persisted** — Sales App keeps it in the picker YAML only. It is UI grouping.
- Current disposition = `superseded_at IS NULL`. Re-dispositioning supersedes the prior row.

## Current Status
🟢 Shipped and verified end-to-end. Both backfill passes complete — every failed BBYS deal now
carries the taxonomy. One open gap (`test_deal` exclusion) and one upstream finding
(`provider_leads.stage`) below.

## What Shipped

### HubSpot — 5 new deal properties (group `bbys`)
| Label | Internal name | Type |
|---|---|---|
| BBYS Failure Category | `bbys_failure_category` | Dropdown (6) |
| BBYS Failure Reason | `bbys_failure_reason` | Dropdown (13) |
| BBYS Failure Sub-Reason | `bbys_failure_sub_reason` | Dropdown (56) |
| BBYS Failure Notes | `bbys_failure_notes` | Multi-line text |
| BBYS Failure Recorded At | `bbys_failure_recorded_at` | Date/time |

`reason_for_fail` was **kept and seeded**, not retired: 32 → 91 options, adding all 60 new
taxonomy values with human-readable labels (`Lost to competitor - UpEquity`). Two pre-existing
eyesores fixed while in there: `non_cc_agent` → "Non-CC Agent", and the double space in
"Denial  - Acreage".

Option labels were normalised to sentence case with acronyms preserved (DR, IR, EU, EB, LO,
HOA, DOM, DTI, BBYS, HELOC). Competitor brand spellings left exactly as Marc supplied them.
Notable rewrites: the run-on `Additional liens found no other option to increase file no longer
workable` → "Additional liens found - file no longer workable"; the redundant "Ineligible
property type (…)" prefix dropped since the parent reason already says it.

### n8n — the two sync workflows that write failure data
- `XQ6UmT3N9GZxHTwGkQ6hu` **HS BBYS Applications Data Sync**
- `yaFIQChaTaXq69OG` **… (manually created deals)**

Both gained: the `bbys_deal_failures` LATERAL join, the five new properties in the payload
builder, the derived category, the enum allow-list guard, and a label-aware `Add new option`.
Full node-level detail in [[n8n-workflows]].

Confirmed **no change needed** in three others: `F7SFAkQybBC1et2Ts_ClA` (creation-only sync),
`cdxpjfJfPEzlue5S` (writes `dealstage` only), `C3vEor4LUH9E1kBT` (milestone dates only).

### Backfill — both passes complete
**Pass 1, picker-sourced** (`mI4bjYukUz5t1QBL`): 27 Crunchy rows → 26 deals patched, 129 field
writes, 1 already current, 0 failures. Verified 27/27.

**Pass 2, legacy-mapped** (`MPWACoMgbylt9M25`): 19,862 historical failed leads →
**18,515 deals patched, 54,650 field writes, 0 failed batches.** 882 legacy values deliberately
left unmapped (`other` 540, `approval_denial_other` 341, `canceled_ir_inspection` 1); 466 Crunchy
leads have no HubSpot deal.

`bbys_failure_source` (created 2026-08-26) distinguishes the two: `sales_app_picker` = an operator
actually chose it; `legacy_mapped` = derived from the old vocabulary. **Category is reliable
across all history; reason is present only where the legacy value mapped 1:1; sub-reason was
never inferred.** Do not treat a mapped value as a recorded disposition.

Coverage of the mapping: category **90.0%**, reason **60.6%**, sub-reason **25.2%**. The two
largest legacy buckets — `unresponsive` (6,194) and `decided_program_not_needed_for_move` (3,706),
47% of history — are unknowable below category level, so they were left blank rather than guessed.

The mapping itself was revised once after review. The first draft made two class errors worth
remembering: it let `approval_denial_high_price_point` land in *"Fee or cost objection"*, which
reframes **"HomeLight declined at underwriting"** as **"the client complained about price"** —
opposite stories in a funnel report; and it was too conservative on
`decided_program_not_needed_for_move`, which does map at reason level ("not needed **for move**"
means the move happened → `client_transacted_without_bbys`). Full corrected table in
[[decisions/2026-08-26-failure-taxonomy-hubspot-mapping]].

## Gotchas Worth Remembering

1. **The join key is not the obvious one.** `bbys_deal_failures.bbys_lead_id` is
   `bbys_leads.id` (= `provider_leads.providable_id`), **not** `provider_leads.lead_id`.
   Measured: the intuitive join matched **0 of 26** rows. Since the sync's WHERE clause keys on
   `pl.lead_id`, getting this wrong yields a silently-always-null field.

2. **Sales App dual-writes the legacy column.** `provider_leads.reason_for_fail` now receives
   `reason_slug:sub_reason_slug` (colon-compound), or bare `reason_slug` when there is no
   sub-reason. This is why nothing had to cut over.

3. **Enum writes have whole-record blast radius.** `bbys_failure_reason` / `_sub_reason` are
   `select` properties; HubSpot rejects an off-list value with a 400 that fails the **entire**
   deal PATCH, not just that field — identical failure mode to the year-3036 date bug already
   guarded in the payload builder. The sync now validates against an allow-list and omits the
   whole failure block on an unknown slug (never writes, never clears), flagging
   `_unmappedFailureReason`. **Three things must stay in step: the maps in the two n8n Code
   nodes, the Sales App picker YAML, and the HubSpot picklist options.**

4. **The dropdown self-heal node was a latent mess-maker.** `Add new option` appended unseen
   values using the *raw slug as the label* — that is where `non_cc_agent` came from. It had
   not yet fired for the new taxonomy when this work started; left alone it would have filled
   the picklist with `insufficient_equity_unlock:didnt_pursue_eb_heloc`-style entries.

5. **n8n `sqlEditor` fields evaluate `{{ }}` with no `=` prefix.** The MCP validator flags
   these as `MISSING_EXPRESSION_PREFIX`; for SQL fields they are false positives. Do not "fix"
   them. `jsonBody` on httpRequest nodes genuinely does need the `=`.

## ⚠️ The new picker does not set `provider_leads.stage = 'failed'`

Verified 2026-08-26: all 27 deals dispositioned Failed through the new picker still carry their
**pre-failure** stage in Crunchy (`approved` 14, `new` 6, `in_review` 5, `ir_contract` 2) — hours
after the fact, so this is not lag. They have `bbys_deal_failures` rows and a populated
`reason_for_fail`, and HubSpot shows them at dealstage `998815824` (Failed), but `pl.stage` was
never advanced.

**Consequence:** anything Crunchy-side that defines "failed" as `pl.stage = 'failed'` will
silently *flatline* as picker adoption grows — it will keep counting the 21,045 legacy failures
and none of the new ones. This may be deliberate (separating "how far did it get" from "did it
fail"), but it is a behaviour change either way. **Raise with Dave Spivey / Marc Kaplan.**

Useful side effect: it makes the legacy-vs-new split trivially clean, which is how the legacy
backfill scoped its population without risk of touching picker-sourced deals.

## Open Questions

- **No single field reports cleanly across both eras.** Deals failed before 2026-08-26 have
  only the legacy `reason_for_fail` vocabulary and blank `bbys_failure_*`; deals failed after
  have both. `reason_for_fail` spans both but its vocabulary changes mid-series, so grouping by
  it mixes 32 legacy values with 60 compound ones and splits the same real-world reason across
  two labels. Proposed direction: backfill `bbys_failure_category` across history from a
  legacy→category map (the big volume drivers map unambiguously at *category* level even when
  the reason is unknowable), backfill `bbys_failure_reason` only where the legacy value maps
  1:1, never invent a sub-reason, and add a provenance flag so a mapped value is never mistaken
  for a recorded one. **Not yet built — needs sign-off.**
- Should the nightly parity repair (`C3vEor4LUH9E1kBT`) be extended to reconcile the failure
  block, or is the one-off backfill plus the webhook sync enough?

## Blockers / Follow-ups
- [x] **`test_deal` exclusion — FIXED on branch `fix/test-deal-exclusion` (local, unpushed).**
      The vault said three places; it was **~69 sites across 37 files** in five different
      literal shapes, including the `src/semantic/bbys/**` model, with
      `EXCLUDED_REASONS_FOR_FAIL` independently redefined in four modules. Rather than typing
      the slug 69 times, the fix adds one source of truth — `src/utils/dealExclusions.ts` —
      and points all 37 files at it. A new parity test fails (and names the file) if the
      semantic YAML ever drifts from it; proved by deliberately drifting `velocity.yml`.
      `admin/` keeps a local mirror because its build is isolated (`include: ["src"]`).
      Verified: tsc clean, 427 test files / 3,799 tests pass (+16 new).
      **Needs review + push + PR.** Nothing had leaked — no `test_deal` rows existed yet.
      Long-term: filter on `bbys_failure_category = 'administrative'` and retire the slug list.
- [ ] Add the five fields to the deal record UI (labels above).

## Tools / Systems Involved
- HubSpot (deal properties, picklists)
- n8n (`homelight.app.n8n.cloud`)
- Crunchy Postgres (`bbys_deal_failures`, `provider_leads`, `bbys_leads`)
- Sales App (disposition modal, picker YAML)

## Related Notes
- [[decisions/2026-08-26-failure-taxonomy-hubspot-mapping]]
- [[hubspot]]
- [[n8n-workflows]]
- [[bbys-edge-cases]]
- [[decisions/2026-08-18-ir-closed-date-incident]]
