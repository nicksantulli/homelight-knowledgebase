---
last_updated: 2026-08-29
status: current
source: homelight/hapi@c14079c76c9a94bc1c9686fba7ec32614bde0dd3 (2026-08-28)
scope: HomeLight's agent referral marketplace as implemented in HAPI — matching algo, intro/claim mechanics, referral economics, Elite programs, PPL
source: HomeLight-Vault/context/homelight-agent-marketplace.md
imported: 2026-09-29
---

# The HomeLight agent referral marketplace

This is HomeLight's original and largest business: match a consumer to a real-estate
agent, introduce them, and collect a share of the agent's commission at close. BBYS is a
newer product bolted onto the same object model ([[repo-hapi-platform]]).

Everything here is observed in `homelight/hapi` at `c14079c76c`. The **matching algorithm
itself is not in HAPI** — HAPI calls an external "Agent Matching Service" over HTTP.

## The economics in one table

| Mechanic | Value | Where |
| --- | --- | --- |
| **Referral fee, most states** | **33%** of the agent's commission | `Lead#set_commission_split` |
| **Referral fee, big states** | **30%** — `CA IL TX FL WA AZ CO` | `Lead::STATES_WITH_30_PERCENT_FEE` |
| Per-deal split actually charged | `provider_agreement.agreement.commission_split` → `agent_agreement.commission_split` → `agent.active_agent_agreement.commission_split` → **30.0** | `AgentLead#commission_split` |
| Fee collection stages | `closed` → `closed_booked` (revenue recognized) → `closed_paid` (cash in) | `LeadStage` |
| Alternative model | **PPL** — pay-per-lead, flat price per intro, billed via Stripe metered usage | `PplReferral`, `PplProviderSetting` |

> 🔑 **The 30/33 split is a hardcoded constant on `Lead`, assigned at induction**
> (`LeadInductionService::AssignLeadCommissionSplit` writes `leads.commission_split`).
> The *agreement* the agent signed can override it per deal. So `leads.commission_split`
> is the list price and `AgentLead#commission_split` is the realized price — **they can
> disagree**, and only the latter drives billing.

`agreements` table carries `commission_split`, `revenue_model`, `late_fee_percentage`,
`late_fee_days`, and a `states` array. Versions in code: `default` v8.0,
`cujo` v7.0 (WA only), `previous_default` v6.0. Addendums: `tpca` v1.0,
**`tcpa_elite` v2.0**, `tcpa_non_dual` v4.0. Categories: `referral_agreement`, `tcpa`,
`usn_sponsorship`, `usn_sponsorship_terms_and_conditions` (US News sponsorship is a
distinct product line).

## Matching: three algorithm generations running simultaneously

`AgentSearchService::AgentMatches::PerformAgentMatches` is the entry point. It organizes:

```
BuildFilters → FetchZipcodes → BuildInputParams
  → RunAgentMatchingLeadRagnarok   (v6.0)
  → RunAgentMatchingLead           (v5.x / v5.2 legacy)
  → PerformFallbackAgentMatches    (only if agent_matches is empty)
```

| Version | Env var for endpoint | Notes |
| --- | --- | --- |
| **6.0 "Ragnarok"** | `AGENT_MATCHING_SERVICE_RAGNAROK_URL` | current-generation; adds account-health filtering, HubSpot task creation |
| **5.1** (default) | `AGENT_MATCHING_SERVICE_PREFIX_URL`, `CURRENT_AGENT_MATCHING_ALGO_VERSION` default `"5.1"` | |
| **5.2 legacy** | `AGENT_MATCHING_SERVICE_CONTROL_URL` | the fallback/control arm |
| **buyer v1** | `<monolith>/api/v1/hapi/buyer-algo-v1` | still calls the monolith |
| **fallback** | in-HAPI SQL over `fallback_agent_zip_code_stats` | last resort when everything returns empty |

`leads.algo_version` records which one ran, per lead; `agent_leads.algo_version` records
it **per match**. Both are strings.

### Ragnarok gating and fallback ladder — the part that matters operationally

`RunAgentMatchingLeadRagnarok`:

1. **Only runs for sellers**, only if `algo_agent_stats_settings["enabled"]`, and only if
   the lead's zip is in the settings' `zip_codes` list — or, when
   `metroplex_check_enabled`, if a `HubspotTaskSetting` exists for the lead's metroplex.
   **This is a SalesSetting-driven rollout, not a deploy.**
2. If the lead already has any `agent_lead` with `algo_version != "6.0"`, Ragnarok is
   skipped and the lead is pinned to **5.2**.
3. After the call: **if ≥ 3 in-contract matches → success.** Otherwise, either fall back
   to 5.2 (`fallback_to_5_2` setting) — additively if Ragnarok found *some* — or keep the
   thin Ragnarok result and skip legacy entirely.
4. Sellers only: if zero matches, retry once at **`FALLBACK_RADIUS = 20`** miles.

The response is split on `match["in_contract"] == true/false`. **Only in-contract agents
are introduced**; not-in-contract agents become HubSpot onboarding tasks.

### Account health — agents can be paused out of the marketplace

`algo_agent_stats` (one row per agent) holds `account_health_status`,
`account_health_reason`, **`account_health_multiplier`**, `agent_in_contract`,
`agent_elite_status`, plus ~35 performance metrics:

- Volume: `property_count_3y`, `listings_1y`, `wins_1y`, `win_rate_1y`
- Referral flow: `ref_sent_30d/60d/90d`, `closes_30d/90d/180d`, `total_commission_180d`
- Funnel rates: `claim_rate_last_30/90_days`, `connection_rate_last_30/90_days`,
  `overall_meeting_rate_last_30/90_days`, `listing_rate_last_90/180_days`,
  `close_rate_last_180_days`, `median_response_min_last_30_days`
- **"Jarvis" call-quality metrics**: `jarvis_total_calls_30d/60d`, `jarvis_conn_calls_*`,
  `jarvis_ms_calls_*` (meeting-set), `jarvis_conn_to_ms_rate_*`,
  `jarvis_avg_client_interest_score_*`, `jarvis_avg_agent_call_score_*` — AI call scoring
  feeds the matching algo.

`account_health_status == "paused"` agents are **filtered out of legacy (5.2) match
results in HAPI itself** (`filter_paused_legacy_agent_matches`). Ops can override via
`AccountHealth::OverrideAccountHealth`, which POSTs to the Ragnarok service's
`/account-health-override/` and caches locally. A change fires
`AccountHealthChangeNotificationWorker` + push batch, and (if
`account_health_change_ticket` is on for the metroplex) a HubSpot ticket.

> 🔑 **Ragnarok writes into HubSpot directly.** Per-metroplex `HubspotTaskSetting`
> toggles gate four task/ticket types: `insufficient_agents_task`,
> `insufficient_claimed_agents_task`, `coaching_task`, `sign_agreement_task`, plus
> `tickets_enabled`, `hubspot_pipeline_id`, `hubspot_pipeline_new_stage_id`,
> `asa_meeting_scheduler`. Default task owner falls back to a **hardcoded email**
> (`kurvyne.frederick@homelight.com`) when no ASA is mapped. Delays default to 30 min
> (tasks), 60 min (claim onboarding), 120 min (claimed-agents onboarding).

`SelectAgentIdsFromAlgo` takes the **top 15** results.

## Intro mechanics — four ways an agent gets a lead

`IntroTracking` names the taxonomy: `INTRO_TYPES = auto · agent · investor ·
agent_investor · dual_path · manual · client_direct`, with `INTRO_DETAILS =
warm_transfer · waterfall · otherside · quiz_consent · ai · intro_request_message`, and a
qualification type of `blind` vs normal.

### 1. Referral guards — who is allowed to be auto-introduced

`LeadDispositionService::ReferralGuard` is a rules engine with two rule families
(`Rules` = blind, `NormalRules` = normal) and per-provider evaluations
(`Blind::Agent`, `Blind::Investor`, `Normal::Agent`, `Normal::Investor`).

**Blind-intro blockers** (`Blind::Agent::FLOW_RULES[:agent]`): already a deal · duplicate ·
undeliverable address · matches an active `BlindIntroException` · recently sold ·
`potential_agent` (lead is itself an agent — by self-report, email match, phone match, or
**broker-domain email match** via `KnownBrokerageDomain`) · `too_many_blind_attempts`
(**`BLIND_INTO_LIMIT = 2`**, `TIMEFRAME = 6.months`) · disallowed property type ·
client unreachable · an `interested` event already exists · a prior `auto_intro_rejected`
event for `Agent`.

`ALLOWED_PROPERTY_TYPES` for blind intro is narrow: **condominium, single family home,
townhome/townhouse only**. Sellers additionally need a positive `price` and a
`property_type`. Phone verification is required unless the lead came from
`gvg_capital`, `realty_com`, or `property_leads` (`PARTNER_MARKETING_SOURCES`) or was a
US News agent request.

Four blind flows with different rule sets: `agent`, `agent_consent`, `appt_setting`,
`call_me_maybe`.

**Normal-intro blockers**: active `AutoIntroException` · area not assigned / city
exception · `hl_home_loans` (mortgage leads never auto-intro) · otherside ·
gender preference · language requirement · prospecting · probate · high touch ·
invalid client name/email · `@homelight.com` email · **property already listed**.

> ⚠️ `AutoIntroException` / `BlindIntroException` (+ their `_def` tables) are **ops-editable
> DB rows**, not code. "Why didn't this lead auto-intro" is usually answered there.

### 2. Broadcast claiming — Elite agents get first look

`Referrals::AutoIntro::BroadcastClaimStateMachine` — state in **Redis** (`broadcast_claim_state_machine:<lead_id>`,
TTL 12h) with a Postgres `state_machines` backup row (`name = "broadcast_claim"`).

```
initiated → elite → rebroadcast_elite → standard → rebroadcast_standard
                                              → completed | exhausted
```

- Providers are pre-bucketed into `elite_agent_ids`, `standard_agent_ids`, `investor_ids`.
  **The elite cohort is broadcast to first and re-broadcast to before standard agents ever
  see the lead.**
- **`available_slots` defaults to 2** — a lead is claimable by two agents.
- A claim taken while another claim is inside its exclusivity window is recorded as
  **`soft_locked`** and does not consume a slot until it's re-broadcast and reclaimed.
- The exclusivity window is `broadcast_intro_settings["claim_exclusivity_minutes"]`
  (SalesSetting) and is measured from **the end of the claiming agent's call log**
  (`SalesCommsService::CallLog.end_date`), not from the claim timestamp. An in-progress
  claim call keeps the lock open indefinitely.
- `completed?` = zero slots left. `exhausted` = ran out of providers.

`RoundRobinPenalty` (`round_robin_key` + `user_id` → `penalty_count`) is the separate
internal-staff distribution penalty box.

### 3. Staggered / dual-path auto-intro

`AgentStaggeredIntro` (+ `InvestorStaggeredIntro`, `EmptyStaggeredIntro`) introduces
agents one at a time on a delay. Each tick re-evaluates: business hours
(`Business.next_callable_datetime` in the lead's area timezone) → has the previously
introduced agent progressed the stage? → delay counter exhausted? → is a call in progress?
Config: `staggered_intro_settings`, `dual_path_intro_configuration`,
`dual_path_missing_provider_intro` (all SalesSetting). The previously-introduced agent is
tracked in a 24h Rails cache key, **not the database**.

`Lead::SELLER_DEFAULT_AUTO_INTRO_COUNT = 3`, `BUYER_DEFAULT_AUTO_INTRO_COUNT = 2`.

### 4. Warm transfer

`WarmTransferService` (50 files) — live phone hand-off. `ReferralPreference.accepts_warm_transfers`
/ `warm_transfer_opt_out` gate eligibility; `lead_warm_transfers` records the attempt;
AgentLead lands in stage `warm_transfer_match`. Notifications go out over SMS + push +
scheduled reminders. A Pipecat voice-AI leg exists (`voice_ai/start`, `voice_ai/transfer`).
`live_activity_warm_transfer_settings` and `warm_transfer_timeout_logs` are the SalesSettings.

## Lead induction — how a lead enters the marketplace

`LeadInductionService::Orchestrator#launch_pipeline` classifies every lead into **one of
17 types** and runs a dedicated organizer. Classification is an ordered `if/elsif` chain —
first match wins:

`incomplete_lead → eave_ingested_lead → cash_close_lead → mortgage_lead →
veterans_united_quiz_lead → idx_landing_page_lead → partner_quiz_lead → quiz_lead →
partner_lead → house_values_lead → profile_lead → phone_lead → sales_lead →
agent_referral → closing_services_lead → lead_success_program → default_lead`

Pre-orchestration steps run for **every** lead: `AssignPersona`, `TrackPplEligibility`,
`TrackPplProviderPotentialMatches`, `SetCallReviewAttributes`. On the way out, unowned
leads are assigned to `Lead::SALES_Q_ID = 14_266`.

Other induction concerns: `AssignLeadChannel` (LeadChannel + `LeadChannelDefinition`, with
five ops methods — `call_first`, `call_only`, `manual_call_first`, `text_first`,
`text_only`), `AssignQueue`, `CalculateQueueWait`, `CheckDuplicate`, `CheckUltraSub`,
`CloneDoubleSidedLead` (a client who is both buying and selling gets two leads).

`LeadDispositionService` is by far the largest engine (**451 files**) — it owns every
stage transition, disposition, and intro decision across all providables.

## Provider matching artifacts

`provider_potential_matches` records **every eligibility evaluation**, not just successes:
`trigger`, `product`, provider (polymorphic), `eligible` boolean, `knockout_reasons`
(jsonb array), `knockout_values` (jsonb), `lead_values` (jsonb).

> 🔑 This is the best available "why wasn't this provider matched" audit trail, and it's
> queryable. `knockout_reasons` is a structured array — not free text.

`provider_disputes` (agent/investor disputing a referral), `agent_lead_offers`,
`agent_lead_packages`, `agent_lead_response_times`, `agent_lead_loss_notif_list`,
`listing_loss_log` round out the marketplace bookkeeping.

## Two unrelated "Elite" programs — do not conflate

### Agent Elite = geographic exclusivity

- `agents.elite_status` ∈ `active · eligible · ready · none`.
- `elite_agent_areas` (polymorphic `area`: City / State / County / ZipCode / Neighborhood)
  has a **unique index on `(area_id, area_type)`** — 🔑 **at most one Elite agent per
  area, full stop.** A second unique index on `(area_id, area_type, agent_id)`.
- Dropping out of `active` fires `disable_elite_agent!` → `EliteAgentArea.where(agent_id:)
  .update_all(agent_id: nil)` — **the area is vacated but the row survives**, ready for
  the next agent.
- `Lead#elite_agent_in_area` looks up the Elite agent by zip and logs a warning if it
  finds more than one (which the index should prevent).
- Elite agents are the first broadcast cohort (above), get `tcpa_elite` addendum v2.0, and
  `elite_agents_aiot_service_states` / `referral_elite_alvm_peer_meeting_scheduled_push_settings`
  give them distinct comms.
- Role `sales_elite_status_admin` gates who can change it.

### Loan Officer Elite = a BBYS volume tier (the LO channel program)

`EliteProgramLevel` + `LoanOfficerRewardTransaction` + `LoanOfficerReward`. Flagsmith flag
**`lo-elite-program`**.

- Tiers: `none · elite · diamond · obsidian`.
- **Thresholds are NOT in code.** `EliteProgramLevel.levels` reads the `levels` object out
  of the `lo-elite-program` Flagsmith JSON — each level has `min`, `max`, `key`, and a
  `rewards` array. Change the flag, change the program.
- Qualifying metric: **count of distinct BBYS leads owned by that LO that hit
  `ir_closed` in the calendar year** (`count_closed_transactions` joins
  `bbys_lead_stage_updates.new_stage = "ir_closed"`, excludes `FAILED_STAGES`).
- **Exclusions**: partner slug `orchard`, any **builder** partner, and any LO whose email
  contains `@orchard`.
- 🔴 **~230 hardcoded loan-officer IDs** in `EliteProgramLevel::OVERRIDE_IDS` are granted
  `elite` regardless of volume — but only **through calendar 2026** (`return nil if
  Date.current.year > 2026`) and only while their count stays under the elite tier's
  `max`. This is a launch-cohort grandfather list living in Ruby source.
- Reaching a tier mints an `invite_program_code` (32 chars) + `invite_eligibility_unlocked_at`
  and flips that tier's `LoanOfficerReward` rows from `locked` to `active`
  (statuses: `locked · active · redeemed · expired`, unique per LO+year+reward_key;
  `redeem_for_lead!` stamps `metadata.redeemed_for_lead_id`).
- 🔑 **Diamond benefit is concrete: `waive_diamond_inspection_fees!` sets
  `waive_inspection_fee = true` on the departing property of every non-closed BBYS lead
  the LO owns.** A tier change silently rewrites deal economics in flight.
- Separately, `RewardProgram` (`active`, `start_date`/`end_date`, jsonb `config`) is a
  sweepstakes layer: stage changes accrue `transaction_count` / `points` / `entries`, with
  entry rules of the form `"<N>_points" => multiplier`. History is an append-only jsonb
  array; credits are idempotent per (lead, program, stage).
- Payouts: `DisbursementService::Payment` with `payment_source = "elite_program"` and
  `category = "referral"`, paid through **Trolley**.

## Pay-per-lead (PPL) — the second revenue model

| Object | Meaning |
| --- | --- |
| `PplProviderSetting` | provider (Agent or Investor) × area (polymorphic) × `user_type` → `price`, `active`. Unique per that tuple. Links to a Stripe `CustomerSubscriptionItem`. |
| `PplReferral` | one billable intro. Status `unpaid → paid`, or `credited` (reverted pre-charge) / `refunded` (post-charge credit). Creates a **Stripe metered usage record** on commit. |
| `PplBillingInfo` (+ `_history`) | the billing cycle |
| `PplProviderAgreement` | the PPL-specific contract (`Agreement.for_ppl_agents` / `for_ppl_investors`) |

Stripe usage-record and customer-balance mutations are wrapped in
`with_lock` + before/after count comparison, because increase and decrease can race.
Gates: `ppl_enabled_agents`, `ppl_enabled_investors`, `ppl_investor_scrub_rate`
(SalesSettings).

The newer generic layer is `ProviderBillingSetting` / `ProviderReferral` /
`ProviderReferralFee` — same status enum plus `uncollectible` and `payment_processing`,
with `rev_share` scopes keyed on `billing_item_type = "SubscriptionService::Price"`.
`ProviderReferral#closed_booked!` **auto-dispositions the AgentLead to `closed_booked`
with note "Payment received via Stripe"** when payment lands — i.e. Stripe events move
CRM stages. Status changes fan out to `ComplianceService` workers, which gate provider
visibility.

## MarketplaceProgram — the enrollment layer

`marketplace_programs` (slug-unique) × `marketplace_program_states` (per-state enable) ×
`marketplace_program_agents` (per-agent enroll). Slugs: `home_loans · fully_funded ·
simple_sale · title_escrow · elite · referrals · disclosures_io · trade_in · cash_offer ·
hlss · bbys · heloc`. Agent status derives from three booleans:
`pending → declined → enrolled → disenrolled` (in that precedence order), each with its
own timestamp plus `enrolled_by_id`.

⚠️ `agent_ever_enrolled_in?` for title/escrow also checks for a legacy
`agent_enrolled_title_escrow` **UserEvent** — enrollment history predates this table.

## Open questions

- What are the actual Ragnarok tier thresholds today? They live in
  `algo_agent_stats_settings` (`benchmark`, `force_top_20_agent`, `minimum_claim_agents`)
  and the Flagsmith `lo-elite-program` JSON — both need a live read.
- Is `account_health_multiplier` a score weight or a payout weight? Not used anywhere in
  HAPI; it must be consumed inside the external matching service.
- Where does the Agent Matching Service (Ragnarok) live — which repo, who owns it? Not in
  HAPI, not in the monolith.
- `OVERRIDE_IDS` expires end of 2026. Is there a plan for those ~230 LOs, or will they
  silently drop to their true tier on 2027-01-01? **This is a RevOps landmine.**
- `BLIND_INTO_LIMIT = 2` and `available_slots = 2` are hardcoded. Have they ever been
  tuned, and should they be settings?
- Does `leads.commission_split` (30/33) ever get used for revenue forecasting where
  `AgentLead#commission_split` is the real number?
