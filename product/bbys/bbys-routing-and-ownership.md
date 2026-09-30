---
last_updated: 2026-08-29
status: current
source: homelight/hapi @ c14079c76c
scope: how BBYS leads are routed to LSM/AE, LRM/AM, BRM, and CA — and why LO ownership is not queryable
source: HomeLight-Vault/context/bbys-routing-and-ownership.md
imported: 2026-09-29
---

# BBYS routing and ownership

How a BBYS lead gets its people assigned, what the roles actually are, and **why nobody can
answer "which AM owns this LO?"**

Read with [[bbys-priority-partners-settings]], [[team]], [[lo-lifecycle]],
[[2026-08-28-vault-refresh-slack-findings]].

## 🔴 Answering Jake's question

On 2026-08-21 Jake asked (via Joel, `#homes-ops-and-efficiency-pod`):

> *"Right now, we don't have a clear way to identify which AM/LRM actually owns a particular
> LO. I'd like to understand what data or logic is available so we can figure out the best
> way to clearly identify and display ownership going forward."*

**The code explains why. There is no LO→AM ownership record.**

Ownership is **derived at lead-routing time** and only the *outcome* is persisted — as a
`lead_users` row on **that one lead**. There is no `loan_officer.owning_lrm_id`, no
LO-to-AM mapping table, and no partner-to-AM table. Every "who owns this LO" question is
answered today by **inferring from the leads they happen to have submitted**, which is why
answers differ depending on who runs the query and over what window.

Worse, the LSM/AE half of routing is resolved by **calling HubSpot at routing time**
(contact → company → company owner → HomeLight user by email). So ownership is:

1. computed from HubSpot state **as of the moment the lead was created**,
2. never written back as an LO attribute,
3. and invisible to anyone who wasn't watching the routing event.

> **If the business wants durable LO ownership, it has to be created — it does not exist to
> be found.** Options worth discussing: a first-class `loan_officer_owner` assignment,
> or promoting the HubSpot contact/company owner to the system of record and mirroring it.
> Either is a project, not a query. This is a good thing to say plainly before Sep 1.

## The routing model: `pod_based_v2`

Captured in the versioned event **`bbys_lead_routing_outcome`** (v1.0.2). Every routing
decision emits one, with a full decision trace.

### What the event records

| Field | Meaning |
| --- | --- |
| `routing_source` | the interactor class that routed |
| `routing_strategy` | e.g. **`pod_based_v2`** |
| `lsm_lookup_path` | `special_case_lennar` · `special_case_builder` · `regular_pod_lookup` |
| `lsm_assignment_method` | `hubspot_owner_lookup` · `partner_lsm_fallback` |
| `lrm_lookup_path` | `pod` · `special_case_builder_no_lrm` · `fallback_round_robin` |
| `lrm_assignment_method` | `pod_experienced` · `builder_uses_brm_team` · `fallback_round_robin` |
| `brm_assignment_method` | `special_case_builder_round_robin` · `special_case_builder_no_brms` |
| `final_lsm_user_id` / `final_lrm_user_id` / `final_brm_user_id` | the outcome |
| `transaction_team_id` | the team created |
| `status` | see below |

**Statuses:** `success` · `failure_no_lrm_found` ·
`failure_lrm_found_no_transaction_team` · **`failure_orchard_pod_missing`**

> 🔑 **This event is the audit trail for routing** and it is genuinely rich — it records the
> candidate pool (`available_lrm_emails_for_fallback`, `available_lrm_ids_for_round_robin`),
> whether an experience check ran, and the full HubSpot lookup trace. **If you want to answer
> "why did this lead go to this AM," query this event, not the lead.**

### The HubSpot lookup chain (LSM/AE assignment)

The event records each step, which tells you the algorithm:

```
hubspot_contact_search_attempted → hubspot_contact_found → hubspot_contact_id
  → hubspot_company_lookup_attempted → hubspot_company_found → hubspot_company_id
    → hubspot_owner_lookup_attempted → hubspot_owner_found → hubspot_company_owner_id
      → partner_user_lookup_attempted  (User.find_by on the owner's email)
        → final_lsm_user_id
```

Fallbacks: `partner_lookup_loan_officer_present` → `partner_lsm_found` →
`partner_lsm_fallback`. Also `lsm_pod_found`, `lsm_pod_team_id`, `lsm_found_via_hubspot`,
`lsm_pods_group_exists`, `initial_lsm_user_id`.

> 🔴 **HubSpot company ownership is the de-facto source of truth for AE assignment.** So:
> - Reassigning a company owner in HubSpot silently changes routing for future leads.
> - A HubSpot company **merge changes the company ID** — and the C2 Financial merge changed
>   it 18 times ([[2026-08-28-vault-refresh-slack-findings]]).
> - A HomeLight user whose email doesn't match the HubSpot owner's email **fails
>   `partner_user_lookup`** and falls through to the fallback.
>
> **Before Sep 1, the AE/AM rename must be reflected in HubSpot ownership, not just in
> titles** — otherwise leads keep routing to the old owner.

## BRM pods

`BrmPodTeam < Team` — STI on `teams`. From the model's own documentation:

> *"BrmPodTeam represents a team structure that manages the relationship between Builder
> Relationship Managers (BRMs) and Lender Relationship Managers (LRMs). The pod system is
> designed to: 1. Maintain consistent BRM-LRM relationships 2. Support special case routing
> for builder partners 3. Enable efficient lead distribution through round-robin assignment"*

**Two pod kinds:**

| Kind | Naming | Behavior |
| --- | --- | --- |
| **Regular pods** | `"BRM Pod - {email}"` (one per BRM) | multiple LRMs per pod; standard routing |
| **Special pods** | `SPECIAL_PODS = { builder: "Builder BRM Pod" }` | **rotating BRMs**; protected from deletion |

Round-robin runs **both** across BRMs and across LRMs within a pod. Created via
`BrmPodTeam.create_for_user(brm_user)`, LRMs added with `add_lrm`.

> 🔑 **Pod membership is keyed on the BRM's email address** (`"BRM Pod - %{email}"`). An
> email change orphans the pod.

> This is the **"BRM pod round-robin"** Joel referenced for bank leads:
> *"banks should use tiffany/michael client facing routing … so we do not want them to go to
> regular LSM/LRMS."* The mechanism is the **Builder BRM Pod** special pod. Since bank
> partners currently need `builder = true` as a stopgap ([[bbys-priority-partners-settings]]),
> **banks are routed through the builder pod by exploiting the builder flag** — which is
> exactly the fragility Joel wants removed.

### Operator availability

`HomesOperatorRoundRobinSetting` — `belongs_to :user`, an `available` boolean, and a role
enum:

```ruby
enum role: { lrm: "lrm", ca: "ca", tp: "tp", brm: "brm" }
```

**LRM · CA · TP · BRM.** `scope :available` filters the round-robin pool.

> 🔑 **This is the on/off switch for whether a person receives new leads.** It is per-role,
> so someone can be available as a CA but not as an LRM. **This table is the operational
> lever for the Sep 1 cutover** — it, plus pod membership, plus HubSpot company ownership,
> is what actually has to change.
>
> ⚠️ Note the enum has **no `lsm`/`ae` role** — AE assignment comes from HubSpot, not from
> this table. The two halves of routing are governed by different systems.

## 🔑 The role vocabulary

`TransactionTeam::EMPLOYEE_ROLES` — the canonical list. Slug → label:

| Slug | Label |
| --- | --- |
| `lender_sales_manager` | **Lender Sales Manager** (LSM → becoming **AE**) |
| `lender_relationship_manager` | **Lender Relationship Manager** (LRM → becoming **AM**) |
| `lo_sales_owner` | **LO Sales Owner** |
| `builder_relationship_manager` | Builder Relationship Manager (BRM) |
| `builder_representative` | Builder Representative |
| `client_advisor` | **Client Advisor** |
| `client_advisor_assistant` | Client Advisor Assistant |
| `contract_advisor` | **Contract Advisor** |
| `client_manager` | Client Manager ⚠️ deprecated |
| `consumer_client_manager` | Consumer Client Manager |
| `deal_manager` | Deal Manager |
| `lender_operations_specialist` | **Lender Operations Specialist (LOS)** |
| `listing_operations_specialist` | Listing Operations Specialist |
| `listing_specialist` | Listing Specialist |
| `processor` | Processor |
| `property_analyst` | Property Analyst |
| `sales_specialist` | Sales Specialist |
| `escrow_officer` | Escrow Officer |
| `agent_account_manager` | Agent Account Manager (AAM) |
| `agent_sales_owner` | Agent Sales Owner |
| `agent_success_manager` | Agent Success Manager |
| `loan_officer` | Loan Officer ⚠️ deprecated |
| `loan_officer_assistant` | Loan Officer Assistant (LOA) ⚠️ deprecated |
| `loan_officer_team_lead` | Loan Officer Team Lead |
| `loan_officer_additional_contact` | LO Additional Contact ⚠️ deprecated |

Plus `LeadUser`-specific: `team_coordinator`, `lender_client_advisor`.
`ClientAdvisoryTeam::ROLES` is a two-role subset: `client_advisor` +
`client_advisor_assistant`.

### 🔑 "CA" is ambiguous — there are two of them

**`client_advisor`** and **`contract_advisor`** are **both real, distinct roles.**

This resolves an inconsistency flagged earlier: the sales-app dashboard code says
*Client Advisor* while Faaz's launch post called it the *"Contract Advisor Dashboard"*
([[repo-sales-app]], [[2026-08-28-vault-refresh-slack-findings]]). **Both terms are correct
— for different roles.** When someone says "CA," ask which.

The round-robin enum uses `ca` — a single value that does not disambiguate.

### Deprecated roles

```ruby
DEPRECATED_ROLES = [client_manager, loan_officer,
                    loan_officer_assistant, loan_officer_additional_contact]
```

Filtered out of `employee_roles` **only when the Flagsmith flag
`sales-ops-centralized-lead-distribution` is enabled** (cached 5 minutes).

> 🔴 **`loan_officer` and `loan_officer_assistant` are on the deprecated list**, yet
> `LeadUser` still has a dedicated `loan_officer_assistants` scope, sales-app has
> `AddLoanOfficerContact` / `AddLoanOfficerAssistantContact` / `LoanOfficerAssistantSection`,
> and Slack traffic is full of LOAs. **The deprecation is conditional on a flag** — so the
> role list literally differs between environments depending on flag state. Verify the flag
> before trusting any role-based report.

## How people attach to a lead

`lead_users` is deliberately minimal: `lead_id` · `user_id` · `role` · timestamps.

- `scope :employees` — rows whose role is in `LEAD_USER_ROLES`
- `scope :loan_officer_assistants` — role `loan_officer_assistant`, ordered by
  `created_at, id`

`LEAD_USER_ROLES = TransactionTeam.employee_roles.merge(ClientAdvisoryTeam::ROLES)` — with a
`TODO` in the code to replace it with `TransactionTeam::EMPLOYEE_ROLES` directly.

> 🔑 **`lead_users` is per-lead, not per-LO and not per-partner.** It is the *only* place the
> routing outcome is durably stored. That single design fact is the whole answer to Jake's
> question.

## Special-case routing

- **Lennar** — `special_case_lennar` LSM path *(the team no longer works Lennar, so this
  path is presumably dead — worth removing)*
- **Builder** — `special_case_builder`, `builder_uses_brm_team`,
  `special_case_builder_round_robin`, `special_case_builder_no_brms`,
  `special_case_builder_no_lrm`
- **Orchard** — has its own pod; a missing pod produces the distinct failure status
  **`failure_orchard_pod_missing`**
- **Manual override** — `ApplyManualBuilderTeamRouting` lets ops force builder-team routing

> Three of the four special cases (**Lennar, Orchard**, and builder-flag-based bank routing)
> are for partnerships the team is exiting or working around. The special-case machinery is
> carrying more history than current business.

### Round-robin fairness

`RoundRobinPenalty` (`round_robin_penalties`) with
`LeadUsers::UpdateRoundRobinPenalty` — the rotation is penalty-adjusted, not naive.

> Related bug fixed 2026-08-27 (PR #20113): **Orchard builder-routed leads incremented the
> BRM round-robin counter on every re-save without assigning a BRM**, skewing Michael's and
> Tiffany's split. Re-saves no longer move the counter. **Any BRM split analysis before
> 2026-08-27 is distorted.**

## What has to change for Sep 1

Derived from the above — the AE/AM rename touches at least five places, none of which are
the org chart:

1. **HubSpot company owners** — drives AE assignment via `hubspot_owner_lookup`
2. **`HomesOperatorRoundRobinSetting`** — `available` + role per user (LRM/CA/TP/BRM)
3. **BRM pod membership** — pods are named by BRM email; LRM lists per pod
4. **`SalesSetting.bbys_homelight_lsm_account_executive_names`** — a hardcoded 9-name list
   ([[bbys-priority-partners-settings]])
5. **Round-robin entries for Orchard / Lennar / D.R. Horton** — Jake asked for these to be
   removed

Plus the cosmetic layer: role labels (`Lender Sales Manager` → `Account Executive`,
`Lender Relationship Manager` → `Application Manager`), portal copy, and email templates.

> ⚠️ Renaming the **labels** without changing the **slugs** is the low-risk path —
> `lender_sales_manager` and `lender_relationship_manager` are referenced across HAPI,
> sales-app, BI (`bbys_denials_filter` filters on `lender_relationship_manager`), and
> HubSpot. **Changing slugs would break BI silently.** Recommend label-only.

## 2026-08-29 addendum — the identity primitive now exists

A **`loan_officer_identities`** table shipped ~2026-06-12 (verified in schema @ HEAD):
NMLS-keyed, with a unique 8-char `homelight_identification_code`, plus
**`loan_officer_account_transitions`** (old/new user + LO records, `moved_by_user_id`,
`reason`, soft-disable) tracking company moves. **This is identity continuity, not
ownership** — the core claim above stands — but it solves the "LO changes companies and we
lose the thread" half of the problem, and any future LO→AM ownership model should key off
`loan_officer_identity_id`, not the per-company `loan_officer` row. Relevant to the TLS→
Averra rename and the Bay Equity→Rocket repointing in [[partners]].

## 2026-08-29 addendum — a live LO-facing routing endpoint is dead in production

🔴 **`loan-officers.homelight.com` is configured-but-dead** — relevant here because it's an
LO-facing endpoint the monolith still actively tries to route people to.
`working/2026-08-29-critic-technical.md` §2.6 confirmed by live check: **DNS returns no
records** for `loan-officers.homelight.com` (control `www.homelight.com` → 200). But
`homelight/config/routes.rb:41–47` still hardcodes it as the production base for
`direct :loan_officer_sales_app`, and
`homelight/app/controllers/sales/agent_services_controller.rb:53` still calls
`loan_officer_sales_app_url(result)` — **that controller action emits a broken link to LOs in
production today.** Full writeup: [[repo-sales-react]] ("`loan-officer-crm` /
`loan-officers.homelight.com` liveness"). The live LO-facing surface is
`equity.homelight.com`, itself mid-migration into `homes-fe` — not this dead host. Do not
treat `loan_officer_sales_app_url` as a working routing target for any future ownership/
assignment work.

## Open questions

- Is there any job that back-fills LO→AM ownership, or is `lead_users` genuinely the only
  record?
- ~~Is the `special_case_lennar` path still reachable now that Lennar is out?~~ **Answered
  2026-08-29:** it survives only as a documented enum *example value* in an event schema —
  `hapi/services/event_service/app/classes/event_service/schema/bbys_lead_routing_outcome.rb:19`
  lists `special_case_lennar` in a comment (`# e.g., "special_case_lennar", "special_case_builder",
  "regular_pod_lookup", etc.`), not a live branch. Actual reachability depends on the routing
  caller, which this file doesn't gate — unconfirmed whether any caller still emits it.
- `tp` in the round-robin role enum — **partially answered 2026-08-29:** the definition
  site is `hapi/app/models/homes_operator_round_robin_setting.rb:11` (`enum role: { lrm:
  "lrm", ca: "ca", tp: "tp", brm: "brm" }`). The **expansion is still unconfirmed** — grepped
  `hapi` and `sales-app` for a "Transaction Processor" / "TP" label near this enum and found
  none. Leaving open.
- Is `sales-ops-centralized-lead-distribution` on in production? It changes the live role list.
- Does anything consume `bbys_lead_routing_outcome` today, or is it write-only? It is the
  best available ownership audit trail.
