---
last_updated: 2026-08-29 (Tulli's DM sweep — Andrew Soss thread D08Q6THTLUW added: DTI Drop
counting confirmed stated, an Elite-3 double-write instance as live evidence for the recompute
risk. See §2. Loop-3 reconciliation, same day: §3 row 5 reclassified — the 152/24/6 figures are
a live, never-recomputed MANUAL HubSpot list, not a stale number; [[hubspot]] independently hit
the same three integers and corrected its own "sixth population" framing.)
status: current
source: homelight/hapi@c14079c76c9a94bc1c9686fba7ec32614bde0dd3 (2026-08-28) · homelight/homes-fe@15ece6248beb9fd53aae096b1f9b184b8b980495 (2026-08-28) · homelight/Data-Bridge@4cbb478620fac (2026-08-18, docs/operator/ read directly; dashboard internals via vault projects/2026-07-28-dbd-83-elite-los-dashboard.md, not re-verified against source) · Slack (#the-boyz, #proj-elite-lender-launch, group DM C0ARTCPPQ83, 2026-04-09 to 2026-08-28) · DM D08Q6THTLUW (Andrew Soss), 2026-08-18/19 · exports/2026-08-03-reorg/reorg_comms_list.csv · context/team.md, projects-active.md, lo-lifecycle.md, bbys-pricing-engine.md, bbys-priority-partners-settings.md, partners.md, hubspot-workflows.md, homelight-agent-marketplace.md
scope: The LO Elite Lender Program end to end — benefits, qualification, the five disagreeing population counts, the 2027-01-01 grandfather cliff, pricing/priority/leaderboard interaction, and who (nobody, currently) owns the policy decision. Supersedes the scattered treatment in [[lo-lifecycle]] (stale, 2026-05-15) for anything this doc covers.
source: HomeLight-Vault/context/lo-elite-program.md
imported: 2026-09-29
---

# LO Elite Program — the channel's retention engine, and the cliff nobody's named

> **2026-09-16 HubSpot:** Elite ladder volume now uses `elite_qualifying_ir_closes_ytd` (new-flat DTI first IR close/year only; old-pricing DTI uncapped). Details: [[2026-09-16-elite-qualifying-ytd-hubspot-calcs]]. HAPI portal count still separate until parity.
>
> **2026-09-17:** Manual ± Elite credits via `elite_manual_credit_ytd` + `elite_manual_credit_note` (folds into qualifying only; `ir_closings_ytd` stays raw). Playbook: [[2026-09-17-elite-manual-credit]].


Elite is HomeLight's volume-tier loyalty program for BBYS loan officers: close enough deals in a
calendar year and the LO gets pricing breaks, a fee waiver, priority routing, and public status —
paid for out of program economics, administered by a Flagsmith flag, and currently owned by no one.
This doc consolidates seven scattered sources into one, and states directly what none of them do:
**the 2026 grandfather clause has a hard code-enforced expiry, nobody has decided what replaces
it, and the mechanism that would apply the expiry doesn't run on a schedule.**

Two unrelated "Elite" programs exist at HomeLight — **Agent Elite** (geographic exclusivity for
real-estate agents, `agents.elite_status`) is a different system entirely. See
[[homelight-agent-marketplace]] §"Two unrelated Elite programs." This doc is **Loan Officer
Elite only** — `EliteProgramLevel`, Flagsmith flag `lo-elite-program`, table `loan_officers`.

## 1. What Elite gets you (benefits, code-verified where possible)

🔑 **Thresholds and reward *names* both live in a Flagsmith JSON blob (`lo-elite-program`), not
in code.** `EliteProgramLevel.levels` reads `config["levels"]` — each level has `min`, `max`,
`key`, and a `rewards` array of free-form string keys (`app/classes/elite_program_level.rb:142`).
Nobody granted me a live Flagsmith read for this pass, so the *current* threshold values and the
*current* reward-key list are unverified — the table below is everything that IS pinned in code,
plus what a live UI/DB read confirms.

| Benefit | Mechanism | Verified how | Caveat |
|---|---|---|---|
| **Diamond: automatic inspection-fee waiver** | `LoanOfficer#waive_diamond_inspection_fees!` sets `waive_inspection_fee = true` on the departing property of every **non-closed** BBYS lead the LO owns, the moment their tier flips to Diamond | 🔑 **observed**, `app/models/loan_officer.rb:328-344`, called from `UpdateLoanOfficerRewardTransactionWorker.rb:295` | Only fires on the tier-up event. A lead that was already `ir_closed`/`dr_closed` before the LO hit Diamond is untouched — the waiver does not retroactively rewrite closed-deal economics, only in-flight ones. Also gated by `EliteProgramLevel.excluded_partner?` — Orchard/builder LOs never get it even if their raw count would qualify. |
| **Elite pricing tier — `elite_only` templates** | `PricingTemplate.elite_only` restricts a template to LOs where `current_elite_status` ∈ `elite/diamond/obsidian`; the resolver filters it in before picking lowest `priority` | 🔑 **observed**, `services/lead_data_service/.../resolve_pricing_template.rb:77` + `ELITE_STATUSES = %w[elite diamond obsidian]` | Full mechanics, ramp schedules, and the CRO fork are in [[bbys-pricing-engine]] §5 — not re-derived here. This is almost certainly the code path behind "$1,500 discount / VP with no minimum fee," but which specific `pricing_templates` row is live, and its exact terms, needs a DB read — the resolver only proves *that* elite-gated templates exist and how eligibility is checked. |
| **BBYS discount checkbox — $1,500, capped 1x/year** | `elite_program.bbys_discount_available` flag on the `User` payload gates a checkbox at the `assistant_or_processor_info` application step: *"Apply a one-time discount on this Buy Before You Sell deal, up to $1.5k (1x/year)."* | 🔑 **observed**, `homes-fe/src/features/application-flow/components/sections/BbysEliteDiscountCheckbox.tsx:55` | ⚠️ **Contradicts [[lo-lifecycle]]'s "$1,500 LO benefit per deal."** The UI copy is explicit: once per calendar year, not per deal. lo-lifecycle.md is stale (2026-05-15, predates this component); trust the code. A real instance is in Slack: *"LO Elite discount of $1500 on this one - program fee $7500"* (#tls-iruc-shemitz-31-clover-st-ct-express, Deborah Shutt, 2026-08-28) — confirms the mechanic fires, not the cap. |
| **"One free extension"** | Not found in HAPI code under this name | ⚠️ **unverified** — likely a `rewards` key inside the Flagsmith JSON (see `reward_key`, free-form string, `app/models/loan_officer_reward.rb:9`), which means it cannot be confirmed or priced without a live flag read | Carried forward from [[lo-lifecycle]] as **(stated)**, not (observed). |
| **Leaderboard badge + welcome banner + progress toast** | `EliteGreeting` (badge + gradient banner keyed `elite/diamond/obsidian`), `EliteProgressToast` ("Close N more to reach Diamond / a luxury trip"), leaderboard `eliteLevel` field on ranked entries | 🔑 **observed**, `homes-fe/src/features/application-flow/components/sections/EliteGreeting.tsx`, `.../hooks/useEliteProgressToast.ts`, `src/features/leaderboard/types/leaderboard.ts:16` | The leaderboard itself (`/api/leaderboard`, `rewardProgramName`-keyed) is the separate `RewardProgram` sweepstakes-points system in HAPI, not the Elite tier engine — it just *displays* `eliteLevel` as a badge alongside points rank. Don't conflate; see [[homelight-agent-marketplace]] on `RewardProgram`. |
| **Priority-queue score bonus** | `lo_quality` signal: **elite +3 / diamond +6 / obsidian +10** points in the BBYS lead-priority formula | 🔑 **observed**, [[bbys-priority-partners-settings]] §`lo_quality` | Full formula (IRUC rate, Modex score, 3-lead minimum) already documented there — not re-derived. |
| **Payouts via Trolley** | `DisbursementService::Payment` with `payment_source = "elite_program"`, `category = "referral"` | 🔑 **observed** per [[homelight-agent-marketplace]] | Real payment threads exist: #proj-elite-lender-launch (2026-08-20→24) shows Sarah Jaka/Taylor Wong manually routing an elite referral payment to a personal vs. company Trolley profile — **the payout path still has manual exception-handling**, it is not fully automatic end to end. |
| **Reward *redemption* tracking** | Google-Sheets-reconciled ledger: `src/elite-rewards/`, migration `20260817210509_elite_reward_redemptions.sql`, twice-daily Railway cron `cron-elite-rewards-sync` (fail-closed) | 🔑 **observed**, `Data-Bridge/docs/operator/elite-reward-redemptions.md` (read directly, current HEAD) | Confirms rewards are *claimed* against a manually-maintained Google Sheet, not derived purely from system state — "claimant identity from the Sheet and current deal LO identity remain separate. A mismatch is reviewable, not silently reassigned." Shipped 2026-08-17/18 (PR #725, follow-up #753 per [[tools-and-stack]]). |

> ⚠️ **Don't conflate this $1,500 with the other $1,500 in the vault.** [[partners]] records a
> flat **$1,500/deal** comp override for Arbor Financial Group / KMC Financial, replacing their
> standard 10bps rev-share — that is a partner-level comp arrangement paid *to* the LO's company,
> unrelated to the Elite discount applied *to* the consumer's program fee. Same dollar figure,
> two different mechanisms, two different ledgers.

## 2. How you qualify

- **Qualifying metric**: count of **distinct BBYS leads** owned by the LO that hit `ir_closed`
  **in the current calendar year** — `EliteProgramLevel.count_closed_transactions`, a `Lead`
  join on `bbys_lead_stage_updates.new_stage = "ir_closed"`, excluding `FAILED_STAGES`, filtered
  `>= Date.new(year, 1, 1)` (`elite_program_level.rb:259-274`).
- **Tiers**: `none · elite · diamond · obsidian` (`ALL_STATUSES`, `elite_program_level.rb:24-29`).
  Data-Bridge's independently-built Elite LOs Dashboard uses **Elite = 3+ / Diamond = 6+ /
  Obsidian = 11+** as its own hardcoded ladder (`projects/2026-07-28-dbd-83-elite-los-dashboard.md`)
  — this is Data-Bridge's copy of the rule for reporting purposes, **not** a read of the live
  Flagsmith JSON, so treat it as *stated*, not *observed* against HAPI.
- **Exclusions** (`EliteProgramLevel.excluded_from_elite_program?`, `elite_program_level.rb:36-77`):
  partner slug `orchard`, any partner where `partner.builder? == true`, or LO email containing
  `@orchard`. Excluded LOs return `NONE` outright — the override list below cannot rescue them.
- **Recompute trigger — event-driven, not scheduled.** 🔴 **This is the load-bearing fact for
  everything in §4.** `current_elite_status` is written in exactly one place:
  `UpdateLoanOfficerRewardTransactionWorker#run_elite_program_update`
  (`app/workers/update_loan_officer_reward_transaction_worker.rb:283-297`), enqueued only by
  `LoanOfficerRewardTrackable#should_update_loan_officer_reward_transaction?`
  (`app/models/concerns/loan_officer_reward_trackable.rb`) — which fires **only** when a BBYS
  lead's `stage` transitions to or from `new`, `ir_contract`, or `ir_closed`, **and** the
  `lo-elite-program` Flagsmith flag is enabled. There is **no cron, no scheduled job, no
  batch recompute** anywhere in `app/workers` or `config/` that touches `EliteProgramLevel` or
  `current_elite_status` — confirmed by grep across `app/workers/`, every `services/*/app/workers/`
  tree, and `config/schedule.yml`. A LO's stored tier is a **cache that only updates when that
  specific LO's own book moves a qualifying stage.**
- **Stated confirmation — DTI Drop counts.** Andrew Soss asked Tulli directly (DM D08Q6THTLUW,
  2026-08-19): *"Are we counting DTI drop in Elite LO calculations? ... Every closed transaction
  regardless of DTI or BBYS?"* Tulli: *"Yes."* **(stated, not re-verified against code this
  pass)** — consistent with the `ir_closed`-stage qualifying metric above if DTI Drop leads are
  BBYS leads under the hood (per [[dti-drop]], DTI Drop is a BBYS variant, formerly BYOC), but
  this DM is the only place the business rule is stated in plain language. Same thread: HubSpot
  has **no property yet for "rewards used"** — Tulli was going to create fields off Jake Vogel's
  spec doc, unconfirmed whether shipped.
- 🔴 **New evidence for the recompute-cache risk above, same week.** Andrew Soss, 2026-08-18 (DM
  D08Q6THTLUW), on a specific HubSpot contact: *"Why did HubSpot change to 'Elite-3' twice on
  this record?"* Tulli's read: *"It's a calculated field so the IR close was probably reverted
  and then pushed again from sales app or something like that."* Not a full explanation, but a
  live, dated instance of the tier field writing itself more than once on one record — direct
  supporting evidence for the "cache, not source of truth" framing above, from a different
  instrument (DM) than the code read that established the mechanism.
- ⚠️ **A second, incompatible threshold set exists in `homes-fe`, client-side only.**
  `DIAMOND_MIN_TRANSACTIONS = 3` and `OBSIDIAN_MIN_TRANSACTIONS = 10`
  (`homes-fe/src/features/application-flow/utils/eliteProgram.ts:22-23`) are used **only** as a
  fallback when the API response omits `transactions_to_next_level` — they compute the "close N
  more to reach Diamond" toast copy, nothing else (not eligibility, not pricing, not the fee
  waiver). They disagree with Data-Bridge's 6+/11+ ladder on both numbers. **(inferred)** — `3`
  and `10` are suspiciously close to the `lo_quality` priority-score bonus values (elite=3,
  diamond=6, obsidian=10, §1 table) rather than real transaction thresholds; a plausible origin
  is a copy-paste from the wrong table during that component's build. Low blast radius (cosmetic
  toast text only, and only when the API omits the real number) but a genuine, uncorrected
  three-way disagreement about what the numbers even are.

## 3. The populations — five numbers, five different things

The critic's round found the vault citing "~230" in one place and "414" in another as if they
might be the same fact. They are not. Neither is either of these the same as two more numbers
this pass found. All five are real, all five are dated, and **none of them is wrong** — they
measure different populations at different times through different systems.

| # | Population | Source | Date | What it actually counts |
|---|---|---|---|---|
| **~235** | Annie Dreshfield's original HubSpot grandfather list | Slack, group DM `C0ARTCPPQ83`, Tulli: *"Annie had given me a list of ~235 to set this on... there were a handful of LOs who hadn't met the criteria but Nick + Annie wanted to include"* | 2026-04-09 | The **origin** list — a one-time manual set of the HubSpot contact property `2026 Grandfathered Lender Elite Status` = `True`. **Not purely rule-based even at creation** — it already included discretionary adds beyond the 3+ all-time-transaction bar. |
| **251** | `EliteProgramLevel::OVERRIDE_IDS` | `hapi@c14079c76c`, `app/classes/elite_program_level.rb:9-23` | as of 2026-08-28 | The **current code-side grandfather list** — hardcoded loan_officer IDs force-set to `elite` regardless of computed volume, capped at the elite tier's `max`, and only while `Date.current.year <= 2026`. Counted precisely by script: **251 unique IDs, zero duplicates.** (Earlier vault passes said "~230" — closer to the 235-origin list than to this exact count; the list evidently grew ~16 entries between April and August, cause not found in this pass.) |
| **416 raw / "414" as stated** | `elite_status` column, `exports/2026-08-03-reorg/reorg_comms_list.csv` | 2026-08-03 | The **reorg's Sep-1 comms segmentation population** — every LO whose live HubSpot `elite_status` property is not `Non-eligible`. Breaks down: Active-1/2 (pre-elite watch, 285) + Grandfathered-0/1/2 (106) + Elite/GF Elite/GF Diamond (25) = **416** unique `contact_id`s (400 unique emails, 12 duplicate emails across companies). [[2026-08-04-reorg-CUTOVER-RUNBOOK]] and [[2026-08-28-reorg-GO-PLAN]] both say **414** — 2 short of the raw count in the same file; not chased further in this pass. **This is the broadest of the five populations** — it counts anyone the program is tracking at all, including LOs below the Elite bar who are merely "Active" toward it. |
| **856 total / 55 at "Elite 3+"** | Data-Bridge Elite LOs Dashboard | `projects/2026-07-28-dbd-83-elite-los-dashboard.md`, cohort as of PR #652, 2026-07-29 | The **current-year computed cohort** — LOs on the cumulative 1+…11+ ladder or with ≥1 YTD IR close, **after** Orchard exclusion (PR #643, −30 dupe rows via #646, −68 builder LOs via #652). 856 is the "1+" rung (broadest); 55 is specifically the "3+" (Elite) rung, down from 58 pre-builder-exclusion. **Cohort size has moved three times in 24 hours** (954→924→856) — any number off this dashboard needs its date attached, per the dashboard's own doc. |
| **152 / 24 / 6** | [[lo-lifecycle]] §"Elite Lender Program" (2026-05-15) — and, as of a **2026-08-29 critic re-verification**, the **live** `hs_list_size` on three MANUAL HubSpot lists: `Elite LOs (Elite)` listId `15725` = 152, `Elite LOs (Diamond)` `15723` = 24, `Elite LOs (Obsidian)` `15721` = 6 (see [[hubspot]] §"Marketing/LO Elite tier lists") | 2026-05-15 written; **2026-08-29 confirmed still live and current via `hubspot_search_lists`** | 🔴 **Reclassified 2026-08-29 — this is not a stale number, it is a live static list nobody recomputes.** [[hubspot]] independently found these same three integers on 2026-08-29 via a direct list-size read and initially logged them as a "sixth" population before the critic traced them back to this exact row. The defect is not that the figure is old — a live re-run today returns the identical 152/24/6 — it's that `Elite LOs (Elite/Diamond/Obsidian)` are **MANUAL** lists (static membership, no refresh job) that still drive Elite comms segmentation and will keep returning "current-looking" numbers indefinitely without ever being recomputed against the actual qualification rule. Do not cite 152/24/6 as a computed cohort — it is a snapshot from whenever these lists were last hand-edited, coincidentally unchanged since at least 2026-05-15. |

🔑 **None of these numbers are wrong readings of the same thing.** They differ because they are
five different definitions (HubSpot property vs. code constant vs. computed YTD count),
measured at five different moments (Apr 9 → Aug 3 → Jul 29 → May 15), through at least three
different systems (HubSpot property, HAPI Ruby constant, Data-Bridge mirror computation) that
**do not sync with each other.** There is no single query in any system that would return "the
current Elite population" and be simultaneously true against code, HubSpot, and the dashboard.

## 4. The 2027-01-01 cliff — sized as precisely as this pass could get it

🔴 **The cliff is real, code-enforced, and undated in every business document that references
it.** Two independent code facts define it:

1. `EliteProgramLevel.check_launch_elite_override` — `return nil if Date.current.year > 2026`
   (`elite_program_level.rb:190-193`). The grandfather override for the 251 `OVERRIDE_IDS` LOs
   stops being applicable the instant `Date.current.year` reads 2027 — i.e., **2027-01-01.**
2. `count_closed_transactions` is scoped `>= Date.new(year, 1, 1)` — every LO's *computed*
   qualifying count also resets to zero-so-far on **2027-01-01**, same as it does every January.

**But the cliff does not land at midnight for everyone at once — because of the recompute
mechanism in §2.** `current_elite_status` on the `loan_officers` table is not touched by any
Jan-1 batch job. It only changes the next time that specific LO's own BBYS lead crosses `new`,
`ir_contract`, or `ir_closed`. So on 2027-01-01:

- Every currently-Elite/Diamond/Obsidian LO — grandfathered or earned — **keeps their stored
  tier, keeps `elite_only` pricing eligibility, keeps their leaderboard badge, and (for Diamond)
  keeps whatever inspection-fee waivers were already applied** to their in-flight leads.
- The override that *would* re-grant `elite` to the 251 launch-cohort LOs regardless of volume
  stops firing — but only the **next time** `check_launch_elite_override` is invoked for that LO,
  which itself only happens inside a lead-stage-change event.
- The **first** BBYS lead any of these LOs closes/contracts/opens in 2027 recomputes their tier
  purely off 2027 YTD closes (0, until that lead), *and* the override no longer rescues them —
  so that event is very likely to drop them, unless they already have enough 2027 closes queued.
- **The net effect: a staggered decay through Q1 2027, not a clean reset.** An LO with an active
  pipeline drops fast (their next stage change recomputes them down). An LO who closes nothing
  early in 2027 simply **keeps looking Elite in every system that reads `current_elite_status`**
  — pricing, priority score, leaderboard — for however long their book stays quiet. This is a
  code-verified, unflagged trap: nobody watching the dashboard or the pricing-template resolver
  would know a given "Elite" LO is actually running on a stale cached value from December.

**Sizing it:**

- **251** is the hardcoded population that structurally loses its regardless-of-volume grandfather
  the moment the code path re-evaluates them (§3).
- **416 (414 as documented)** is the broader HubSpot-tracked population getting the Sep-1 reorg's
  personal-handoff treatment ([[2026-08-28-reorg-GO-PLAN]] Stage 5.2) — this is **not** the same
  list as the 251 override IDs (it includes 285 "Active" pre-elite LOs who were never in
  `OVERRIDE_IDS` at all), but it is the closest thing to "who currently identifies as Elite to
  the business" and is a reasonable upper bound on who will *notice* something changed in Q1 2027.
- **% of channel volume**: **could not be sized this pass.** [[partners]] gives concentration by
  *company* (Orchard ~30% of applications, top 4 ~60%) — Orchard is excluded from Elite entirely,
  so that figure is not usable here. No vault doc or table maps Elite-tier LOs to their share of
  total BBYS closed volume or revenue; that requires a query joining `current_elite_status` (or
  the HubSpot equivalent) against closed-deal revenue that this pass did not run. **Flagged as an
  open question below, not answered.**
- The Sep-1 reorg handoff (414 LOs, 2026-09-01) and the grandfather cliff (2027-01-01) are **four
  months apart, same rough population, two different discontinuities** — a LO getting a
  personal rep handoff this week may be a different tier three months later with no comms plan
  for that second event at all.

## 5. Program ownership — currently nobody

🔴 [[team.md]] §"Homes Pods" lists **Elite Lender Program** with its own private Slack channel
**`#elite-lender-program`** and states plainly: *"⚠️ Unassigned — was Anirudh Bhutani, departed
2026-08-07. Javy holds the external-events/partner-conference slice from Mette; the
product/tiering side needs an owner."* [[projects-active]] confirms the same: *"🟡 Live, owner
unassigned... the product/tiering side has no owner."* Ashwin Dayal has Data-Bridge admin access
(PR #721, 2026-08-17) but that is infrastructure access, not a stated program-ownership handoff.

This is not a stale claim — it is current activity, three days before this doc was written:
**2026-08-25, #the-boyz**, Andrew Soss asks *"What is the best way to get data on Elite numbers
this time last year?"*; Guilherme Batista asks *"Ignoring the grandfather elites right?"*; Tulli
himself asks *"Is orchard excluded from 2025? I feel like those 2025 numbers are inflated, I
don't remember that many being grandfathered but maybe I'm wrong."* — leadership is actively
pulling Elite YoY numbers and is explicitly aware the grandfather clause confounds any comparison,
**but no message in this search names 2027, a renewal decision, or an owner for that decision.**

## 6. Interaction map (cite, don't re-derive)

| System | How Elite plugs in | Canonical doc |
|---|---|---|
| Pricing | `elite_only` template eligibility gate | [[bbys-pricing-engine]] §5 |
| Priority queue | `lo_quality` signal: +3/+6/+10 by tier | [[bbys-priority-partners-settings]] |
| Leaderboard | `eliteLevel` badge on ranked entries (separate `RewardProgram` points system underneath) | [[homelight-agent-marketplace]] §RewardProgram |
| HubSpot nurture | Tier × grandfathered nurture sequences (`ELITE - 3 and GF ELITE - 3` etc.), `[Property Update] Last Elite Tier Upgrade` | [[hubspot-workflows]] §23 |
| Payments | Trolley, `payment_source = "elite_program"` | [[homelight-agent-marketplace]] |
| Reward redemption | Google-Sheet-reconciled ledger, twice-daily cron | `Data-Bridge/docs/operator/elite-reward-redemptions.md` |
| Sep-1 reorg comms | 414 Elite LOs get a personal handoff, not the 3,393-LO blast | [[2026-08-28-reorg-GO-PLAN]] Stage 5.2 |

## Open questions

1. **Who decides what happens to grandfathering on 2027-01-01 — renew, sunset, or re-qualify a
   subset?** No owner exists today (§5). Likely candidates by prior scope: Andrew Soss
   (campaigns/marketing execution, actively pulling Elite numbers as of 2026-08-25), Ashwin Dayal
   (holds Data-Bridge admin, GM New Ventures — closest thing to a product owner right now), Marc
   Kaplan (absorbed some of Anirudh's product scope). None has been asked this specific question
   in any source this pass found. **(inferred)** candidate list, not a confirmed assignment.
2. **What % of BBYS channel volume or revenue sits with LOs who would lose tier status if the
   grandfather clause is not renewed?** Not sizeable from vault data alone — needs a join of
   `current_elite_status` (or the HubSpot `elite_status` mirror) against closed-deal revenue that
   this pass did not run.
3. **Why did the grandfather list grow from ~235 (Apr 9, HubSpot) to 251 (Aug 28, code)?** 16
   entries' worth of drift, no changelog or PR found explaining additions.
4. **Why 416 raw vs. 414 stated in the reorg comms list?** A 2-record gap in the same CSV; not
   resolved this pass.
5. **Live Flagsmith `lo-elite-program` JSON was not read.** Actual current `min`/`max` per tier
   and the actual current `rewards` key list (which would confirm or refute "one free extension"
   and pin the exact VP/no-minimum-fee terms) require a live flag read this pass did not have
   access to — same open item [[homelight-agent-marketplace]] already carries.
6. **Does the Sep-1 rep reassignment orphan the 2027 requalification comms?** The 414 Elite LOs
   get a personal handoff from their *outgoing* rep on 2026-09-01 tied to the reorg. Four months
   later their tier may lapse with no comms plan and, per the reorg's own new pod structure, a
   *different* rep than the one who did the handoff. No doc connects these two events.
7. **Which specific `pricing_templates` row(s) are `elite_only` today, and what are their exact
   terms?** The resolver code proves the gate exists; confirming "VP with no minimum fee" as the
   live terms needs a DB read against `pricing_templates`, not done this pass.
