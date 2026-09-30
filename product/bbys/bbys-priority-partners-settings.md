---
last_updated: 2026-08-28
status: current
source: homelight/hapi @ c14079c76c
scope: LRM priority queue scoring, the partner model, and runtime BBYS settings
source: HomeLight-Vault/context/bbys-priority-partners-settings.md
imported: 2026-09-29
---

# BBYS priority, partners, and runtime settings

The LRM/AM work queue scoring model, how partners are typed, and the runtime settings that
change BBYS behavior without a deploy.

Read with [[repo-hapi]], [[partners]], [[lo-lifecycle]], [[team]].

## 🔑 The LRM priority queue

`bbys_lead_priorities` — one row per BBYS lead (unique on `bbys_lead_id`), recomputed by
`RecomputeLeadPriorityWorker` / `RecomputeAllActivePrioritiesWorker`.

| Column | Meaning |
| --- | --- |
| `score` | integer, default 0 |
| `tier` | `critical` / `high` / `normal` / `low` |
| `subscores` | jsonb — per-signal contribution |
| `reasons` | jsonb array — why it scored |
| `alerts` | jsonb array |
| `manual_flags` | jsonb — operator overrides |
| `assigned_lrm_user_id` | the owning LRM/AM |
| `under_contract`, `stage`, `pinned`, `active`, `snoozed_until` | |
| `inputs_hash`, `computed_at`, `inputs_snapshot_at` | recompute bookkeeping |

Queue order: **`pinned` desc, then `score` desc** (`scope :queue`). Scopes exist for
`for_lrm`, `under_contract`, `with_alerts`, `critical_or_high`, `snoozed` / `not_snoozed`.

Gated by flags **`bbys-lead-priority-queue`** and **`bbys-lead-priorities`**.

### Tier thresholds

| Tier | Score |
| --- | --- |
| **critical** | ≥ 50 |
| **high** | ≥ 30 |
| **normal** | ≥ 15 |
| **low** | < 15 |

### 🔑 The scoring weights

**Configured at runtime in `SalesSetting.bbys_lead_priority_rules` (JSON), not in code.**
These are the *defaults* — production may differ. Check the live setting before treating
them as current.

| Signal | Contribution |
| --- | --- |
| **`under_contract`** | base **14**, +2 if stagnant 48h |
| **`stage_age`** | thresholds: `new`/`in_review`/`approved` = **2 days**, `agreement_signed` = **30 days**; contributions yellow **2** / orange **4** / red **6** |
| **`target_equity_hit`** | hit + under contract **10** · hit otherwise **5** · **miss + under contract 7** |
| **`revenue_potential`** | DR value: ≥$500k → 2, ≥$1M → 4, ≥$1.5M → 6 · low-EU-ratio: <0.20 → 3, <0.40 → 2 · **HELOC opportunity bonus 3** · fee revenue: ≥$10k → 1, ≥$25k → 3, ≥$40k → 4 |
| **`lo_quality`** | elite **3** / diamond **6** / obsidian **10** · IRUC rate ≥30% → 6, ≥15% → 3 (min **3 leads** to qualify) · Modex score ≥80 → 4, ≥50 → 2 |
| **`strategic_partner`** | **+5** for `uwm` · `crosscountry` · `fairway` · `tls` · `apm` |
| **`touchpoint_recency`** | up to **12**, scaling over **14 days** |
| **`sla_timeline_risk`** | **9** if closing within 14 days |
| **`agreement_expiration`** | **7** if expiring within 14 days, **+2** if agreement signed |
| **`partner_exclusion`** | **`orchard`** |
| **`manual_escalation`** | `force_top` **100** · `executive_relationship` **20** · `consumer_complaint_risk` **20** · `pricing_exception` **15** · `funding_risk` **15** · `expiring_contract` **10** |
| **`snooze_override`** | **−50** |
| `engagement_signals` | **disabled** (`enabled: false`) |

**Manual flag keys** (`BbysLeadPriority::MANUAL_FLAG_KEYS`): `under_contract_override` ·
`executive_relationship` · `pricing_exception` · `consumer_complaint_risk` · `funding_risk` ·
`expiring_contract` · `force_top`

> 🔑 **`force_top: 100` alone exceeds the critical threshold twice over.** Any lead flagged
> `force_top` goes to the top regardless of every other signal. Similarly
> `snooze_override: -50` is large enough to drop almost anything out of `critical`.

> 🔑 **Orchard is explicitly excluded from the priority queue** (`partner_exclusion.slugs`),
> consistent with the team no longer working Orchard ([[2026-08-28-vault-refresh-slack-findings]]).

> 🔑 **The strategic-partner list is `uwm`, `crosscountry`, `fairway`, `tls`, `apm`** — note
> this is a *different* list from the variable-pricing partners (`apm`, `envoy`,
> `arbor-financial`, `nfm-lending`, `21st-century`) in [[bbys-pricing-engine]]. Only **`apm`**
> is on both. There is no single "strategic partner" definition.

> ⚠️ `lo_quality` uses **Modex score** — so [[modex-scrapers]] data feeds LRM queue ordering.
> If Modex enrichment is stale, queue order silently degrades.

### The signal classes

`under_contract` · `stage_age` · `target_equity_hit` · `revenue_potential` · `lo_quality` ·
`strategic_partner` · `touchpoint_recency` · `sla_timeline_risk` · `agreement_expiration` ·
`partner_exclusion` · `manual_escalation` · `snooze_override` · `engagement_signals`

Plus `LastMeaningfulTouchAt` (the recency helper), `Engine`, `Result`, `SignalResult`,
`Signals::Base`. `enabled_signals` in the setting controls which run —
**`engagement_signals` is in the registry but not in the enabled list.**

## The partner model

`partners` table:

| Column | Notes |
| --- | --- |
| `slug` | the canonical key used by pricing, priority, and intake URLs |
| `name`, `poc_name`, `active` | |
| **`partner_types`** | array — `builder` / `bank` / `lender` |
| **`builder`** (boolean), `builder_label` | ⚠️ **legacy**, see below |
| `token`, `secret` (+ `_temp` variants) | partner API auth, with rotation slots |
| `webhook_url`, `valid_domains`, `url_decode` | |
| `white_label_style` (jsonb) | logo + colors for branded intake |
| `source_page_type`, `source_form`, `sub_source`, `marketing_source` | attribution |
| `user_id` | |

```ruby
PARTNER_TYPE_BUILDER = "builder"
PARTNER_TYPE_BANK    = "bank"
PARTNER_TYPE_LENDER  = "lender"
```

Query scope: `with_partner_type` uses `partner_types @> ARRAY[?]::varchar[]`.
`Partner has_many :bbys_leads, foreign_key: "source_partner_id"` and `has_many :lenders`.
Admins attach via `admin_partners`.

> 🔴 **The `builder` boolean and the `partner_types` array both exist.** This is exactly the
> problem Joel raised in Slack: **63 guards key off `lead.builder?`** rather than a journey
> predicate, which is why bank and Orchard leads need workarounds and why the stopgap under
> discussion is *"set `builder = true` on SWBC and bank partner records."*
> The array is the modern model; the boolean is what the code actually branches on.
> See [[2026-08-28-vault-refresh-slack-findings]].

Named bank partners (e.g. **SWBC**) must be **created manually** in sales-app Partners with
logo and colors — they are not auto-provisioned. `white_label_style` is where that branding
lives.

## 🔑 State-level program availability — `MarketplaceProgram`

**This is the master on/off switch for every HomeLight program, per state.**

```ruby
MarketplaceProgram.enabled_for_state_id?("bbys",  state_id)
MarketplaceProgram.enabled_for_state_id?("heloc", state_id)
```

**Program slugs:** `fully_funded` · `simple_sale` · `home_loans` · `title_escrow` · `elite` ·
`referrals` · `disclosures_io` · `trade_in` · `cash_offer` · `hlss` · **`bbys`** · **`heloc`**

`HL_HOMES_PROGRAMS = [trade_in, cash_offer, hlss]` — with the comment
*"HL HOMES previously called CASH CLOSE"*. That explains the `CCBBYS` (Cash Close BBYS)
namespace in [[repo-sales-app]].

Structure: `marketplace_programs` (name, slug) ↔ `marketplace_program_states`
(`enabled`, **default `false`**) ↔ `states`. Also `marketplace_program_agents` with an
`enrolled` boolean for agent-level enrollment (`agent_enrolled_in?`,
`agent_ever_enrolled_in?`).

> 🔑 **"Where do we operate?" is a database question, not a code question** — and states are
> **opt-in** (`enabled` defaults to false). There is no hardcoded state list for BBYS or
> HELOC anywhere. This is also the mechanism behind "HELOC is unavailable in Hawaii."
>
> The denial message is fixed text: *"HomeLight's HELOC program is currently not available in
> this state."*

## Runtime settings that change BBYS behavior

`SalesSetting` holds **147 settings**. The BBYS-relevant ones:

| Setting | Default | What it does |
| --- | --- | --- |
| **`bbys_lead_priority_rules`** | (see above) | the entire priority scoring model |
| **`bbys_homelight_lsm_account_executive_names`** | 9 names, below | the AE roster |
| `bbys_builder_client_tasks` | all `true` | routes three builder client tasks to homes-fe: `property_photos_link_in_homes_fe`, `equity_boost_app_link_in_homes_fe`, `ir_contract_upload_app_link_in_homes_fe` |
| `bbys_quiz_slug_selection_settings` | `{enabled: false, quiz_slugs: {}}` | per-partner quiz slug routing — **disabled by default** |
| `simple_client_approval_flow` | `{enabled: false, eligible_partners: ["tls","fairway"], max_leads_count: 15}` | **disabled**; limited to TLS and Fairway, capped at 15 leads |
| `orchard_comms_cutover_overrides` | `{lead_ids: []}` | per-lead Orchard comms overrides |

> `bbys_quiz_slug_selection_settings` being disabled by default is relevant to Joel's ask for
> **dynamic "submit a client" routing in the LO portal by partner slug** — the mechanism
> exists but is off.

### 🔑 The AE roster in code

`BBYS_HOMELIGHT_LSM_ACCOUNT_EXECUTIVE_NAMES_DEFAULT`:

**Kim Tanner · John Labrada · Brian Banes · Kara Kleingarn · Marisa Drake · Richie Helali ·
Tejas Narkhede · Tierney Izar · TJ Sims**

These nine map exactly onto the Sep 1 reorg roster
([[2026-08-28-vault-refresh-slack-findings]]) and supply the full surnames:

| Sep 1 role | Names |
| --- | --- |
| Wholesale/Broker Enterprise Rep | **John Labrada** |
| Wholesale/Broker AEs (ex-LSMs) | **Tejas Narkhede · Tierney Izar · Kara Kleingarn · Brian Banes** |
| Retail Enterprise Rep | **Kim Tanner** |
| Retail AEs | **Marisa Drake · TJ Sims · Richie Helali** |

> ⚠️ **This list is a hardcoded default in `sales_setting.rb`** (overridable at runtime).
> The setting is still named `..._lsm_...` even though the title is becoming **AE** on Sep 1.
> **Someone has to update this value when the roster changes** — it is not derived from user
> records or HubSpot. Add it to the reorg checklist.

## Open questions

- What are the **live** values of `bbys_lead_priority_rules` in production vs these defaults?
- Should the strategic-partner list and the variable-pricing partner list be reconciled?
  Only `apm` is on both, and both are hand-maintained.
- Is `engagement_signals` disabled deliberately, or unfinished?
- Which states actually have `bbys` and `heloc` enabled in `marketplace_program_states`?
  That table is the authoritative service-area map and is not mirrored anywhere in the vault.
- ~~Does `bbys_homelight_lsm_account_executive_names` get read anywhere that would break if
  a name is stale (routing, filters, reporting)?~~ **Answered 2026-08-29:** exactly one
  consumer, `hapi/services/quiz_service/app/classes/quiz_service/homes/uwm_account_executive_validator.rb:24`
  (`Array(SalesSetting.bbys_homelight_lsm_account_executive_names)`), which validates the
  UWM AE name a borrower enters on the quiz path. Lands on the **wholesale** side of the
  Sep-1 Wholesale/Retail split. Added to [[2026-09-01-reorg-system-checklist]] row B4.
