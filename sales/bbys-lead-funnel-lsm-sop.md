---
date: 2026-05-15
type: sop
owner: Lender Sales
last_reviewed: 2026-05-18
tags: [bbys, lead-scoring, mql, sal, sql, lsm, sales, events]
source: HomeLight-Vault/sops/2026-05-15-bbys-lead-funnel-lsm-sop.md
imported: 2026-09-29
---

# SOP: LSM Use of the BBYS Lender Qualification Lead Queue

## Purpose

This SOP tells LSMs how to work the new `[BBYS] Lender Qualification` queue. The queue replaces the old "engaged equals qualified" model with a cleaner MQL -> SAL -> SQL process and a P-tier priority system.

The point is simple: work the best opportunities first, log the activity that proves what happened, and move each Lead to the right next state.

## When to Use

- At the start of every workday
- After meetings or call blocks
- Any time a new Lead lands in the `[BBYS] Lender Qualification` pipeline
- When deciding whether to qualify, pause, or disqualify an LO
- Before manually creating a Lead for outbound work

## Prerequisites

- Access to HubSpot Leads and Contacts
- Access to your assigned `[BBYS] Lender Qualification` view
- Ability to log calls, emails, meetings, and notes in HubSpot
- Understanding of BBYS basics and the Magic 3 activation goal
- Awareness that event-sourced Leads can enter the queue at New, Attempting, or Qualified depending on the event activity type

## Quick Definitions

| Term | Meaning |
|------|---------|
| MQL | Marketing Qualified Lead. The system says this LO deserves LSM review. |
| SAL | Sales Accepted Lead. You have a real sales engagement signal, not just a name in the queue. |
| SQL | Sales Qualified Lead. You confirmed fit and a credible path to revenue. |
| P-tier | The priority level telling you what to work first. |
| Sales Responsiveness | Hot / Warm / Cold / Unresponsive label based on logged activity. |
| Event Lead | A Lead created from an uploaded in-person event list. Conference = New, booth/raffle = Attempting, dinner = Qualified. |

## Priority Rules

Work Leads by P-tier before working names you personally recognize.

| Tier | SLA language | What to do |
|------|--------------|------------|
| P0 | Drop everything | Work first. These are rare escalations or highest-intent Leads. |
| P1 | Same day | Work today with personalized outreach. |
| P2 | Next day | Work by next business day. |
| P3 | End of week | Standard sales work. |
| P4 | Best effort | Work when higher tiers are clear. |
| P5 | Nurture only | No proactive sales push unless a new signal changes the picture. |

Do not use the raw MKT Score as your work order. The P-tier already combines fit, score, persona, and routing rules.

## Start-of-Day Routine

1. Open your `[BBYS] Lender Qualification` Lead view.
2. Sort by `lo_lead_priority`, then newest/oldest depending on the view design.
3. Scan P0, then P1, then P2.
4. For each high-priority Lead, review:
   - LO name and company
   - Lead type and current stage
   - MQL channel/activity/source detail
   - Persona / ICP if visible
   - Any prior BBYS apps or closes
   - Existing Contact owner and recent activity
5. Build your call/email block from the top of the queue.
6. Log every touch in HubSpot.

## Working a New MQL or Event Lead

### 1. Review why the Lead exists

Check the MQL attribution fields:

- `lo_qualification_funnel_mql_channel`
- `lo_qualification_funnel_mql_activity`
- `lo_qualification_funnel_mql_source_detail`

Examples:

- Email / Click / subject line
- Web / Page View / pricing or portal URL
- Event / Attended / event name
- Direct / Fit-Only / no recent behavior found

Use this as your opener. "Saw you were looking at..." is usually better than a generic BBYS pitch.

If `hs_lead_type = "[Lender] Event"`, use the event name and activity as the context:

| Event activity | Starting stage | How to treat it |
|----------------|----------------|-----------------|
| Conference attendee | New | Work as an MQL with event context. |
| Booth visit / raffle entry | Attempting | Treat as an active sales attempt is warranted; follow up quickly and log the touch. |
| Dinner attendee | Qualified | Treat as high intent, but still confirm fit, next step, and revenue path before considering the cycle worked. |

### 2. Confirm no obvious mismatch

Before outreach, check for:

- Already active BBYS Deal
- Existing active Lead
- Not actually an LO
- Bad data
- Current customer / Land or Expand context

If something looks wrong, do not work around it by creating another Lead. Flag it to RevOps.

### 3. Make the first touch

Use the highest-value channel available:

- Call first for P0/P1 when phone exists.
- Email with a specific reason tied to the attribution signal.
- Book a discovery/demo when the LO engages.

Do not mark the Lead as accepted just because you attempted outreach. SAL requires a real engagement signal.

### 4. Log the activity

The system depends on logged activity. Log:

- Calls, including disposition and duration
- Voicemails
- Emails / replies
- Meetings booked and attended
- Notes from discovery
- Any reason the Lead should be paused or DQ'd

If it is not logged, RevOps and Sales leadership cannot distinguish "worked and no response" from "not worked."

## Moving from MQL to SAL

A Lead becomes SAL when there is a real sales engagement signal, such as:

- Call answered with real conversation
- Meaningful email or SMS reply
- Meeting booked or attended
- Discovery conversation started
- SPICED conversation in progress

The Sales Responsiveness label should become Hot, Warm, Cold, or Unresponsive based on logged activity. You should not force a label manually as a shortcut. Log the activity and let the workflow classify it.

Some Event Leads may already start in Attempting because the event activity itself was strong enough to warrant immediate sales follow-up. That does not remove the need to log your actual outreach and conversation.

## Moving from SAL to SQL

Move the Lead to Qualified / SQL only when you confirm:

- The LO understands BBYS at a usable level.
- The LO sees value in the product for their clients.
- The LO has a credible path to revenue, such as a ready client, near-term referral opportunity, or a BBYS Deal created within the active Lead window.
- The next step is onboarding, application motion, or a specific revenue-producing follow-up.

Do not qualify only because the LO is friendly, high-production, or clicked emails. SQL means credible revenue path, not general interest.

Dinner-sourced Event Leads may already land in Qualified because attendance is treated as a high-intent signal. Still document the follow-up, confirmed fit, and next step so the record is usable for reporting and future re-entry.

## Pausing vs Disqualifying

Use Paused when the LO is not a bad fit, but timing is the issue.

| Situation | Action |
|-----------|--------|
| Bad Timing | Move to Paused for 30 days. |
| Competitor Considering, not exclusive | Move to Paused for 90 days. |
| Exclusive competitor contract | DQ as Competitor (Exclusive). |

Paused Leads auto-requeue after the timer. They are not DQs and should not be used to judge model quality.

## Disqualification Reasons

Use one of the standard DQ reasons:

| Reason | When to use |
|--------|-------------|
| No Response | All channels exhausted; no substantive engagement. Can be auto-DQ'd when Unresponsive. |
| Not Interested | LO engaged and clearly declined. |
| Not a Good Fit | Product economics or LO profile make BBYS a poor fit. |
| Already a User | Duplicate or already active BBYS user. |
| No Longer an LO | Left the industry or license inactive/revoked. |
| Lead Expired | Stale Lead aged out before acceptance. |
| Competitor (Exclusive) | Exclusive competitor arrangement blocks BBYS. |
| Bad Data | Invalid email/phone, wrong person, test/spam record. |

Always add enough context in notes for future re-entry. "Not interested" without why is not useful six months later.

## Manual Outbound Leads

Manual outbound is allowed, but guardrails still apply.

Before creating a manual Lead:

1. Search for an active Lead for the LO.
2. Check whether they have an active BBYS Deal.
3. Confirm they are not already Land / Expand in a way that should be handled as expansion.
4. Confirm Contact ownership.
5. Set the Lead source / type accurately so it is not confused with system-generated MQLs.

If you are unsure whether a manual Lead will duplicate something, ask RevOps before creating it.

## Win-Back Leads

Win-Back Leads are re-engagement opportunities for At Risk / Churned LOs. Treat them differently from cold net-new prospects:

- Review prior BBYS history before outreach.
- Reference their past usage when relevant.
- Look for a reason to restart the relationship, not a generic intro.
- If a prior issue caused dormancy, document whether that issue still exists.

Win-Back Leads still use the same P-tier queue. Work the priority first.

## What Not To Do

- Do not create duplicate Leads because a Contact "looks important."
- Do not move every MQL straight to SAL.
- Do not move SAL to SQL without confirmed fit and credible revenue path.
- Do not use "No Response" after one or two attempts.
- Do not use DQ when the right state is Paused.
- Do not manually relabel Hot/Warm/Cold because it "feels right."
- Do not work P4/P5 names while P0/P1/P2 are aging.
- Do not rely on old legacy Lender Sales pipeline views for new work.

## Common Issues & Troubleshooting

| Issue | What to do |
|-------|------------|
| Lead has no priority | Flag RevOps. Do not guess the tier. |
| Lead looks duplicated | Stop and flag RevOps with both Lead links. |
| The LO is already working a BBYS Deal | Treat it as deal follow-up / expansion, not net-new qualification. |
| The attribution says Fit-Only | Use persona/fit as the reason for outreach, but do not overstate recent behavior. |
| The Lead came from an event | Reference the event and activity type. Do not assume it was score-generated. |
| Sales Responsiveness label looks wrong | Confirm all calls, emails, meetings, and notes are logged. Then flag RevOps if it still looks wrong. |
| LO says "not now" | Use Paused / Bad Timing when there is future potential. |
| LO says they use a competitor | Use Paused if considering; DQ only if exclusive arrangement blocks BBYS. |
| No phone number | Use email and flag bad/missing phone data where appropriate. |

## Owner & Review

- **Owner:** Lender Sales
- **Operational owner:** LSM team
- **RevOps owner:** Nick Santulli / Gui Batista
- **Sales leadership:** Wei
- **Review cadence:** Monthly during rollout, quarterly after stabilization
- **Last reviewed:** 2026-05-18

## Related Notes

- [[bbys-lender-qualification-funnel]]
- [[lo-lifecycle]]
- [[bbys-overview]]
- [[hubspot]]
- [[team]]
