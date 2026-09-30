---
last_updated: 2026-08-26 (BBYS deal-failure taxonomy shipped — the legacy `reason_for_fail` picklist is superseded as a vocabulary from 2026-08-26; see the Top Failure Reasons banner and [[projects/2026-08-26-bbys-failure-taxonomy]]. Previous: 2026-04-03)
type: context
source: HomeLight-Vault/context/bbys-edge-cases.md
imported: 2026-09-29
---

# BBYS Edge Cases & Failure Modes

> Reference for deal failures, exceptions, stuck apps, and operational issues. Claude reads this for troubleshooting.

## Top Failure Reasons (752 failed leads, Oct 2025 - Feb 2026) — PRE-TAXONOMY

> ⚠️ **Superseded as a vocabulary, still valid as history.** The table below uses the *legacy*
> `reason_for_fail` picklist. On **2026-08-26** Sales App shipped the tiered failure picker
> (6 categories → 13 reasons → 56 sub-reasons) and HubSpot gained five `bbys_failure_*`
> properties. Deals failed from that date carry the new taxonomy; deals failed before it carry
> only the legacy value. **There is currently no single field that reports cleanly across both
> eras** — see the open question in [[projects/2026-08-26-bbys-failure-taxonomy]].


| Reason | Share | Notes |
|--------|-------|-------|
| "BBYS not needed" | 62% | Suggests misqualification at intake |
| Unresponsiveness | ~7.5% | Client or LO goes dark |
| Not enough equity unlock | Significant | Cross-reference with EB/HELOC |
| Loan approval failure | Case-by-case | e.g., additional property blocking loan |
| Client chose competitor | Case-by-case | UpEquity undercutting on fees |
| HOA/title issues | Case-by-case | Late-discovered lawsuits, HOA portal issues |
| Pivoted to DTI Drop | Case-by-case | Cancelled full BBYS, kept DTI component |

~~Mette and Marc aligning SalesApp dispositions with HubSpot failure reasons — Q2 2026 launch.~~
✅ **Shipped 2026-08-26.** Dave Spivey announced it in Slack; Marc supplied the full slug/label
list. Selecting Disposition → "Failed" now shows the tiered picker (left→right: category →
reason → sub-reason). An envelope icon on a sub-reason means choosing it auto-sends an
"Application Declined" email to the LO, quoting the reason and any notes from the modal.
See [[projects/2026-08-26-bbys-failure-taxonomy]].

## Conversion Rate Crisis (Active — March 2026)

Wei identified a 3-point / 15% decline in apps-to-IR-close:
- **Approval-to-Agreement: 30%** (was 41%) — nurture workflow turned off in Feb, plus quality paradox (app-to-approval jumped to 75-80% vs historical 60%)
- **Agreement-to-IRUC: 78%** (was 89%)
- Wei and Gui actively investigating

## Stuck Applications

120 apps stuck in "submitted" status (not progressing to photo submission):
- Worth ~24 closings, ~28 IRUCs at normal conversion
- Technical issues: EB portal submissions failing silently, HELOC upload bugs, builder apps not syncing to HubSpot

## Exception System

Jake Vogel and Jason Smith are primary exception approvers. Philosophy: "the approval matrix is more of a guideline than rules."

### Common Exception Types (from Objections Playbook)
- Variable pricing on $1M+ homes (9/10 approved)
- Funding escrow without IR purchase
- Construction loans (BBYB)
- Pending divorces (50% EU only)
- IR purchases outside USA
- Fee reductions to match competitors (UpEquity, FlyHomes)
- Extension amendments (180/240/300 days)
- Builder-specific pricing overrides

## Extension/Repaper Lifecycle

BBYS agreements have 3 standard extension amendments:
- Extension fee adds 1.2% at each tier (180/240/300 days)
- 30+ automated emails track DR listing progress (Day 0 through Day 160)
- Repaper requires: amendment to RPA + BBYSA, new note, notary scheduling

## EUC (Equity Unlock Calculator) System

Clients who begin BBYS but drop out before completion:
- **Lender Sourced EUC Dropouts** — pipeline 70103322 (deprecated)
- **Agent Sourced EUC Dropouts** — pipeline 80806768 (deprecated)
- BBYS Opportunities - pipeline 725454515 (consolidated lender & agent EUCs)
- Field: `deals_bbys_euc_dropout_type` (Lender Opportunity, Agent Opportunity, Builder, NHC)
- Builder leads were defaulting to "Other EUC Dropout" — integration bug

## Channel-Specific Edge Cases

### Builder Deals
- Attribution requires setting TWO separate fields correctly
- Pre-lead confirmation emails failing
- HubSpot property validation errors on builder-specific fields
- DTI Drop is common for builders (backup offer on departing)

### Lender Deals
- Portal bugs: EB apps not coming through, 1003 uploads failing
- Fee competition with UpEquity ($1,500 cheaper on some deals)
- LOs gaming approvals ("target 100K EU but we only approve for DTI drop")

### Agent-Sourced Deals
- 25% comp discount on agent-sourced deals
- Attribution must be preserved even when LO completes submission later
- Different EUC pipeline than lender-sourced

## Recurring Operational Pain Points

- No dedicated payment system owner — tickets flow into Slack unstructured
- HubSpot/Periscope reporting misalignment frustrating Drew
- Data quality: wrong valuation fields pulled, duplicate leads, missing dates
- EVA document processing needs rigid BBYS-specific workflows
- Notary scheduling: TCs use Penny (not Snapdocs) for DTI Drop

## Related Notes

- [[bbys-overview]]
- [[dti-drop]]
- [[heloc-product]]
- [[q2-priorities]]
