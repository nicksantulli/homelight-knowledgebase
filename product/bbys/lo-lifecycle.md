---
last_updated: 2026-08-29 (Elite section superseded by [[lo-elite-program]], banner added. Previous: 2026-05-15 — Lead scoring / qualification funnel is now active rollout, not proposed. See [[bbys-lender-qualification-funnel]] for live MQL/SAL/SQL rules, P-tier priority, and workflow map. Previous: 2026-05-11)
type: context
source: HomeLight-Vault/context/lo-lifecycle.md
imported: 2026-09-29
---

> ⚠️ **2026-08-29:** this doc's Elite section is **superseded by [[lo-elite-program]]** —
> its tier populations (152/24/6) predate three exclusion PRs, and "$1,500 per deal" is
> wrong (code says up to $1.5k **once per year**). Read the new doc for anything Elite.

# Loan Officer Lifecycle & Engagement

> Full LO journey from first contact to repeat closer. Claude reads this for LO-related work.
>
> Related: [[bbys-lender-qualification-funnel]] (active MQL/SAL/SQL funnel — the Magic 3 / activation / retention numbers in this file are the same ones the lead-scoring project cites; keep them in sync)

## The LO Funnel

| Stage | Trigger | HubSpot Property | Population |
|-------|---------|-----------------|------------|
| Raw Lead | Enters system (form, enrichment, Modex) | hl_contact_type = Lender | ~38,584 unregistered |
| Registered | Fills registration form or LSM books meeting | lender_registered = true | ~27,303 |
| Educated | LSM conducts demo/education session | lender_educated = true | ~5,732 registered but not educated |
| Onboarded | Portal access + education complete | hl_lender_onboarding_date set | ~6,838 |
| First App | Submits BBYS application | applications >= 1 | 4,580 with apps but 0 closings |
| Activated (1st Close) | IR closed date set on a deal | ir_closings_bbys >= 1 | Retention jumps to ~63% |
| Power User (3+) | 3+ IR closings | ir_closings_bbys >= 3 | 270 LOs |

**Key insight:** 88-94% of portal sign-ups submit their first app the same day they register. But 71.3% of LOs who submit an app never close a deal — this is the biggest churn point.

## Activation & Churn Definitions

- **Activated:** At least 1 IR closing. After first close, retention jumps to ~63%.
- **Repeat/Retained:** 3+ closings → retention jumps to ~85%
- **Dormant:** 30+ days no activity post-first-app
- **Extended dormancy:** 230 days (window being refined)
- **Deactivated:** Manual trigger — contact type set to "Deactivated", owner unassigned, associations removed, associated to PLACEHOLDER company. Reverses on merge.

⚠️ **Numbers disagree with two internal pitch decks (added 2026-08-29, Gamma mining pass —
read-only, via Gamma MCP).** Two Feb-2026 Gamma decks ("Reimagining Our Lead Funnel" and its
v2, "Land & Expand: HomeLight BBYS Qualification System v2") propose a different dormancy
vocabulary: **90 days** post-first-app with no deal progress → At Risk (vs. this file's
**30+ days**), **170 days** post-close → At Risk (not present here at all), **Lapsed** (never
closed) flagged at **365 days**, **Churned** (was active, then stopped) flagged at **230
days**. Only the 230-day figure matches this file's "Extended dormancy" line — the rest do
not appear here. These decks read as **proposals/pitches, not confirmed shipped logic** (they
frame themselves as "the ask" / "Phase 1 of a roadmap"), so this file's code-adjacent 30-day
figure should be trusted over the deck's 90-day one unless someone confirms the proposal
shipped. Flagging per house style rather than silently picking a winner — verify with
whoever owns the dormancy-sweep logic (see [[bbys-lender-qualification-funnel]]) before citing
either number in a deck of your own.

## Automated Touchpoints by Stage

### Registration
- Registration form confirmation (HubSpot workflow)
- LSM notification of new registration (workflow 620822114)
- Per-LSM onboarding campaigns (Richie, Marisa, Tejas, Michael)
- PPC leads → LSM email handoff (workflow 593156880)

### Education & Onboarding
- LSM Demo "Complete" updates education fields (workflow 570505010)
- TAM ↔ LSM education notification (workflow 592689415)
- TAM ↔ LSM onboarded notification (workflow 605606031)
- LRM check-ins: day 1, 3, 7 post-onboard
- Onboarding email drip (concern: "We send so many transactional emails, these could get buried")

### Nurture & Re-engagement
- Lender Insights Reports (quarterly, per-LSM segmented)
- DTI Drop campaigns (32,052 lenders, historically 16-18% click conversion)
- Elite Lender nurture (Active, Grandfathered, Elite tiers)
- "Win in 2026" seasonal campaign
- Previous Day Website Visits list (9 LOs/day — intent signal)
- MQL alerts in #hs-mqls channel (engagement threshold triggers)

## Elite Lender Program

| Tier | Population | Notes |
|------|-----------|-------|
| Elite | 152 LOs | Base tier |
| Diamond | 24 LOs | High tier |
| Obsidian | 6 LOs | Top tier |

**Benefits:**
- $1,500 LO benefit per deal
- Variable pricing with no minimum fee
- 1 free extension on deals
- Grandfathered: LOs with 3+ all-time transactions qualify for 2026

**Infrastructure:**
- ELP Dashboard (ID 19015702) — 790 March apps
- Elite prospect workflows for Prospect → Pitched → Re-engage
- Per-LSM Elite lists (Tiffany: 14, Richie: 25, Tierney: 10)
- Orchard LOs do NOT participate in the Elite program

**Key people:** Andrew Soss (campaigns), Ashwin + Marc Kaplan (product — *Anirudh Bhutani held this until his departure Friday 2026-08-07*), Javy (external events / Elite / partner-conference scope after Mette Adams departure on Friday 2026-06-12). Mette Adams was the prior strategy/dashboard stakeholder and is no longer with HomeLight.

## Lead Scoring & Qualification Funnel (Active Rollout)

- Canonical context: [[bbys-lender-qualification-funnel]]
- MQL gate is live at `[Lenders] Marketing Score` >= 60, launched 2026-04-28.
- New active Lead pipeline is `[BBYS] Lender Qualification` (`866607668`); legacy Lender Sales pipeline `735605687` is read-only for new Lead creation.
- Contact-level MKT Score decides who should enter the sales queue; Lead-level Sales Responsiveness (Hot / Warm / Cold / Unresponsive) takes over after SAL.
- Priority is Lead-level `lo_lead_priority`, P0-P5. Current daily SLA language: P0 drop everything, P1 same day, P2 next day, P3 end of week, P4 best effort, P5 nurture only.
- Re-MQL and Win-Back paths are now explicit workflows instead of manual re-entry: cooldown-expired net-new contacts enter via Re-MQL Sweep; At Risk / Churned contacts enter via Win-Back MQL Trigger.
- DQ reasons are standardized; Bad Timing and Competitor Considering are Paused states, not true DQs.
- Numeric Sales Score and DQ penalty decay remain deferred until enough post-launch Sales Responsiveness / outcome data exists.

## Patterns: What Makes LOs Close

**High performers (3+ closings):**
- High app-to-approval rate (60-85%)
- Agreement count ≈ contract count (strong follow-through)
- Onboarded 12+ months ago
- Often cluster by company (Lower: 5+ top LOs, Arbor: 3+, CCM: 5+)

**Never-closers (4,580 LOs):**
- Often submitted only 1 app that wasn't approved
- Onboarding alone doesn't activate — need first close
- Some have 2-5 apps with 0-1 approvals (product-market fit issues)

## Key Gaps

1. Win-back workflow is now built as a score-crossing path for At Risk / Churned contacts, but the paired cooldown-expiry sweep for the win-back population is still a follow-up.
2. App-to-close churn (71.3%) is the single biggest lever
3. Company-level enablement drives repeat closers but isn't systematized
4. Lead scoring is live, but SQL remains a manual LSM advancement after confirmed fit and credible path to revenue.
5. RevOps taking over sales campaigns from Marketing (transition in progress)

## Related Notes
- [[top-producers]]
- [[bbys-overview]]
- [[q2-priorities]]
- [[team]]
