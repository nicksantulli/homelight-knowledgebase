---
last_updated: 2026-08-28
status: current
source: homelight/hapi @ c14079c76c
scope: BBYS email/comms journeys — eligibility gating, the 54 email classes, the post-funding cadence
source: HomeLight-Vault/context/bbys-comms-journeys.md
imported: 2026-09-29
---

# BBYS comms journeys

Which email a BBYS lead gets, and why. This is the system behind the "comms journey saga"
Joel and Dave worked through in August 2026.

Read with [[repo-homes-fe]], [[bbys-routing-and-ownership]],
[[2026-08-28-vault-refresh-slack-findings]].

## 🔑 `ops_channel` — the two-value spine

`BbysLeadOpsChannel` concern on `BbysLead`:

```ruby
CLIENT = "client"; PROVIDER = "provider"
validates :ops_channel, inclusion: { in: ["client", "provider"] }, allow_nil: true
```

**These are the same two values as `BBYS_CHANNELS` in `homes-fe`** ([[repo-homes-fe]]) —
front end and back end agree on the channel vocabulary.

### Derivation

```ruby
def resolve_ops_channel(source:, source_partner: nil)
  return CLIENT if ClientDirectVariant.client_direct_journey_submission?(source)
  return CLIENT if source_partner&.builder?
  PROVIDER
end
```

Two ways to become `client`: a **client-direct journey submission**, or a **builder
source partner**. Everything else is `provider`.

### `effective_ops_channel` — the fallback

```ruby
ops_channel.presence || resolve_ops_channel(source: lead&.source, source_partner: source_partner)
```

> 🔑 **`ops_channel` is nullable and re-derived on read when unset.** So a lead's channel can
> change if its `source_partner` changes — which is exactly why PR #20086 (2026-08-27) had to
> **auto-update the ops Slack channel when a lead's partner type changes**.
>
> This is also the mechanism behind Joel's ask for a **default slug**: `/bbys-intake` with no
> partner resolves to the generic variant with `ops_channel = client`, and he wants
> pre-lead→lead conversion to be unambiguous to ops rather than relying on derivation.

## The eligibility gate

```ruby
def comms_journey_eligible?
  return true if lead&.builder? || client_ops_channel?
  orchard_builder_comms_enrollee?
end
```

**Three ways in:**
1. `lead.builder?` — the legacy boolean
2. `client_ops_channel?` — the modern derivation
3. `orchard_builder_comms_enrollee?` — Orchard, post-cutoff

Logged as `[COMMS_JOURNEY]` and `[ORCHARD_COMMS]` lines with the deciding values —
**useful for debugging why a lead did or didn't get a journey email.**

> 🔴 **Conditions 1 and 2 overlap but are not identical**, and that redundancy is the whole
> problem Joel named:
> > *"the fact that we have `comms_journey_eligible?` and `builder?` and
> > `client_ops_channel?` and `bbys_leads.ops_channel` and `provider_ops_channel? via
> > resolve_ops_channel` and `comms_variant_for_lead` is concerning — why things are so
> > complicated to change"*
>
> The recommended fix — replace `lead.builder?` with a journey predicate across the guards —
> would collapse condition 1 into condition 2 and cover builder, bank, and Orchard in one
> change. **Not yet done.** Until it is, bank partners need `builder = true` as a stopgap
> ([[bbys-priority-partners-settings]]).

## 🔑 Orchard is a hybrid, deliberately

From the code comments:

> *"Orchard is not a builder partner and stays `provider_ops_channel` (agent-first upfunnel),
> but from agreement-signed onward product enrolls Orchard in this same
> **Tiffany/Michael/Patricia** journey. Early/upfunnel builder emails must still exclude
> Orchard via `#not_orchard_builder_comms_enrollee?`."*

```ruby
def orchard_builder_comms_enrollee? = orchard? && orchard_changes_effective?
def not_orchard_builder_comms_enrollee? = !orchard_builder_comms_enrollee?
```

So Orchard leads are:
- **provider** channel, **agent-first**, through upfunnel
- **builder-journey** from agreement-signed onward
- explicitly **excluded** from early builder emails by a dedicated negative guard

> The journey is named for its owners — **Tiffany, Michael, and Patricia**. Slack referenced
> "Tiffany/Michael"; the code adds Patricia.

`SalesSetting.orchard_comms_cutover_overrides` (`{"lead_ids": []}`) allows **per-lead**
override of the cutover, and `orchard_changes_effective?` implements the July-2025 cutoff
Dave described.

### Orchard CC rules (from Slack, 2026-08-18)

Post-cutoff Orchard client journey emails CC `homelightdirect@homelight.com` +
`equityadvance@orchard.com`; **from IR Contract onward** `contracts@orchard.com` is added.
`transactionsupport@` and legacy `builders@` are removed. Implemented via an
`OrchardCc.decorate!` helper.

**Still unresolved (Slack):** after IR close, does `contracts@orchard.com` stay on, drop off,
or does everything drop to `homelightdirect@` only? Joel: *"good question, we should ask Evan
and Karly."*

## The 54 BBYS email classes

`services/lead_data_service/app/models/lead_data_service/emails/` — all inheriting
`BaseBbysEmail`, which holds the shared guards (`comms_journey_eligible?`,
`orchard_builder_comms_enrollee?`, `not_orchard_builder_comms_enrollee?`).

### Builder / client-channel (18)

`builder_client_welcome` · `builder_application_submission_confirmation_to_builder` ·
`builder_conditional_approval_to_builder` · `builder_conditional_approval_to_client` ·
**`builder_client_conditional_approval_signature_request`** ·
**`builder_client_conditional_approval_signature_request_dti_drop`** ·
`builder_agreement_signed_confirmation_client` ·
`builder_final_agreement_generated_client` · `builder_apply_for_equity_boost_client` ·
`builder_approved_for_equity_boost` · `builder_equity_boost_missing_documents` ·
`builder_missing_photos_alert` · `builder_photo_submission_confirmation` ·
`builder_property_conditions_and_repairs_request_client` ·
`builder_notary_scheduling_request_client` · `builder_client_request_ir_contract_upload` ·
`builder_handoff_to_ca_for_builder` / `_for_client` ·
**`builder_handoff_to_ca_for_builder_dti_drop`** / `_for_client_dti_drop`

> ✅ **Two Slack items resolved.** The template-name mismatch Joel flagged
> (tracker: `BBYS_Builder_conditional_Approval_to_client_DTI_Drop`; DB:
> `BBYS_Builder_Client_conditional_Approval_signature_request_DTI_Drop`) — **the class name
> confirms the DB name is correct**; the tracker is wrong.
> And `bbys_builder_handoff_to_ca_for_builder_dti_drop`, listed as "not yet built" on
> 2026-08-18, **now exists as a class**.

### Orchard-specific (2)
`orchard_submission_confirmation_to_dr_agent` · `orchard_missing_photos_alert_to_dr_agent`
— both **DR-agent** targeted, consistent with the agent-first upfunnel.

### Provider / LO / agent
`ir_contract_dr_listed_and_next_steps_agent` ·
`ir_contract_dr_not_listed_process_and_next_steps_agent` ·
`ir_contract_dr_listing_process_intro_lo` ·
`ir_contract_lo_prior_to_funding_final_confirmation` ·
`iruc_dr_is_listed_final_agreement_signed_next_step_to_agent` ·
`iruc_dr_not_listed_final_agreement_signed_next_step_to_agent` ·
`final_agreement_listing_plan_follow_up_email_agent` ·
`final_agreement_preferred_escrow_intro_email_agent` ·
`agreement_signed_next_step_to_client`

### BR-sourced (builder rep)
`br_sourced_new_client_next_steps` · `br_sourced_conditional_approval_to_lo_reminder`

### Operational
`dr_in_escrow_payoff_request` · `wire_instructions_request` · `upfunnel_document_request` ·
`mortgage_calculator_scenario_comparison` · `day_0_all_status` · `day_0_all_status_byoc`
(BYOC = bring your own client)

## 🔑 The post-EU-funding escalation ladder

The email class names encode a precise day-based cadence after equity-unlock funding — this
is the **DR-not-listed nurture ladder**:

| Day | Email | Target |
| --- | --- | --- |
| **0** | `day_0_all_status` / `day_0_all_status_byoc` | all |
| **7** | `day_7_post_eu_funding_dr_not_listed_agent_check_in` | **agent** |
| **17** | `day_17_post_eu_funding_dr_not_listed` | |
| **21** | `day_21_post_eu_funding_dr_not_listed` | |
| **28** | `day_28_post_eu_funding_dr_not_listed` | |
| **30** | `day_30_post_eu_funding_activity_update_request_agent` | **agent** |
| **35** | `day_35_post_eu_funding_dr_not_listed` | |
| **42** | `day_42_post_eu_funding_dr_not_listed` | |
| **45** | `day_45_post_eu_funding_activity_update_request_agent` | **agent** |
| **80** | `80_days_post_eu_funding_end_of_program_approaching_agent` | **agent** |
| **90** | `90_days_post_eu_funding_end_of_program_approaching` | |
| **150** | `day_150_second_extension_vs_purchase_alert` | |
| **160** | `day_160_second_extension_vs_purchase_alert` | |

Plus `dr_sale_contract_cancellation_extension_or_purchase_alert` (event-driven).

> 🔑 **This ladder is the clearest statement of the BBYS operating tempo that exists.**
> Read alongside the task calendar in [[bbys-lifecycle-operations]] (Day 85 refresh
> valuation → Day 91 extend-vs-buy → Day 100 extend-vs-buy → 20/10 days to expiry) and the
> **120-day** `BASE_DAYS` program period, the picture is:
>
> - **Days 0–45** — chase the DR listing, escalating to the agent
> - **Days 80–100** — end of program approaching; extend vs. buy decision
> - **Days 150–160** — *second* extension vs. purchase
>
> Days 150/160 imply a **second extension** well past the 120-day base — consistent with
> `MAX_EXTENSIONS = 3`.
>
> ⚠️ Note the ladder counts from **EU funding**, while the extension clock counts from
> `BASE_DAYS` off a different base date. **"Day 90" in an email is not "day 90" in the
> extension math.** Confirm the anchor before correlating them.

## Where template names live

- **SendGrid** holds the rendered templates; `bbys_notary_scheduling_valid_template` suggests
  template validation exists
- Milestone/notification slugs are catalogued in [[bbys-lifecycle-operations]]
- The **comms journey tracker spreadsheet** (Dave/Joel) is the business-side map and
  **does not always match the DB template names** — see the mismatch above

## Support inboxes

| Channel | Inbox |
| --- | --- |
| Client channel (builder / generic / bank) | **`homelightdirect@homelight.com`** |
| Legacy builder | `builders@` — still an alias for in-flight items |
| Provider channel | `transactionsupport@homelight.com` |
| Orchard | `equityadvance@orchard.com`, + `contracts@orchard.com` from IR Contract |

**Open ops item:** confirm `builders@` forwards to `homelightdirect@` in SendGrid.

> Joel flagged (2026-06-10) that **listing-ops emails static-CC `transactionsupport@`** and
> should use Dave's configurable support-email mechanism like the upfunnel emails do.
> Status unknown.

## Known gaps

- **Cardinal Financial** explicitly de-scoped from the bank work
- Two client-channel templates were "not yet built" as of 2026-08-18: the client inspection
  reminder (slug now exists: `bbys_client_channel_inspection_reminder_for_client`) and the
  builder DTI-drop handoff (**class now exists**)
- `BBYS_Lender_CTC_Wire_Confirmation` should send for **all** channels, with extra CCs on
  Orchard — status unconfirmed
- The allowlist question Joel corrected: listing-ops emails apply to **all** journeys; the
  analysis that said otherwise was wrong

## Open questions

- Has the `lead.builder?` → journey-predicate refactor started?
- Does `contracts@orchard.com` stay CC'd after IR close?
- How is `BBYS_lender_iruc_ca_handoff_dti_drop` actually sent? Joel: *"idk ask faaz."*
- Is `bbys_builder_property_conditions_and_repairs_request_client` live? Joel: *"check the
  code"* — **the class exists**, but whether a journey references it is unverified.
