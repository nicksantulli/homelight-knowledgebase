---
date: 2026-04-09
type: decision
status: active
tags: [data-bridge, partner-matching, deal-lo-association]
source: HomeLight-Vault/decisions/2026-04-09-partner-waterfall-reorder.md
imported: 2026-09-29
---

# Decision: Partner Slug Is the Authoritative Signal in Deal Partner Matching

## What We Decided
Reordered the `findPartnerDeterministic` waterfall in the deal-LO association pipeline so that the deal's `partner_slug` (set at intake) is the first step checked — ahead of enterprise fields, domain maps, and all other signals.

New waterfall order (10 steps):
1. **Partner slug** — search partnerships by deal's `partner_slug`
2. **Email comms** — wholesale lender signals in contact email history
3. **Enterprise field / domain map** — `hldb_lender_enterprise_partner` keywords + `DOMAIN_TO_PARTNER` lookup
4. **Deal Slack channel** — scan messages for partner domains/keywords
5. **Partnership domain search** — search partner objects by LO email domain
6. **Contact's company associations** — company → partnership associations
7. **Contact's existing partner associations** — partner objects linked to the LO contact
8. **LO's other BBYS deal properties** — deal history for this LO
9. **LO's other deal partner associations** — partner associations on LO's past deals
10. **Peer LOs from same company** — other LOs at same company, their deal partners (skipped for non-partner lender domains)

Also added:
- **Agent slug override** in the trigger's agent fallback — if the AI agent picks a partner whose slug doesn't match the deal's `partner_slug`, the handler overrides with the slug-matched partner, removes wrong associations, and associates the correct one.
- **New-contact company domain lookup** — when a contact is newly created (empty properties), the pipeline now searches companies by email domain before running the assignment waterfall, so company LSM/pod routing works for first-time LOs.

## Why
Deal 59051258956 came in with `partner_slug: uwm` but got associated with TLS (The Loan Store). Root cause: the LO (`gawada@emortgagecapital.com`) works at E Mortgage Capital, a broker that routes loans through multiple wholesale lenders. The old waterfall checked enterprise field/domain map first (step 1), which matched TLS before the slug was ever consulted.

Broker LOs are inherently ambiguous for domain/company-based matching — the slug from intake is the only explicit signal about which partner this specific deal belongs to.

A second deal (58882024089) revealed that newly created contacts got the wrong LSM assignment (Matt instead of Kara) because `findOrCreateContact` returns empty properties for new contacts, causing the assignment waterfall to skip company-based routing and fall to round-robin.

## Impact
- Partner slug now overrides all other signals — if a deal has a slug, the matching partner will be found regardless of the LO's enterprise field or company associations
- Broker LOs (NEXA, E Mortgage Capital, etc.) will correctly route to UWM or TLS based on the deal's slug, not the broker's company associations
- New LO contacts will get correct LSM assignment on first deal (no more round-robin fallback for contacts with known companies)
- Agent fallback has a safety net: even if the deterministic waterfall fails, the agent's result is validated against the deal's slug

## Additional Fix (2026-04-09) — Slug Correction on Re-runs

A second bug (deal 58997203012) revealed that when `partner_slug` is set **after** deal creation, re-running the workflow would stack associations instead of correcting them. The slug matched UWM, but TLS (from the initial email-domain match) remained associated.

Fix: in `runDealLoAssociation`, when the match reason starts with `"Deal partner_slug:"`, the code now fetches all current deal↔partnership associations before calling `classifyAndAssociatePartner`. Any existing Main or Secondary association to a **different** partner is removed first. This ensures the correct partner is always the only one associated after a slug-triggered re-run.

## Additional Fix (2026-04-09) — `main_partner__ai_` Is Read-Only

`main_partner__ai_` and `secondary_partner__ai_` are HubSpot **calculated properties** — HubSpot computes them automatically from the deal's association labels. All code that previously tried to write or clear these properties has been removed:
- `classifyAndAssociatePartner` — was writing the partner ID after association
- Sub-partner parent resolution — was writing parent ID to `main_partner__ai_`
- Slug-correction block — was clearing the property after removing a stale association
- `handleWholesaleLenderUpdate` in the trigger — was clearing on wholesale partner swap

HubSpot updates the calculated values automatically when associations change; no code writes are needed.

## Open Issue
The deterministic path's slug correction removes the wrong partner association, but the subsequent PUT to add the correct partner sometimes fails with a HubSpot cardinality error ("One or more association limits exceeded"). The agent slug override catches this, but a retry/delay after removal would be cleaner.

## Revisit Date
2026-05-01 — check if the cardinality error is still occurring and whether a fix is needed.

## Related Notes
- [[data-bridge]]
- [[partners]]
- [[deal-operations]]
