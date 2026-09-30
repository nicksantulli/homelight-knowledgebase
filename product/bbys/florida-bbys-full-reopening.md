---
created: 2026-07-01
type: sop
status: active
owner: Jason Smith
source: "Florida Full Re-Opening (July 2026).docx"
privacy: INTERNAL USE ONLY
related:
  - [[2026-07-01-bbys-dr-underwriting-buy-box-source-of-truth]]
  - [[2026-07-01-florida-zip-county-cltv-reference]]
  - [[bbys-overview]]
source: HomeLight-Vault/sops/2026-07-01-florida-bbys-full-reopening.md
imported: 2026-09-29
---

# Florida BBYS Full Re-Opening (July 2026)

**INTERNAL USE ONLY** · **July 2026**

> Full relaunch of remaining Florida BBYS markets after phased re-entry (Orlando/Jacksonville Jan 2025 → Central/East Apr 2025 → Tampa Jan 2026). Pair with [[2026-07-01-bbys-dr-underwriting-buy-box-source-of-truth]] for national buy-box rules.

## Context

Florida operations were suspended in **October 2024** due to substantial losses and hurricane-related catastrophe risk. After refining underwriting standards and pricing, HomeLight executed a phased rollout and is prepared for **full relaunch** of the remaining Florida markets.

## CLTV Zones (Equity Unlock only)

Florida is split into two regions by county. Limits are based on **HomeLight proprietary valuation**.

| Max CLTV | Applies to |
|---|---|
| **75%** | Inland / lower-exposure counties (see list below) |
| **70%** | Coastal / higher-exposure counties (see list below) |

**Important:** These 70% / 75% limits apply to **Equity Unlock only**.

- **Asset Equity Boost:** same **85% CLTV** (HouseCanary value) as other areas
- **HELOC Boost:** same **90% CLTV** (LO DR value) as other areas

### Counties — 75% CLTV Limit

Alachua, Baker, Bay, Bradford, Brevard, Calhoun, Citrus, Clay, Columbia, Dixie, Duval, Escambia, Flagler, Franklin, Gadsden, Gilchrist, Gulf, Hamilton, Hernando, Hillsborough, Holmes, Indian River, Jackson, Jefferson, Lafayette, Lake, Leon, Levy, Liberty, Madison, Marion, Martin, Nassau, Okaloosa, Okeechobee, Orange, Osceola, Pasco, Pinellas, Polk, Putnam, Santa Rosa, Seminole, St. Johns, St. Lucie, Sumter, Suwannee, Taylor, Union, Volusia, Wakulla, Walton, Washington

### Counties — 70% CLTV Limit

Broward, Charlotte, Collier, DeSoto, Glades, Hardee, Hendry, Highlands, Lee, Manatee, Miami-Dade, Monroe, Palm Beach, Sarasota

### ZIP-level lookup

Full ZIP reference: [[2026-07-01-florida-zip-county-cltv-reference]] (1,495 ZIPs — CSV in `exports/florida-zip-county-cltv-reference-july-2026.csv`).

## Location & Property Type Restrictions

- **Single-family detached only** — no townhomes or condos
- Property must be **> 2 miles** from Gulf or Atlantic coast
- No part of property or lot in FEMA Special Flood Hazard Area (Zones A, AE, A1–A30, AH, AO, AR, A99, V, VE, V1–V30)
- No properties in known documented active sinkhole areas
- All other standard BBYS eligibility rules apply (see buy-box SOP)

## Standard Fee

Default BBYS fee for all Florida properties: **2.9%**

## Insurance Requirements

Properties must be insured for **replacement cost** for at least the LPV amount at time of funding.

## HOA Review

All HOA properties undergo HOA review before final approval — CC&Rs and bylaws checked for BBYS eligibility alignment.
