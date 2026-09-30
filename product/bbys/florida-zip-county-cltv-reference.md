---
created: 2026-07-01
type: reference
status: active
source: "Florida ZIP _ County _ CLTV Reference (July 2026).xlsx"
privacy: INTERNAL USE ONLY
related:
  - [[2026-07-01-florida-bbys-full-reopening]]
  - [[2026-07-01-bbys-dr-underwriting-buy-box-source-of-truth]]
source: HomeLight-Vault/sops/2026-07-01-florida-zip-county-cltv-reference.md
imported: 2026-09-29
---

# Florida ZIP / County / CLTV Reference (July 2026)

**INTERNAL USE ONLY**

> ZIP-level lookup for Florida Equity Unlock CLTV limits. Pair with [[2026-07-01-florida-bbys-full-reopening]] for policy context.

## Summary

| CLTV zone | ZIP count |
|---|---|
| **75%** | 1,005 |
| **70%** | 490 |
| **Total** | 1,495 |

- **67 Florida counties** represented; each county maps to exactly one CLTV zone (no conflicts).
- CLTV values apply to **Equity Unlock only** (Asset Equity Boost 85%, HELOC Boost 90% unchanged).

## How to use

1. Look up the departure-property **ZIP code** in the CSV export.
2. Read the `cltv_pct` column (**70** or **75**).
3. Confirm county and metro area match the subject property.
4. Apply all other Florida restrictions (SFR detached, >2mi from coast, flood/sinkhole, buy box).

## Data file

Full machine-readable export: [`exports/florida-zip-county-cltv-reference-july-2026.csv`](../exports/florida-zip-county-cltv-reference-july-2026.csv)

Columns: `zip_code`, `metro_area`, `county`, `cltv_pct`

## County → CLTV (quick reference)

### 75% counties

Alachua, Baker, Bay, Bradford, Brevard, Calhoun, Citrus, Clay, Columbia, Dixie, Duval, Escambia, Flagler, Franklin, Gadsden, Gilchrist, Gulf, Hamilton, Hernando, Hillsborough, Holmes, Indian River, Jackson, Jefferson, Lafayette, Lake, Leon, Levy, Liberty, Madison, Marion, Martin, Nassau, Okaloosa, Okeechobee, Orange, Osceola, Pasco, Pinellas, Polk, Putnam, Santa Rosa, Seminole, St. Johns, St. Lucie, Sumter, Suwannee, Taylor, Union, Volusia, Wakulla, Walton, Washington

### 70% counties

Broward, Charlotte, Collier, DeSoto, Glades, Hardee, Hendry, Highlands, Lee, Manatee, Miami-Dade, Monroe, Palm Beach, Sarasota
