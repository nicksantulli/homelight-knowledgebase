---
date: 2026-03-26
type: decision
status: active
tags: [data-bridge, partnerships, hubspot, associations]
source: HomeLight-Vault/decisions/2026-03-26-partner-cascade.md
imported: 2026-09-29
---

# Decision: Sub-Partner → Main Partner Associations Cascade to Contacts

## What We Decided
When a contact is associated with a sub-partner, the system fetches that sub-partner's partner-to-partner associations. For each parent partner with `partnership_hierarchy = "Main Partner"`, a contact→partnership association is created with the "Lender Partner (Main)" label.

Additionally, the Main vs Sub label is now determined by the `partnership_hierarchy` field on the partnership object itself, NOT by whether the partnership domain matches the contact's company domain.

## Why
Real-world case: Xpert Home Lending is a sub-partner with partner-to-partner associations to The Loan Store (Main) and United Wholesale Mortgage (Main). When a contact at Xpert was enrolled, only the direct sub-partner was associated — the main partners were missing.

The old approach (inferring Main/Sub from company↔partner domain match) was wrong for many cases. NFM Lending has `partnership_hierarchy = "Main Partner"` but the domain doesn't match the company — so it was labeled Sub incorrectly.

## Impact
- `associationFixer.ts` — Step 4b cascades partner-to-partner associations
- `associationFixer.ts` — Step 4 fetches `partnership_hierarchy` to determine label
- Cascaded partners are included in the "valid set" so stale removal doesn't delete them
- Uses `p_partnerships` schema name (not `2-35285415`) for v4 associations API

## Revisit Date
Q3 2026 — check if partner-to-partner structure has changed or if new hierarchy levels exist.

## Related Notes
- [[2026-03-25-data-bridge-enrichment-assignment]]
