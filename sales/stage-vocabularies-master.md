---
last_updated: 2026-08-29
status: current
source: homelight/hapi @ c14079c76c · homelight/homelight @ 1a8ca3eb6f · homelight/business-intelligence @ b42f4e4711 · homelight/Data-Bridge @ 4cbb4786 · context/hubspot.md · context/lending-mechanics.md · context/bbys-deal-channel-vocabulary.md · live HubSpot portal 4744876 (2026-08-29, read-only, `hubspot_get_deal_pipeline_analytics`)
scope: Master reconciliation of every stage/status vocabulary at HomeLight — values, owner, storage column, cross-mappings, and the columns that have two writers
source: HomeLight-Vault/context/stage-vocabularies-master.md
imported: 2026-09-29
---

# Stage vocabularies — the master reconciliation

**Why this file exists.** Two vault docs independently announced the *same ordinal* — "a
**sixth** stage vocabulary" ([[bbys-equity-boost]]:126 and [[repo-eave-hlhl]]:159, about
different finds) — while four others gave competing totals: "at least eight"
([[repo-hapi-platform]]), "three" ([[repo-business-intelligence]]), "all three"
([[repo-bi-periscope]]), "two more for the pile" ([[repo-homelight-monolith]]), plus "a
fifth numeric scale" ([[heloc-product]]). Ordinal counting failed because nobody was
counting the same set.
**This file replaces the counter with a table.** When a doc wants to say "another stage
vocabulary", it links here instead.

> 🔑 **Counting rule used here** (state it or the number is meaningless): a *vocabulary* is a
> named, enumerated set of stage/status values that is (a) written to a persisted column or
> API field, and (b) defined independently of every other set — a differing copy counts
> separately. By that rule: **39**. Under a narrower rule that collapses divergent duplicates
> and excludes tier/priority scales, ~26. The critic's `≥20` was a floor, not a count.

**The number is not the point.** The point is §4 (columns with two writers) and §6
(safe joins).

---

## §1 Master index

| # | Vocabulary | Owner system | Count | Stored where |
| ---: | --- | --- | ---: | --- |
| 1 | `LeadStage` | HAPI Rails | **83** | `leads.stage`, `leads.furthest_stage` |
| 2 | `AgentLead::STAGE_*` | HAPI | **34** | `agent_leads.stage`, `provider_leads.stage` |
| 3 | `InvestorLead::STAGE_*` | HAPI | 16 | `provider_leads.stage` |
| 4 | `TradeInLead::STAGE_*` | HAPI | 23 | `provider_leads.stage` |
| 5 | `CashOfferLead::STAGE_*` | HAPI | 20 (`LEAD_STAGES` + `PROVIDER_STAGES`) | `provider_leads.stage` |
| 6 | `HlSimpleSaleLead::STAGE_*` | HAPI | 16 | `provider_leads.stage` |
| 7 | `LenderLead::STAGE_*` | HAPI | **31** | `provider_leads.stage` |
| 8 | `EscrowOfficeLead::STAGE_*` | HAPI | 11 | `provider_leads.stage` |
| 9 | `BbysLead::STAGES` | HAPI | 10 | `provider_leads.stage` |
| 10 | `Order::STAGE_*` | HAPI | 5 | `orders.stage` |
| 11 | `HelocLead::STAGES` | HAPI | 8 | `heloc_leads.stage` |
| 12 | `LeadStage` (monolith copy) | `homelight/homelight` | **60** | *same* `leads.stage` |
| 13 | `AgentLead` (monolith copy) | monolith | **32** | *same* `agent_leads.stage` |
| 14 | `InvestorLead` (monolith copy) | monolith | 15 | *same* `provider_leads.stage` |
| 15 | `LenderLead` (monolith copy) | monolith | 25 | *same* `provider_leads.stage` |
| 16 | `equity_boost_stage` enum | HAPI | **5** | `bbys_leads.equity_boost_stage` |
| 17 | `approval_type` enum | HAPI | 4 | `bbys_leads.approval_type` |
| 18 | `HelocCardApplication::STATUSES` | HAPI `HelocCardService` | 3 | `heloc_card_applications.status` |
| 19 | `LeadDataService::MortgageLead::STAGE_*` | HAPI engine | 23 | mortgage-lead API payloads |
| 20 | BBYS Applications pipeline | HubSpot `681595694` | 10 | `deals.dealstage` (+ mirrors) |
| 21 | Lead Funnel pipeline | HubSpot `866607668` (obj `0-136`) | 4 known | `hub_leads.lead_stage_id` |
| 22 | HubSpot Help ticket pipeline | HubSpot | 2 known | ticket `hs_pipeline_stage` |
| 23 | `referral_stage_index` | dbt seed | 13 stages / idx 0–12 | `fur_stage_index` family |
| 24 | `homes_bbys_stages_d` | dbt seed | 22 keys → 8 indices | `new_stage_index` |
| 25 | `homes_stages_d` | Redshift `bi.` source table | 27 stages / idx 1.00–15.00 | `cur_stage_index`, `fur_stage_index` |
| 26 | `trade_in_stages_d` | dbt seed | 16 | `sort` 0–13 |
| 27 | `lender_lead_stages_d` | dbt seed | 21 | `sort` 1–11 |
| 28 | `agent_stages_d` | dbt seed | 28 | 🔴 **merge-conflicted, see §4.4** |
| 29 | `investor_stages_d` | dbt seed | 19 | `sort` 0–100 |
| 30 | `cash_offer_stages_d` | dbt seed | 16 | `sort` 0–15 |
| 31 | `agent_and_investor_stages_d` | dbt seed | 47 rows / `funnel_index` 0–13 | funnel rollups |
| 32 | Eave `HLStage` | `eave-go` (frozen 2023-12-11) | 13 | pushed to HAPI mortgage lead |
| 33 | `UnderwritingSummaryLoanStatus` | `eave-go` | 19 (→ 6 `LoanMilestone`) | Eave loan file |
| 34 | `WireStatus` | `eave-go` | 3 | Eave wire record |
| 35 | `GRAZIELLE_STATUS_TO_STAGE` | monolith | 5 | MLS-status → stage map |
| 36 | `StageKey` label bucketing | Data Bridge `admin/src/lib/stageOrder.ts` | 10 | derived, not stored |
| 37 | Deal priority `Tier` | Data Bridge `priorityScore.ts` | 4 (`P0`–`P3`) | `hub_deals` priority |
| 38 | LO-lead priority | Data Bridge `queueTriggers.ts` | 6 (`P0`–`P5`) | `lo_lead_priority` |
| 39 | Slack channel stage tokens | Sales App channel renamer | 9 | channel **name** only |

---

## §2 The HAPI sets in full

### 2.1 `LeadStage` — 83 values (`hapi/app/classes/lead_stage.rb`)

Each constant is a **two-element array** `["machine_name", "Human Readable"]`. The `[0]` form
goes in the DB; the `[1]` form goes in dropdowns. Confusing them is the single most common
stage bug in this codebase.

Only **72** of the 83 appear in the display hash `STAGES` (line 115); **83** appear in
`STAGE_PROGRESS` (line 374, the numeric weight map); 4 are "sub stages" (line 190).
19 further group-arrays (`ACTIVE_STAGES` 25, `NOT_TRANSACTING_STAGES` 15,
`TALKED_TO_CLIENT_STAGES` 18, `FILE_CLOSED_STAGES` 6, …) slice the same 83.

> ⚠️ `LeadStage.display_stage` returns the literal string **`"Unknown"`** for any value not
> in `STAGES` — so the 11 values in `STAGE_PROGRESS` but not `STAGES`
> (`awaiting_agent`, `interested_via_text`, `introduced_to_agent`, `in_pre_escrow`,
> `offer_accepted`, `on_hold`, `pending_failed`, `pending_order`, `processing`, `requested`,
> `quiz_match`) render as "Unknown" in the UI while still scoring in the funnel.

### 2.2 The providables — one column, ten vocabularies

`provider_leads.stage` is a **polymorphic** column. Which vocabulary is valid depends
entirely on `provider_leads.providable_type`.

| Providable | n | Values (abbreviated) |
| --- | ---: | --- |
| `AgentLead` | 34 | `quiz_match` `client_added_match` `auto_intro_match` `warm_transfer_match` `intro_request` `investor_qualified` `pre_salesforce` `awaiting_response` `referral_declined` `reassigned` `call_in_progress` `no_response` `client_cancelled` `removed` + the shared referral ladder + `in_contract` `offer_accepted` `making_offer` |
| `InvestorLead` | 16 | as AgentLead minus the match/ops values; ⚠️ three carry `TODO: change this stage` comments (`coming_soon` really = offer sent, `in_escrow` really = offer accepted) |
| `TradeInLead` | 23 | `pitched` `application_started` `application_complete` `inspection_complete` `conditionally_approved` `ip_in_escrow` `ip_closed` `client_listed` `client_in_escrow` `client_closed` `in_escrow_purchase/_sell` `closed_purchase/_sell` `listed` `nurture` |
| `CashOfferLead` | 20 | **two parallel ladders**: `LEAD_STAGES` (`new` `connected` `pitched` `agreement_signed` `in_escrow_*` `closed_*` `nurture` `failed`) and `PROVIDER_STAGES` (`property_pending` → `property_submitted` → `property_approved` → `offer_made` → `offer_accepted`, plus `*_rejected` / `*_cancelled`) |
| `HlSimpleSaleLead` | 16 | TradeIn minus `conditionally_approved`, plus `approved` `agreement_signed` |
| `LenderLead` | 31 | `new_no_app` `pre_approval_started` `account_created` `pre_approved` `application_created` `credit_check_passed` `signed_borrower_auth` `in_review` `conditionally_approved` `in_contract` `clear_to_close` `funded` `underwritten` `1003_taken` `incomplete_application` `offer_declined` `did_not_qualify` `lost` `qualified` … |
| `EscrowOfficeLead` | 11 | `awaiting_agent` `pending_order` `processing` `on_hold` `in_pre_escrow` `in_escrow` `closed` `closed_paid` `duplicate` `new` `failed` |
| `BbysLead` | 10 | `new` `in_review` `approved` `agreement_signed` `ir_contract` `clear_to_fund` `ir_closed` `dr_closed` `nurture` `failed` |
| `Order` | 5 | `new` `active` `complete` `nurture` `failed` (on `orders.stage`) |
| `HelocLead` | 8 | `new` `in_review` `needs_human_review` `preliminary_prequalified` `prequalified` `approved` `denied` `withdrawn` (on `heloc_leads.stage`; `STAGE_VALUES` gives a contiguous 0–7 scale) |

> 🔑 **`bbys_leads` has no `stage` column.** BBYS stage lives on `provider_leads.stage`
> where `providable_type = 'BbysLead'`. `bbys_leads` carries only `equity_boost_stage`.
> Anyone querying Crunchy for "the BBYS stage" must join `provider_leads`.

### 2.3 🔴 Collision matrix — the same string in many vocabularies

Computed across the 11 HAPI sets. Reading `provider_leads.stage` **without** filtering
`providable_type` mixes these:

| Value | In how many vocabularies | Which |
| --- | ---: | --- |
| `failed` | **10** | every one |
| `new` | 9 | Bbys, CashOffer, EscrowOffice, Heloc, HlSimpleSale, LeadStage, Lender, Order, TradeIn |
| `closed_paid` | 7 | Agent, CashOffer, EscrowOffice, HlSimpleSale, Investor, LeadStage, TradeIn |
| `connected` | 7 | Agent, CashOffer, HlSimpleSale, Investor, LeadStage, Lender, TradeIn |
| `nurture` | 6 | Bbys, CashOffer, HlSimpleSale, Lender, Order, TradeIn |
| `in_escrow` / `closed` | 5 | Agent, EscrowOffice, Investor, LeadStage, Lender |
| **`approved`** | 4 | **Bbys** (valuation approved) · **Heloc** (HELOC approved) · HlSimpleSale · TradeIn |
| **`in_review`** | 4 | **Bbys** (ops underwriting) · **Heloc** · LeadStage (`"In UW Review"`) · Lender |
| `agreement_signed` | 4 | Bbys, CashOffer, HlSimpleSale, TradeIn |
| `pitched` | 4 | CashOffer, HlSimpleSale, Lender, TradeIn |
| `conditionally_approved` | 3 | LeadStage, Lender, TradeIn |

> 🔴 **`stage = 'approved'` is not a question with one answer.** A BBYS approval, a HELOC
> approval and a trade-in approval are the same 8 characters in the same column.

---

## §3 The non-HAPI sets

### 3.1 HubSpot — BBYS Applications pipeline `681595694`

| Order | Label | Stage ID | HAPI equivalent | Notes |
| ---: | --- | --- | --- | --- |
| 1 | New | `998755441` | `new` | operator says "Application / Apps" |
| 2 | In Review | `998755442` | `in_review` | |
| 3 | Approved | `998755443` | `approved` | |
| 4 | Agreement Signed | `998755444` | `agreement_signed` | |
| 5 | IR In Escrow | `998755445` | **`ir_contract`** | ⚠️ names differ; operator says IRUC / UC / Contract |
| 6 | Clear to Fund | `998755446` | `clear_to_fund` | ⚠️ operator says **"Clear to Close"**, which is a *different* `LenderLead` stage |
| 7 | IR Closed | `998755447` | `ir_closed` | |
| 8 | DR Closed | `998815822` | `dr_closed` | closed-won |
| — | Nurture | `998815823` | `nurture` | terminal for LSM/LRM, **revive target** for `LRM_ASSISTANT` |
| — | Failed | `998815824` | `failed` | closed-lost |

**IRUC+** = `998755445/446/447/998815822`. Also live: BBYS Opportunities pipeline
`725454515` (Data Bridge `BBYS_OPPORTUNITIES_PIPELINE_ID`).

### 3.1b HubSpot — BBYS Opportunities pipeline `725454515` (found 2026-08-29)

Pulled live via `hubspot_get_deal_pipeline_analytics` — the first time this vault has
enumerated it. Full detail and 2026-08-29 deal counts in [[hubspot]] § BBYS Opportunities
Pipeline.

| Order | Label | Stage ID | HAPI equivalent | Notes |
| ---: | --- | --- | --- | --- |
| 1 | New | `1057558065` | `new` | 13,122 deals sitting here — see 🔴 below |
| 2 | Reached Out | `1082564609` | — | no clean HAPI equivalent (pre-`BbysLead`) |
| 3 | Connected | `1057558066` | `connected`-ish | not a `BbysLead` value; closest is `AgentLead`/`Lender`'s `connected` |
| 4 | Follow Up | `1057558067` | — | no clean HAPI equivalent |
| 5 | Closed Won - Application Submitted | `1057558070` | — | closed-won; feeds into `BbysLead.new` on the Applications pipeline |
| — | Closed Lost | `1057558071` | `failed`-ish | closed-lost |

⚠️ Unlike BBYS Applications (§6 "`BbysLead` ↔ HubSpot is 1:1 by position"), **this pipeline
has no faithful HAPI stage mapping** — it's a pre-application opportunity funnel with its own
vocabulary, not a HAPI mirror. Don't attempt a positional translation here.

⚠️ Stage IDs `1057558068` and `1057558069` do not exist in the live sequence between Follow Up
(`…067`) and Closed Won (`…070`) — inferred archived/deleted stages, not confirmed.

🔴 **13,122 of the deals in this pipeline sit in "New"; only 110 ever reach Closed Won** (2026-08-29
snapshot). Either a low-intent import dumping ground or a stage that means "never contacted" —
treat any conversion-rate claim built on these stage counts as suspect until someone confirms
which.

### 3.2 HubSpot — Lead Funnel pipeline `866607668` (object `0-136`)

`1297783695` New · `1297783696` Attempting · `1297783698` Qualified · `1297783699` Disqualified.

> ⚠️ **`1297783697` is missing from the hardcoded map** (`Data-Bridge/src/triggers/queueTriggers.ts:207`).
> Either a deleted stage or an omission; the code falls back to a live pipelines API fetch
> (5-min cache), so a real `…697` stage would render as its raw ID in some paths.

**Attempted 2026-08-29, still unresolved — instrument limitation, not a dead end.** Pipeline
`866607668` is a **Leads object (`0-136`) pipeline, not a Deals pipeline.** The live-portal
tool available this pass (`hubspot_get_deal_pipeline_analytics`) calls
`crm/v3/pipelines/deals/{id}` — pointed at `866607668` it 404s with an HTML error body, not
JSON (confirmed by direct test). The four known stages (`…695/696/698/699`) all sit between
New and Disqualified with `…697` alphanumerically/numerically in the gap after Attempting —
consistent with a deleted "something between Attempting and Qualified" stage, same pattern as
the `1057558068`/`069` gap found in BBYS Opportunities today (§3.1b), but still not confirmed.
Resolving this needs a `crm/v3/pipelines/leads/866607668` call (no tool in this vault's
current HubSpot MCP roster exposes Leads-object pipelines) or a live Lead record carrying the
stage.

### 3.3 BI / dbt numeric scales

`referral_stage_index` (idx 0–12), `homes_bbys_stages_d` (`new_stage_index`
1 / 4 / 7 / 8 / 9 / 9.01 / 9.1 / 15) and the five `*_stages_d` seeds are transcribed in
[[bi-metrics-definitions]]. Two additions that doc does not have:

- **`homes_stages_d` is a ninth scale, and it is *not* `homes_bbys_stages_d`.** 27 stages,
  index `1.00`–`15.00`, loaded as a Redshift **source** (`bi.homes_stages_d`,
  `models/sources.yml:119`), not a dbt seed — so it is hand-maintained outside the repo.
  It disagrees with `homes_bbys_stages_d` on shared keys: `ip_closed` 9.10 vs 9.1,
  `client_listed` **9.20 vs 9.1**, `in_escrow_purchase` **10.00 vs 9.1**,
  `closed_purchase` **11.00 vs 15**. Both are joined on `furthest_stage`.
- **`fur_stage_index` is five different columns**, all built from the *same*
  `referral_stage_index` seed: `d2c_fur_stage_index`, `buyer_`, `seller_`, `mortgage_`,
  `bbys_`. See §4.3 for why the mortgage one is broken.

### 3.4 Lending (Eave / HLHL)

- **`HLStage`, 13 values** pushed back to HAPI: `pre_approval_started` ·
  `pre_approval_completed` · `pre_approval_failed` · `credit_application_started` ·
  `credit_check_passed` · `borrowers_auth_signed` · `in_borrower_uw` ·
  `incomplete_application` · `conditionally_approved` · `contract_received` ·
  `mortgage_approved` · `adverse_action` · `funded`. Overlaps `LenderLead` on 4 values only
  (`pre_approval_started`, `credit_check_passed`, `conditionally_approved`, `funded`,
  `incomplete_application`) — the other 8 have **no HAPI equivalent**. See [[repo-eave-hlhl]].
- **`UnderwritingSummaryLoanStatus`, 19 values** → 6 `LoanMilestone`s. See [[lending-mechanics]].
- **`WireStatus`, 3**: `StatusWireRequested` → `StatusWireApproved` → `StatusFunded`.

### 3.5 Slack channel stage tokens (Sales App renamer)

`-new-` → `-rev-` → `-appr-` → `-as-` → `-iruc-` → `-ctc-` → `-irx-` → `-drx-`, plus
`-term-` reachable from any stage. **The rename event is the most reliable stage-transition
signal in the company** ([[bbys-deal-channel-vocabulary]]:104) — but the token is only in the
channel *name*; the bot message body says `Stage UPDATED to Ir Closed`, i.e. a third casing.

### 3.6 Two `P0` scales

| Scale | Values | Where |
| --- | --- | --- |
| Deal priority `Tier` | `P0` `P1` `P2` `P3` | `Data-Bridge/src/utils/priorityScore.ts:61` |
| LO-lead priority | `P0` … **`P5`** | `Data-Bridge/src/triggers/queueTriggers.ts:211` |

> ⚠️ Same tokens, different cardinality, different objects. "P3" means *lowest* on deals and
> *middle* on LO leads.

---

## §4 🔴 Same column, two writers

### 4.1 `leads.stage` — HAPI (83) vs monolith (60)

Both apps run against the **same Postgres** ([[monolith-vs-hapi]]). Verified by
constant-extraction on both files:

| | HAPI `c14079c76c` | monolith `1a8ca3eb6f` |
| --- | ---: | ---: |
| `LeadStage` constants | **83** | **60** |
| `STAGES` display hash | 72 | 59 |
| `STAGE_PROGRESS` weight map | 83 | 69 |

**The monolith's set is a strict subset** — zero monolith-only values. The 23 HAPI-only
values the monolith cannot render (they fall through to `"Unknown"` or `nil` in monolith
code paths):

`active_ultra_client` · `awaiting_agent` · `awaiting_lender` · `blind_intro` ·
`human_voicemail_blind` · `in_pre_escrow` · `interested_via_text` · `introduced_to_agent` ·
`leave_voicemail` · `no_action` · `normal_intro` · `offer_accepted` · `offer_submitted` ·
`on_hold` · `pending_failed` · `pending_order` · `previous_lead_created` · `processing` ·
`quiz_match` · `requested` · `scheduled_call` · `sent_to_agent_ae` · `unsubscribe`

> 🔴 **Correction to [[2026-08-29-critic-technical]] §1.4**, which reported 78 / 57 and a
> 21-value delta. Counting every `CONST = ["value", "Label"]` / `%w[...]` definition in each
> file gives **83 / 60 / 23**. (78 is the HAPI `STAGES`+`SUB_STAGES` display count minus
> dupes; 57 is close to the monolith's 59-entry `STAGES`. Both prior figures were
> display-hash counts, not vocabulary counts.) Trust 83/60 for "what can be in the column".

### 4.2 🔴 `STAGE_PROGRESS` weights *disagree* — nobody has flagged this

Same stage, same column, two different funnel weights depending on which app computed it.
This drives `furthest_stage` promotion and every "furthest stage" metric downstream.

| Stage | HAPI weight | Monolith weight |
| --- | ---: | ---: |
| `claimed` | 10 | **20** |
| `client_connected` | 60 | 59 |
| `client_left_vm` | 50 | 49 |
| `duplicate` | 1 | **0** |
| `bad_number` · `fake` · `not_interested` · `not_transacting` · `not_transacting_agent` · `not_transacting_rental` · `unqualified` | 11 | 10 |

Plus 14 stages weighted in HAPI and **absent** from the monolith's map (`spam`,
`tax_valuation`, `refinance_valuation`, `curious_valuation`, `looking_for_specific_agent`,
`blind_intro`, `normal_intro`, `human_voicemail_blind`, `leave_voicemail`,
`interested_via_text`, `scheduled_call`, `unsubscribe`, `no_action`, `offer_accepted`) —
these evaluate to `nil` in monolith progression comparisons.

`LeadStage.get_stages_by_progression_weights(min, max)` is the shared reader. A threshold
tuned against one app's weights returns a different stage set in the other.

### 4.3 🔴 `mortgage_fur_stage_index` joins mortgage stages against the *referral* seed

`business-intelligence/models/d2c-lender-referrals/staging/_stg_order_mortgage_lead.sql:20`:

```sql
left join {{ ref('referral_stage_index') }} mort_index
       on mortgage.furthest_stage = mort_index.stage
```

`referral_stage_index` holds 13 **agent-referral** stages. `LenderLead` has 31 values, of
which exactly **5** appear in that seed (`introduced`, `connected`, `in_escrow`, `closed`,
`failed`). **26 of 31 mortgage stages produce a NULL index** — including `funded`, the
actual mortgage close. The very next line orders by `mort_index.index desc`, so **funded
loans sort last**. The same seed is joined to buyer/seller `furthest_stage` in
`_stg_order_lead_details.sql:35`, where 70 of `LeadStage`'s 83 values are unmatched.

The commented-out line right below it (`-- left join homes_stages_d`) suggests someone
started fixing this and stopped. **(inferred)**

### 4.4 🔴 `agent_stages_d.csv` has an unresolved merge conflict committed to `main`

`seeds/lead_stages_normalization/agent_stages_d.csv` lines 2 / 31 / 60 contain
`<<<<<<< HEAD`, `=======`, `>>>>>>> df967a5…`. Both halves are the full 28-row agent table;
they differ only in `updated_at` (7/13/2022 vs 7/6/2022) and row order. Present in
`git show HEAD` at `b42f4e4711` (2026-08-21). Any `dbt seed` of this file loads three junk
rows and 56 stage rows instead of 28.

### 4.5 Other divergent duplicates

| Constant | HAPI | monolith | monolith is missing |
| --- | ---: | ---: | --- |
| `AgentLead::STAGE_*` | 34 | 32 | `in_contract`, `offer_accepted` |
| `InvestorLead::STAGE_*` | 16 | 15 | `listing` |
| `LenderLead::STAGE_*` | 31 | 25 | `1003_taken`, `connected`, `duplicate`, `new`, `nurture`, `pitched` |

Same pattern every time: **HAPI is a strict superset; the monolith is frozen at an older
snapshot.** Practical read: the monolith can *write* only old values, but must *read* rows
containing new ones.

### 4.6 🔴 `bbys_leads.equity_boost_stage` — the DB does not contain what the vault says

`hapi/app/models/bbys_lead.rb:283` is a Rails `enum` with **string values**, so the column
stores the *value*, not the key:

| Ruby key (what code sees) | **Column value (what SQL/BI/HubSpot sees)** |
| --- | --- |
| `not_apply` | `N/A` |
| `waiting_for_application` | `Waiting for application` |
| `in_review` | `In-review ` ← **trailing space** |
| `approved` | `Approved` |
| `denied` | `Denied` |

> 🔴 Two corrections to [[bbys-equity-boost]]: the enum has **5** values, not 4
> (`waiting_for_application` is missing from the doc), and the column contains
> title-cased labels, not snake_case. `WHERE equity_boost_stage = 'in_review'` returns
> **zero rows**; the correct predicate is `= 'In-review '`, trailing space included.
> `bbys_lead.rb:1041` (`equity_boost_stage.in?(%w[in_review approved denied])`) works only
> because Rails' reader returns the key.

### 4.7 `SF_TO_HL_STAGES` has one inverted row

`homelight/app/classes/lead_stage.rb:492` maps Salesforce display label → HL machine name,
28 rows, all `CONST[1] => CONST[0]` — **except** `CLOSED_BOOKED[0] => CLOSED_BOOKED[1]`
(line 515), which is backwards. The hash is also not `.freeze`d. Legacy Salesforce import
path; low blast radius today, but it is wrong.

---

## §5 Cross-mapping — what translates to what, and where it leaks

| From → To | Where the mapping lives | Lossy? |
| --- | --- | --- |
| `BbysLead` 10 → HubSpot pipeline `681595694` | `hapi/services/provider_api_service/.../update_hubspot_deal_stage.rb`; `order_data_service/.../update_bbys_opportunity_hubspot_deal.rb` | **Label-lossy** — `ir_contract` ↔ "IR In Escrow", `clear_to_fund` ↔ "Clear to Fund"/"Clear to Close" |
| HubSpot labels → Data Bridge `StageKey` | `Data-Bridge/admin/src/lib/stageOrder.ts` | **Yes — substring matching.** Buckets by `.includes("dr close")`, `"closings"`, etc. Any new HubSpot label that contains "clos" lands in `ir_closed` by accident |
| `TradeInLead` / `HlSimpleSaleLead` → BBYS 8-index | `seeds/homes/homes_bbys_stages_d.csv` | **Yes** — `conditionally_approved` and `approved` both → 7; the index cannot separate conditional from full approval |
| `furthest_stage` → `fur_stage_index` | `seeds/referral_stage_index.csv` | **Catastrophically** — see §4.3. Also idx `1` is blank and idx `7` maps to two stages |
| `furthest_stage` → `cur_stage_index` | `bi.homes_stages_d` (hand-loaded source) | **Yes** — disagrees with `homes_bbys_stages_d` on 4 shared keys (§3.3) |
| Eave `HLStage` → `LenderLead` | `POST /lead-data-service/home-loans/mortgages` | **Yes** — 8 of 13 Eave stages have no HAPI stage |
| `UnderwritingSummaryLoanStatus` → `LoanMilestone` | `eave-go/underwriting/underwriting_summary.go` | By design (19→6); `canceled` maps to nothing |
| BBYS stage → Slack channel token | Sales App channel renamer | One-way; token is in the channel **name**, the bot body uses a third casing (`Ir Closed`) |
| Salesforce label → `LeadStage` | monolith `SF_TO_HL_STAGES` (28 of 60) | **Yes** — 32 stages unmapped, 1 row inverted (§4.7) |
| MLS status → `LeadStage` | monolith `GRAZIELLE_STATUS_TO_STAGE` | 5 statuses → stage *lists*; `cant_find` → `[]` |
| `LeadStage[0]` ↔ `LeadStage[1]` | `LeadStage.display_stage` / `.machine_names` | Two stage pairs share the human label **"New"** (`new` and `claimed`) and two share **"Introduced to Agent"** (`introduced`, `introduced_to_agent`) — the label→machine direction is **not injective** |

**There is no mapping at all** between: HELOC 8-stage and BBYS 10-stage; `equity_boost_stage`
and anything; the two `P0` tier scales; `homes_stages_d` and `referral_stage_index`.

---

## §6 Safe joins / never-join — for HubSpot report builders

**Never:**

1. **Never filter `provider_leads.stage` without `providable_type`.** Ten vocabularies share
   the column; `failed` is in all ten and `approved` in four (§2.3).
2. **Never join two systems on a stage *string*.** `in_review` in BBYS = ops underwriting;
   in HELOC = HELOC review; in `LeadStage` = "In UW Review". Join on IDs and dates.
3. **Never compare a `new_stage_index` threshold to a `fur_stage_index` threshold.** Different
   scales; `>= 3` means different things ([[bi-metrics-definitions]]).
4. **Never subtract stage indices** to measure funnel distance — `new_stage_index` is
   non-contiguous and fractional (9, 9.01, 9.1, 15).
5. **Never use `mortgage_fur_stage_index`** until §4.3 is fixed.
6. **Never assume a stage value seen in prod is renderable** — the monolith knows 60 of
   HAPI's 83 (§4.1).
7. **Never write `equity_boost_stage = 'in_review'` in SQL** (§4.6).

**Safe:**

- **Dates over stages.** `bbys_approved_date`, `bbys_agreement_signed_date`,
  `hl_deals_ir_contract_date`, `ir_closed_date` are single-writer milestone timestamps and
  are the right basis for funnel and cohort reporting.
- **HubSpot stage *IDs*, never labels.** `998755445` is stable; "IR In Escrow" / "IRUC" /
  "Contract" / "UC" are four names for it.
- **`BbysLead` ↔ HubSpot is 1:1 by position** for all 10 stages — the only fully faithful
  stage mapping in the company. Translate by position, not by name.
- **HELOC's `STAGE_VALUES` 0–7 is contiguous** and safe to compare/threshold *within HELOC*.
- **Channel-rename events** as the ground-truth transition log when the mirror is stale.

---

## §7 Dated corrections issued by this doc (2026-08-29)

| Doc | Was | Now |
| --- | --- | --- |
| [[bbys-equity-boost]]:126,133 | "`equity_boost_stage` is a **sixth** stage vocabulary … Six now documented" (4 values) | #16 of 39 here; **5** values; stored as labels (§4.6) |
| [[repo-eave-hlhl]]:159 | "`HLStage` … a **sixth** stage vocabulary" | #32 of 39 here |
| [[repo-hapi-platform]]:123 | "at least eight, all live"; `LeadStage` "~85" | 11 HAPI sets; `LeadStage` = **83** |
| [[repo-homelight-monolith]]:139 | "two more for the pile"; `AgentLead` "30" | 4 divergent copies; `AgentLead` = **32** |
| [[repo-business-intelligence]]:175 / [[repo-bi-periscope]]:55 | "three stage vocabularies" | **9** BI scales (§1 rows 23–31) |
| [[bi-metrics-definitions]]:18 | "four numeric stage scales" | **nine**; `homes_stages_d` and the five `fur_stage_index` variants were missing |
| [[2026-08-29-critic-technical]]:§1.4 | 78 / 57, 21-value delta | **83 / 60, 23-value delta** (§4.1) |
| This doc:§3.1 / Open questions (own prior text) | "BBYS Opportunities `725454515` — separate, undocumented stage set" | **Documented, §3.1b** — 6 stages, live IDs, 2026-08-29 deal counts |

---

## Open questions

- **Does the monolith still *write* `leads.stage`, or only read it?** §4.2's weight
  divergence only bites if both write. Settling it needs prod row-level access or a
  `provider_lead_stage_observer.rb` trace (the monolith has one at
  `app/jobs/observer/provider_lead_stage_observer.rb` — suggestive of writes).
- **Which app last wrote a given row?** There is no `stage_source` / `updated_by_app` column.
  Without one, §4.1 and §4.2 are undiagnosable from data.
- **Is `bi.homes_stages_d` still refreshed?** It is a hand-loaded Redshift source, not a seed.
  If it is stale, `cur_stage_index` is stale for every new stage value.
- **What is HubSpot stage `1297783697`?** Missing from the Lead Funnel map (§3.2). **Still
  open as of 2026-08-29** — confirmed it's a Leads-object (`0-136`) pipeline stage, which no
  tool in this vault's current HubSpot MCP roster can query (the deal-pipeline tool 404s
  against it). Needs a `crm/v3/pipelines/leads/866607668` call or a live Lead record.
- ~~**What are BBYS Opportunities pipeline `725454515`'s stages?**~~ **Resolved 2026-08-29** —
  see §3.1b. 6 stages, IDs `1057558065/1082564609/1057558066/1057558067/1057558070/1057558071`.
- **Was the `agent_stages_d.csv` conflict (§4.4) ever loaded?** If dbt seeds that file on a
  schedule, agent funnel indices may be wrong today.
- **Do HubSpot workflows write `dealstage` in parallel with HAPI?** [[hubspot-workflows]] is
  2,816 lines and unreconciled against `update_hubspot_deal_stage.rb`. If yes, `dealstage`
  is a third two-writer column.

## Related
- [[bi-metrics-definitions]] · [[bbys-stage-progression]] · [[bbys-equity-boost]] ·
  [[heloc-product]] · [[lending-mechanics]] · [[repo-eave-hlhl]]
- [[repo-hapi]] · [[repo-hapi-platform]] · [[repo-homelight-monolith]] · [[monolith-vs-hapi]]
- [[hubspot]] · [[hub-mirror-gotchas]] · [[data-bridge]] · [[bbys-deal-channel-vocabulary]]
- [[2026-08-29-critic-technical]]
