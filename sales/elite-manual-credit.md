---
last_updated: 2026-09-17 ~11:10 MT
date: 2026-09-17
type: decision
status: live
owner: Nicholas Santulli
tags: [hubspot, elite, manual-credit, lo]
portal: 4744876
related:
  - [[lo-elite-program]]
  - [[hubspot]]
  - [[2026-09-16-elite-qualifying-ytd-hubspot-calcs]]
source: /workspace/standup-20260917/2026-09-17-elite-manual-credit.md
source: HomeLight-Vault/decisions/2026-09-17-elite-manual-credit.md
imported: 2026-09-29
---

# Elite manual credit (HubSpot) — how to adjust LOs

**Date locked:** 2026-09-17  
**Owner:** Nick / RevOps  
**Why:** One-off Elite transaction credits (e.g. negotiated HELOC-card deals) without dummy IR deals or per-LO formula hacks.

## Properties (contact)
| Internal name | Type | Use |
|---|---|---|
| `elite_manual_credit_ytd` | number | Can be positive or negative. Blank/0 = no adjust. |
| `elite_manual_credit_note` | text | Audit: who / when / why |

## Formula
`elite_qualifying_ir_closes_ytd` = (existing DTI-capped IR qualifying logic) + COALESCE(`elite_manual_credit_ytd`, 0)

- Feeds `elite_lender_status` (Elite / Diamond / Obsidian + GF).
- **`ir_closings_ytd` stays raw** — do not use manual credit for revenue/IRUC volume reporting.
- DTI new-flat “first close only” rules and Diamond/Obsidian ladder unchanged.

## How to adjust another LO
1. Open the LO contact in HubSpot.
2. Set `elite_manual_credit_ytd` (e.g. `1`, `2`, or `-1`).
3. Set `elite_manual_credit_note` with date, approver, and reason.
4. Confirm `elite_qualifying_ir_closes_ytd` and `elite_lender_status` update (calc lag possible).
5. Clear / zero at year reset if the credit was YTD-scoped.

## Do not
- Hardcode emails into the Elite formula.
- Create dummy IR-closed BBYS deals for Elite points (pollutes revenue / `ir_closings_ytd`).
- Put Kim Tanner or John Labrada as LO `hubspot_owner_id`.

## First use
- **Ashlee Sheppard** contact `197928080812` · ashlee.sheppard@fairwaymc.com
- Credit **+1** on 2026-09-17 (Nick): Fairway HELOC-card negotiation (3 cards sold); not an IR close.
- Before: qualifying 8 · GF Diamond - 8  
- After: qualifying 9 · GF Diamond - 9 · `ir_closings_ytd` still 8  
- HubSpot: https://app.hubspot.com/contacts/4744876/record/0-1/197928080812  
- Exec artifact: `/workspace/hubspot-orientation/standup-20260917/elite-manual-credit-execute-20260917.json`

## Suggested vault homes
- `decisions/2026-09-17-elite-manual-credit.md` and/or HubSpot Elite section under context/projects — Curator pick existing pattern; no new system.
