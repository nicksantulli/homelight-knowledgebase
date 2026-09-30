---
date: 2026-05-15
type: sop
owner: RevOps
last_reviewed: 2026-05-18
tags: [bbys, lead-scoring, mql, sal, sql, hubspot, revops, events]
source: HomeLight-Vault/sops/2026-05-15-bbys-lead-funnel-revops-sop.md
imported: 2026-09-29
---

# SOP: RevOps Operation of the BBYS Lead Funnel and Lead Scoring System

## Purpose

Keep the BBYS Lender Qualification funnel healthy after launch. This SOP covers how RevOps monitors the MQL gate, score-based and event-sourced Lead creation workflows, P-tier priority, Sales Responsiveness labels, re-MQL paths, DQ hygiene, and calibration checkpoints.

The goal is not "more MQLs." The goal is a trusted, auditable queue that helps LSMs work the right LOs first and ties qualification back to IR Contracts.

## When to Use

- Daily during the active rollout period
- Any time Sales reports missing, duplicate, stale, or mis-prioritized Leads
- Before changing scoring rules, P-tier logic, DQ logic, Sales Responsiveness weights, or workflow enrollment criteria
- During the 2-week, 30-day, 60-day, and 90-day calibration checkpoints

## Prerequisites

- Access to:
  - HubSpot workflows, properties, Lead pipeline, and workflow execution history
  - HubSpot reports for Leads, Contacts, and workflow performance
  - n8n event scoring workflow logs
  - Slack `/event-upload` batch summaries and the n8n Lender Event Activity workflow
  - Data Bridge / Hub CRM mirror dashboards where applicable
  - HomeLight-Vault context docs
- Context to read first:
  - [[bbys-lender-qualification-funnel]]
  - [[hubspot]]
  - [[hubspot-workflows]]
  - [[lo-lifecycle]]
  - [[team]]

## System Quick Reference

| Layer | Canonical place | RevOps responsibility |
|-------|-----------------|-----------------------|
| Bowtie phase | Contact `lender_lifecycle_phase` | Ensure W-A / W-B / W-C keep phase aligned. Do not report off `lifecyclestage`. |
| MKT Score | Contact `[Lenders] Marketing Score` / `lenders_marketing_score_total` | Monitor distribution and score-crossing behavior. HubSpot Score criteria are edited in the UI, not through API. |
| MQL Trigger | Contact workflow | Confirm eligible score crossings create Leads only after guardrails pass. |
| Event intake | Slack `/event-upload` -> n8n Lender Event Activity | Audit on-demand event batches, stage mapping, guardrail skips, and row-error summaries. Event Leads use `hs_lead_type = "[Lender] Event"` and the same guardrails as MQL Trigger. |
| Lead queue | Lead pipeline `866607668` | Keep new lender qualification work in one active pipeline. |
| Priority | Lead `lo_lead_priority` | Confirm every new Lead gets P-tier quickly and Sales views sort correctly. |
| Sales Responsiveness | Lead `hs_lead_label`, `lo_sales_responsiveness_points` | Confirm labels are signal-derived and not skewed. |
| Re-entry | Re-MQL Sweep / Win-Back MQL Trigger | Confirm cooldown and win-back paths create Leads without duplicates. |
| DQ / pause | Lead `lender_dq_reason`, `dq_date`, Paused stage | Keep DQ reasons clean and separate Bad Timing / Competitor Considering pauses from true DQs. |

## Daily Operating Routine

### 1. Check workflow health

Open HubSpot workflow performance and inspect:

- MQL Trigger
- Priority Matrix Writer
- Re-MQL Sweep
- Win-Back MQL Trigger
- Sales Responsiveness Label
- Sales Responsiveness - Unresponsive Sweep
- W-A / W-B / W-C bowtie workflows
- n8n event scoring workflow
- Lender Event Activity workflow, when an event CSV was uploaded

Escalate immediately if:

- Any custom code action errors
- MQL Trigger volume drops more than 50% vs prior day without an expected reason
- Priority Matrix Writer errors or leaves non-trivial new Leads without `lo_lead_priority`
- Unresponsive Sweep suddenly produces a large spike in Auto-DQs
- n8n event scoring misses a scheduled run
- Any `/event-upload` batch reports row errors, unexpected stage counts, or guardrail-skip patterns that do not match the event list

### 2. Validate new Lead creation

Filter HubSpot Leads:

- `hs_pipeline = 866607668`
- `createdate = today`

Check:

- Every Lead has exactly one primary Contact association.
- `hubspot_owner_id` is populated.
- Nick fallback owner volume is small. If fallback volume rises, investigate upstream Contact ownership.
- `hs_pipeline_stage` starts at New (`1297783695`).
- `hs_lead_label` is blank at creation.
- MQL attribution fields are populated unless intentionally `Fit-Only`.
- No duplicate active Lead exists for the same Contact in the current or legacy lender Lead pipelines.

If duplicates appear, pause the Lead-creation workflow causing the duplicate before cleaning records. Duplicate prevention is load-bearing.

### 3. Validate priority assignment

Filter active Leads in New stage where:

- `lo_lead_priority` is unknown, OR
- `lo_priority_last_computed_at` is unknown

Expected result: near zero after the writer has had time to run.

If records are stuck:

1. Open the workflow execution history for Priority Matrix Writer.
2. Confirm the Lead has a primary Contact association.
3. Confirm the associated Contact has `lender_icp` and `[Lenders] Marketing Score`.
4. Confirm the Lead is in pipeline `866607668`, stage `1297783695`.
5. Re-enroll affected Leads manually after fixing the root cause.

Review P-tier distribution daily during rollout. P0 should remain rare. A sustained P0 increase means escalation overuse or matrix under-prioritization.

### 4. Validate Sales Responsiveness

Check active Leads with a non-blank `hs_lead_label`.

Watch for:

- Hot > 80% of labeled Leads. This was Wei's launch-call red flag.
- Leads with `lo_sal_date` populated but `hs_lead_label` blank.
- Leads with `hs_lead_label` populated while still in New.
- Cold Leads older than 21 days that are not transitioning to Unresponsive when outreach / age gates are met.
- Auto-DQ `NO_RESPONSE` count spiking above prior-week baseline.

If labels look wrong, do not manually relabel in bulk. First verify activity data, points calculation, and workflow enrollment. Labels should move because reps log calls, meetings, replies, and attempts.

### 5. Validate MQL attribution

Report on today's new Leads by:

- `lo_qualification_funnel_mql_channel`
- `lo_qualification_funnel_mql_activity`
- `lo_qualification_funnel_mql_source_detail`

Watch for:

- `Fit-Only` > 50% of new MQLs. This means score movement is being driven mostly by fit/property changes rather than recent behavior.
- Blank attribution fields.
- One channel > 70% without a known campaign/event explanation.
- Event channel Leads have the expected source:
  - Score-based event influence from the daily n8n Event Score workflow, OR
  - Direct event upload with `hs_lead_type = "[Lender] Event"` and source detail equal to the uploaded event name.
- Direct event-upload Leads are in the expected stage:
  - `attended_conference` -> New (`1297783695`)
  - `visited_booth` / `entered_raffle` -> Attempting (`1297783696`)
  - `attended_dinner` -> Qualified (`1297783698`)

### 6. Check re-entry paths

For Re-MQL Sweep:

- Confirm daily workflow ran.
- Review Leads created with Re-qualification marker.
- Confirm the live origin marker being used for reporting. Current event/re-entry builds use standard `hs_lead_type`; older docs may mention `lo_mql_origin`.
- Check skip counts for active Lead, active Deal, and <90-day cooldown.
- Investigate high Nick fallback owner rate.

For Win-Back:

- Confirm Leads are only coming from At Risk / Churned populations.
- Confirm Lead type / origin marks Re-engagement.
- Confirm Contact classification moves to Re-engaging or self-heals by W-C within 24 hours.
- Watch for active Deal guard failures. Win-back should not create a Lead for a Contact already in an active BBYS Deal.
- Watch Win-Back fallback owner rate separately. A rate above 20% means prior-owner hygiene is broken in the win-back population.

### 7. Review event-upload batches

When Marketing, Sales, or RevOps uploads an in-person event CSV through Slack `/event-upload`, review the batch summary the same business day.

Check:

- `events_written` equals the expected number of valid attendee rows, minus true row errors.
- Lead counts by stage match the uploaded activity type mix:
  - `leads_mql_created` for conference attendees.
  - `leads_sal_created` for booth visits and raffle entries.
  - `leads_sql_created` for dinners.
- `expansion_count` reflects Land / Expand customers where the event was recorded but no new Lead was created.
- `skipped_orchard`, `skipped_active_deal`, `skipped_active_lead`, and `skipped_cooldown` make sense for the attendee list.
- `error_count = 0`. If errors exist, review the Slack thread, fix the source rows, and re-upload only the corrected rows when possible.
- New Event Leads have:
  - `hs_lead_type = "[Lender] Event"`
  - `lo_qualification_funnel_mql_channel = Event`
  - `lo_qualification_funnel_mql_activity = Attended`
  - `lo_qualification_funnel_mql_source_detail` populated with the event name
  - Owner inherited from Contact, with only small Nick fallback volume

Operational guardrails:

- Only run one event CSV upload at a time. The n8n workflow stores counters in shared workflow state during execution.
- Do not bypass the active Deal, active Lead, Orchard, or 90-day cooldown checks for event lists. Event attendance is high-signal, but it is not allowed to create duplicate active work.
- Remember that event upload and event scoring are separate. `/event-upload` writes the behavioral event and may create a Lead immediately; the daily Event Score workflow updates `lead_scoring_calculation_event_score` on its next run.

### 8. Maintain Sales reporting separation

Keep these lanes separate:

- New-model lane: Leads created in `866607668` after launch
- Legacy clearing lane: active historical Leads finishing in old pipeline(s)
- Historical lane: archived legacy Leads used for audit/cooldown only

Do not judge the new model by legacy backlog outcomes. Do not bulk-migrate legacy Leads unless a separate migration plan is approved.

## Weekly Operating Review

Every week, publish a short readout:

- New MQL volume by day
- MQL persona mix
- MQL channel/activity attribution
- P-tier distribution
- P0/P1/P2 SLA attainment
- Event Leads by stage and source event
- MQL -> SAL conversion
- SAL -> SQL conversion
- DQ reason distribution
- Active Leads with zero reach-out activity
- Owner fallback rate
- Workflow errors and fixes
- Event-upload row errors / guardrail skips, if any
- Marketing Events audience-type backfill progress
- Any proposed changes, with rationale

Keep the writeup concise. The goal is to spot operational problems, not recalibrate the model every week.

## Calibration Cadence

Use the launch date 2026-04-28 as the anchor.

| Checkpoint | Approx date | What to decide |
|------------|-------------|----------------|
| 2-week health check | 2026-05-12 | Plumbing only: workflows, attribution, owner inheritance, duplicate prevention. |
| 30-day operational read | 2026-05-28 | Volume and distribution sanity. No outcome tuning yet. |
| 60-day engagement check | 2026-06-27 | Whether Sales Responsiveness data is stable enough to consider numeric Sales Score. |
| 90-day primary tuning | 2026-07-28 | First meaningful outcome signal. Tune threshold, weights, channel-type logic, deferred items. |
| Quarterly thereafter | Ongoing | Drift, threshold sensitivity, DQ patterns, score contribution. |

Do not tune the MQL threshold before the 90-day checkpoint unless the system is clearly broken.

## Change Control

Before changing scoring or workflow rules:

1. Write the exact problem statement.
2. Pull examples of affected Leads / Contacts.
3. Confirm whether the problem is data quality, workflow enrollment, scoring criteria, or Sales behavior.
4. Check the change against the guardrails in [[bbys-lender-qualification-funnel]].
5. Document the proposed change, expected impact, rollback plan, and owner.
6. Make the smallest change possible.
7. Monitor for 24-48 hours.
8. Update the vault context if the operating model changed.

Never change these casually:

- MQL threshold
- Active Deal / active Lead duplicate guardrails
- 90-day cooldown
- Land/Expand scoring exclusion
- P0/P1/P2 semantics
- DQ reason set
- `lifecyclestage` vs `lender_lifecycle_phase` source-of-truth decision

## Common Issues & Troubleshooting

| Issue | Likely cause | Fix |
|-------|--------------|-----|
| New Lead has no priority | Priority Matrix Writer did not enroll, Contact association missing, or Contact lacks persona/score | Check writer history, association, `lender_icp`, and score; re-enroll after fix. |
| Duplicate active Leads | Guardrail scan missed current or legacy pipeline | Pause creator workflow, identify duplicate source, close/merge duplicate, fix cross-pipeline check. |
| Too many Leads assigned to Nick fallback | Missing Contact owners upstream | Audit Contacts with blank `hubspot_owner_id`; fix assignment/enrichment source. |
| High `Fit-Only` share | Score changing from persona/milestone updates rather than behavior | Validate campaign/event activity ingestion; review recent mass property changes. |
| Sales says labels are wrong | Activity not logged, workflow not enrolled, or points weights too aggressive | Pull activity log and `lo_sales_responsiveness_points`; fix data/enrollment before touching weights. |
| Hot share > 80% | Weights too generous or label being set too early | Review points distribution and SAL trigger conditions. |
| Cold Leads never Auto-DQ | Unresponsive Sweep not running or filters too narrow | Check daily sweep execution, Lead stage filters, age, and outreach counters. |
| Win-Back Lead appears for active customer | Active Deal / lifecycle guardrail failed | Close the bad Lead, inspect workflow logs, patch guardrail. |
| Event attendee did not become Lead | `/event-upload` row skipped by Orchard, active Deal, active Lead, 90-day cooldown, Land/Expand expansion handling, or row error | Check the Slack batch summary and row-error thread before touching workflow logic. |
| Event Lead created in wrong stage | Activity subtype mapping or source CSV value is wrong | Confirm `attended_conference` -> New, `visited_booth` / `entered_raffle` -> Attempting, and `attended_dinner` -> Qualified. |
| Event attendance did not affect MKT Score | Marketing Event audience type missing, n8n Event Score run missed, or event has not completed yet | Confirm `hs_audience_type = Lender`, event status Completed, and daily 02:00 UTC n8n run. |
| Sales asks to lower threshold immediately | Early outcome data is not mature | Hold threshold changes until 90-day primary tuning unless workflow plumbing is broken. |

## Owner & Review

- **Owner:** RevOps
- **Primary operator:** Nick Santulli / Gui Batista
- **Sales stakeholder:** Sales leadership owner TBD after Wei Wang departure on Friday 2026-06-12
- **Marketing stakeholder:** Marketing owner TBD after Mette Adams departure on Friday 2026-06-12; Javy now covers Mette's external-events / Elite / partner-conference scope
- **Review cadence:** Weekly during rollout; quarterly after stabilization
- **Last reviewed:** 2026-05-18

## Related Notes

- [[bbys-lender-qualification-funnel]]
- [[hubspot]]
- [[hubspot-workflows]]
- [[lo-lifecycle]]
- [[projects-active]]
