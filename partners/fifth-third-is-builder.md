---
date: 2026-09-04
type: decision
status: active
tags: [reorg, hubspot]
source: HomeLight-Vault/decisions/2026-09-04-fifth-third-is-builder.md
imported: 2026-09-29
---

# Decision: Fifth Third is Builder (supersedes morning packet)

## What We Decided

Fifth Third Bank Company `5301241106` is **Builder**.

- `assigned_pod` = Builder
- `lender_sales_account_executive` = Plamondon `331990682`
- `lender_sales_manager` = Tiffany;Coffey `798340532;90311375`

This supersedes the 2026-09-04 morning packet answer that Fifth Third was Retail under the banks rule.

## Why

Nick, 2026-09-04 afternoon, after a meeting. Granola had no captured notes for that meeting yet.

## Impact

Phase 10 LSM write to the Builder pool is **correct** and must not be reversed to Retail. Phase 7 Retail pod and Phase 9 Kim AE on this company must be corrected in a follow-on one-property sequence. Classifier `BUILDER_POD_OVERRIDE_IDS` beats live Retail pod and the bank allowlist.

## Revisit Date

If meeting notes land in Granola, attach the citation; policy does not wait on that.
