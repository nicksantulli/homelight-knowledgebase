---
last_updated: 2026-08-29
status: current
source: homelight/eave-go @ e06abd3ef0 · homelight/eave-web @ eca5fd3b95 · homelight/hapi @ c14079c76c · Slack #hlhl-processing, #lab-rats-lp-sales, #proj-bbys-heloc-flow (2024-06-24 to 2026-07-31, appended 2026-08-29)
scope: The loan lifecycle as implemented — origination, underwriting, rate lock, funding, warehouse, investor delivery, servicing, payoff
source: HomeLight-Vault/context/lending-mechanics.md
imported: 2026-09-29
---

# Lending mechanics

How a loan actually moves through HomeLight's lending stack, in code. This is the mechanical
counterpart to the business views in [[heloc-product]], [[bbys-lifecycle-operations]],
[[bbys-stage-progression]] and [[bbys-unit-economics]].

## 🔑 There are two lending rails, and only one of them is current

| | **Rail A — first-lien mortgage** | **Rail B — BBYS lien + HELOC** |
| --- | --- | --- |
| System of record | `eave-go` / `trus.io` ([[repo-eave-hlhl]]) | HAPI + **MeridianLink** |
| Products | conforming, high-balance, jumbo, VA, investment | BBYS equity-unlock lien, HELOC Equity Boost, HELOC Card |
| Code status | **frozen at 2023-12-11** | actively developed |
| Underwriting | in-repo `DecisionCriteria` buy boxes + Fannie DU | HLHL `/prequalifications` API + manual UW |
| Docs / closing | DocMagic + DocuSign | MeridianLink EDocs + Snapdocs + Simplifile |
| Capital exit | warehouse → correspondent investor sale | balance sheet / capital partner (not in code) |

> 🔴 **Rail A is the richest lending code HomeLight has ever written, and it is dead.** The
> whole apparatus below — buy boxes, hedging, warehouse tapes, investor delivery, servicing
> transfer — describes a correspondent mortgage bank that HomeLight ran and stopped running.
> Treat it as **historical mechanism**, not current operations, unless a specific piece is
> shown live. Its constants are stale: `ConformingLimit1Unit = 647200` is the **2022** limit.

Rail B is what BBYS and the LO channel touch today. Both rails share the same licensed entity
(HLHL, NMLS **1529229**) and the same `hlhl_contacts` staff roster.

---

# Rail B — what runs today

## 🔑 The HELOC prequalification contract

`HelocLead#prequalify!` computes nothing locally. It calls
`LeadDataService::Clients::HomelightHomeloans` → `POST {HLHL_API_URL}/prequalifications`,
`Authorization: Bearer {HLHL_API_TOKEN}`, `Content-Type: application/json`.
`HLHL_API_URL` defaults to `https://staging.homelighthomeloans.com`.

**Request** (all money in **cents**; `nil` fields are `.compact`ed out):

| Field | Source on the HAPI side |
| --- | --- |
| `external_application_id` | `heloc_lead.external_id` (uuid) |
| `home_value_cents` | `heloc_lead.estimated_home_value` — the **incoming residence** |
| `ir_first_mortgage_balance_cents` | derived: `estimated_home_value × (1 − target_down_payment_percentage)` |
| `dr_loan_payoff_value_cents` | `bbys_lead.loan_payoff_value`, else `lien_balance_1+2+3` |
| `dr_estimated_home_value_cents` | `bbys_lead.heloc_cltv_home_value` |
| `bbys_equity_unlock_cents` | `dp_max_downpayment_equity` ‖ `dp_target_new_home_purchase_equity` |
| `equity_boost_cents` | `bbys_lead.equity_boost_amount_used` |
| `monthly_pitia_cents` | HOA + taxes + HOI + MI, summed in HAPI |
| `monthly_liabilities_cents` | Σ `heloc_lead_liabilities.estimated_monthly_payment` |
| `total_monthly_income_cents` | `client_estimated_income_monthly` |
| `credit_score_range` | e.g. `"720_739"` — falls back to `DEFAULT_CREDIT_SCORE = "700_719"` |
| `state` | `incoming_residence_state_code` ‖ `lead.state_code` |
| `monthly_taxes/insurance/hoa/mortgage_insurance_cents` | individual PITIA components |
| `purchase_loan_rate_percent` | float, e.g. `6.875` |
| `program_fee_percent` | `bbys_lead_fee.estimated_program_fee_percent`, normalized to a percent |
| `bbys_lead_year` | `lead.created_at.year` — **pricing vintage** |
| `borrowers` | `"Jane Doe and John Doe"` (string, not array) |
| `liabilities[]` | `source_liability_id`, `account_type` (canonicalised: `revolving`/`installment`/…), `creditor`, `account_number_last4`, `unpaid_balance_cents`, `monthly_payment_cents`, `paid_off_at_or_before_closing`, `payoff_preference` (`pay_off` \| `auto`) |
| `locked_heloc_amount_cents` | set only when an optimizer scenario was locked |
| `prequal_id`, `locked_optimizer_payoffs` | carried when re-running against an existing prequal |
| `prequal_letter_drive_folder_id` | Google Drive folder HLHL writes the letter into |

**Response**:

```json
{ "prequalification_request_id": "prequal-123",
  "id": "prequal-123",
  "status": "completed",
  "prequal_letter_pdf_url": "…",
  "prequal_letter_google_doc_url": "…",
  "calculations": [ { "partner": "default", "eligible": true,
                      "heloc_range": [125000, 250000] } ] }
```

HAPI post-processes it into `default_partner_eligible` and `default_partner_heloc_max` (the
**first**, i.e. **low**, element of `heloc_range`). Everything is written to `api_logs` with
`api_service = "homelight_homeloans"`.

> 🔑 **`calculations` is an array keyed by `partner`.** HLHL evaluates the same borrower
> against multiple capital partners and returns a per-partner eligibility + line range. HAPI
> only surfaces the one named `"default"`. **The other partners' answers are being computed
> and thrown away** — that is a live pricing/capacity signal nobody is reading.

> ⚠️ **`heloc_range` vs `heloc_range_cents`.** `HelocsController#build_prequal_from_log`
> accepts either key. The client's own parse path only reads `heloc_range`. Unit ambiguity in
> a money field across two readers of the same payload — verify before charting any
> prequal-amount series.

### Scenario updates (the HELOC Optimizer)

A second endpoint, `POST /prequalifications/{id}/scenario-updates`, re-prices an existing
prequal with a locked line amount, chosen liability payoffs, and buydown assumptions
(`buydown_points`, `buydown_cost_cents`, `effective_mortgage_rate`,
`rec_mortgage_amount_cents`). It is driven by an **iframe**: HAPI mints an encrypted
`Heloc::OptimizerToken` (heloc_lead_id + hlhl_prequal_id + lo_user_id + `apply_immediately`,
1-hour expiry) and hands back
`{HLHL_API_URL}/admin/heloc-optimizer?embedded=1&token=…`, rendered inside Sales App
(`HelocOptimizerEmbedDialog.tsx`).

`heloc_optimizer_submissions` are `pending` → `completed`/`cancelled`. Pending ones are
rendered as **synthetic `change_requested` rows** in the prequal history so the LO sees a new
row rather than the old prequal silently mutating.

> ⚠️ **A pending optimizer submission is matched back to its HLHL log by a time window**
> (`applied_at` within `−5 min … +5 s` of the log) **plus an exact cents match**
> (`optimizer_submission_matches_prequal_log?`). Two submissions of the same amount inside
> five minutes will mis-attribute the submitter.

## LOS: MeridianLink

MeridianLink is the **current** loan origination system for both BBYS loans and HELOCs.
`bbys_leads.los_source` is set to the literal string `"MeridianLink"`;
`heloc_leads.loan_number` holds the ML loan number. HAPI's integration lives in
`services/provider_api_service/.../meridian_link/` and is documented in-repo at
`services/provider_api_service/docs/MeridianLinkIntegration.md` and
`MeridianLinkHelocIntegration.md`.

| Surface | Endpoint |
| --- | --- |
| SOAP webservice | `https://webservices.mortgage.meridianlink.com/los/webservice` (`Loan.asmx`, `EDocsService.asmx`) |
| OAuth | `https://secure.mortgage.meridianlink.com/oauth/token` — ticket cached 3 hours |
| Webhook in | `POST /api/provider-api-service/meridian-link/webhooks/loan-files`, `Authorization: Bearer {MERIDIAN_LINK_SECRET}` |

Core operations: `create("HELOC")` (production, from the **`HELOC` template**),
`create_with_options({IsTestFile: true, TemplateNm: "HELOC"})` (non-prod), `save(…)`,
`delete_applicant`, `upload_pdf_document`.

> 🔑 **Environment isolation is by loan-number prefix.** The webhook controller processes
> non-`TEST` loan numbers in production and only `TEST`-prefixed ones elsewhere. A test file
> created without the prefix would be processed as production.

**1003 intake, two paths.** A **PDF** must be classified by Extend.ai as
`processorName: "EU Loan Doc Classifier"` / `initialOutput.type: "1003_loan_application"`;
`LoanFileBuilder::BuildAndSaveLoanFileData` then assembles applicants, borrower info, assets,
REO, declarations, loan + subject property, other income, employments and liabilities into the
ML file, after which **`document_data_extractions.extracted_data` is cleared** so PII does not
persist in HAPI. An **XML** must be MISMO **3.4** (namespace
`http://www.mismo.org/residential/2009/schemas`, `MISMOReferenceModelIdentifier` starting
`3.4.`) and is imported wholesale via `Loan#save(loan_number, content, :xml_mismo_34_file, 12)`.

Credit reports run through `ProcessCreditReportWorker`, which reads `median_score_overall` onto
`HelocLead.estimated_credit_score`. Once **both** the 1003 and the credit report are processed,
`prequalify!` fires. Document upload retries **3× at 5-minute intervals** waiting for
`loan_number` to exist.

> ⚠️ **MeridianLink webhooks do not drive HELOC stages.** They update `BbysLead`/`BbysLoan`
> only. HELOC stage transitions are local model callbacks — if a HELOC stage looks wrong, the
> ML webhook log is the wrong place to look.

Gating flags: `bbys_heloc_phase_1`, `meridianlink-loan-creation-integration-phase_1`/`-phase2`,
Flagsmith `pdf_merge_in_hapi`. A dormant `blend-encompass-poc-enabled-loan-app-ids` flag marks
an **Encompass** PoC that never shipped.

---

# Rail A — the first-lien machine (historical mechanism)

## Origination → the 19 loan statuses

`UnderwritingSummaryLoanStatus` (`eave-go/underwriting/underwriting_summary.go`), collapsed
into six borrower-visible `LoanMilestone`s:

| Milestone | Statuses |
| --- | --- |
| `application_started` | `account_created`, `credit_pull_authorized`, `credit_report_errors` |
| `credit_report_received` | `credit_report_received` |
| `pre_application` | `auth_package_signed`, `document_collection`, `ready_for_uw`, `in_borrower_underwriting` |
| `respa` | `application_complete`, `in_processing_underwriting`, `sign_off_by_uw` |
| `closing` | `closing_package_sent`, `in_closing`, `pre_funding_qc_audit` |
| `post_closing` | `funded`, `post_closing`, `closed_loan_purchased`, `servicer_branding_complete` |

Plus `canceled`, which maps to no milestone.

> 🔑 **`closed_loan_purchased` and `servicer_branding_complete` are post-sale states.** The
> loan lifecycle in this system does not end at funding — it ends when the investor has bought
> it and the servicer has re-branded the borrower relationship.

## The hurdle system — 67 gates

`underwriting/hurdles_consts.go` registers **67** named hurdles; each has a matching editor
under `eave-web/.../underwriter_dashboard/hurdle_editors/`. This is the underwriter's actual
worklist:

| Group | Hurdles |
| --- | --- |
| Identity/credit | PersonalInfo · ContactInfo · EmailConfirmed · CreditReportPull · CreditSummary · Declarations · Fraud · History · SpecialSituations · Vesting · OwnerOccupancy |
| Income/assets | Employment · IncomeSummary · IncomeDocumentsReview · SalariedIncomeAnalysis · TaxReturnIncomeAnalysis · OtherIncomeCashFlowAnalysis · AssetSummary · ResidualIncome · TaxTranscripts (+ match-to-ingested-returns) · VVOE · WVOE |
| Property | AppraisalReport · PropertyType · PropertyCondition · SubjectPropertyReport · FloodCertification · HazardInsurance · Escrows · Title · TitleCommitment |
| Product/pricing | LoanProduct · LoanProductScenario · TransactionTerms · **BorrowerRateLock** · **InvestorRateLock** · Amortization · MortgageInsuranceSummary · Prepaids · Fees · PointsAndCredits |
| Compliance | DisclosureEntities · DisclosureAPRMismatch · DisclosureRegeneration · DisclosureRequestLintResults · DocmagicWarnings · ConflictOfInterest · **CQMBasicInfoReview / CQMMiscInfoReview / CQMDocMagicReview** |
| Close/fund/post | ClosingSummary · ClosingExplanations · Funding · PostClosingDocs · CashCloseSummary |

> 🔑 **"CQM" = Compliance/Quality Management.** Three separate CQM review hurdles gate the
> file before docs go out — the pre-funding QC audit in structural form.

## Underwriting: buy boxes as versioned, dated code

`DecisionCriteria` implementations are named
`Eave<YYYYMMDD activation><Conforming|Jumbo><min-downpayment>`. Thirteen exist; **three were
active** at freeze:

| Criteria | Notes |
| --- | --- |
| `Eave20210924Conforming` | the main conforming box |
| `Eave20190624Jumbo` | jumbo, FICO-tiered 660→740 across ~30 rows |
| `EaveVA` (`Eave20220112VA`) | VA, incl. high-balance |

The conforming box (`decision_criteria_eave_20210924_conforming.go`) — hardcoded constants,
**not** config:

| Occupancy | Units | Max LTV | Max loan | Min FICO | Max DTI |
| --- | --- | --- | --- | --- | --- |
| Primary | 1 | 95% | conforming limit | 620 | 49.9% |
| Primary (**FTHB only**) | 1 | 97% | conforming limit *(absolute — no county uplift)* | 620 | 49.9% |
| Primary | 2 | 85% | 2-unit limit | 620 | 49.9% |
| Primary | 3 | 75% | 3-unit limit | 620 | 49.9% |
| Primary | 4 | 75% | 4-unit limit | 620 | 49.9% |
| Second home | 1 | 90% | conforming limit | 620 | 49.9% |
| Investment | 1 | — | conforming limit | 620 | 49.9% |

`minLoanAmount = $25,000`. `MaxLoanAmountIsAbsolute` on the 97% FTHB row means county
high-balance uplift does **not** apply there. Operator overrides
(`OperatorMaxLTV`, `OperatorMaxDTI`, `OperatorMaxLoanAmount`, `OperatorMinimumLoanAmount`,
`OperatorMaxPurchasePrice`) can widen any box per file — so a booked loan may sit outside the
published box legitimately.

> ⚠️ **The credit floor is 620 on both rails, but the DTI ceilings differ.** Rail A conforming
> caps DTI at **49.9%**; the HELOC product caps at **45%** with a 50% exception ([[heloc-product]]).
> Do not treat "our max DTI" as one number.

AUS is Fannie Mae **Desktop Underwriter** (`handlers/desktop_underwriter.go`), with full
FNMA 3.2, MISMO 2.3.1 and MISMO 3.4 message builders in `eave-go/fnma/`.

## Products, rates, and locks

**Loan types (9):** conforming · high_balance · jumbo · jumbo_low_downpayment · va_conforming ·
va_high_balance · investment_conforming · investment_high_balance · investment_jumbo.
**Loan programs (9):** `30yr fixed` (default) · `15yr fixed` · `5/1`, `7/1`, `10/1 ARM` ·
`5/6`, `7/6`, `10/6 30-Day SOFR ARM` · **`second lien`**.

Each program carries `Margin`, `InitialCapPercent`, `YearlyCapPercent`, `MaxCapPercent`,
`QualificationRateAdjuster`, an `IndexType`, and a `DocmagicProductIdentifier`. FNMA plan
numbers `GEN5`/`GEN7`/`GEN10` map the SOFR ARMs. Rate tables are per-program **per lock period
(30/45/60 days)** and carry `external_investor_id` / `external_program_id` — i.e. rates are
**investor-specific**, pulled from Optimal Blue (`marketplace.optimalblue.com`, business
channel `100078`, originator `1283402`). `InvestorProgramRateCapsAndMargins` stores the
per-investor cap/margin overlay.

> 🔑 **Two independent rate locks exist and they are separate hurdles:**
> `BorrowerRateLockHurdle` (what the borrower is promised) and `InvestorRateLockHurdle` (the
> takeout commitment). `investor_details` records `investor_rate_lock_date` and
> `investor_rate_lock_expiration_date` separately from the borrower's. Pipeline risk lives in
> the gap between them.

**MCT (Mortgage Capital Trading)** is the hedge/lock desk. `ingestion/mct/mct.go` builds a
~90-column loan tape — lock date/expiry, LO price, best-efforts price, sell price, purchase
advice date, commitment number, investor, **`warehouse_bank`**, `mustgo`/`cantgo` investor
constraints, LTV/CLTV/FICO/DTI, AUS type + case file ID, PIW flag, ULI, eNote flag.
Delivered via `POST /api/v1/public/mct_upload`.

## Closing, TRID, and the wire

`EarliestAllowedSigningDate` implements the TRID waiting period exactly: baseline **7 calendar
days** from the preliminary CD being sent; but if **every** borrower has e-signed
(`DocmagicCompleteEsign` count ≥ borrower count) it becomes the **earlier** of that or
**3 CFPB business days** from the last borrower's e-sign. `CDWaitPeriodHurdle` and
`ChangeInCircumstance` (which forces re-disclosure) are first-class models;
`trid_balancing.go` reconciles LE→CD tolerances, and fees carry `Tolerance`, `IncludedInAPR`,
`LESection` and `CDSection` so the disclosure math is derived, not typed. Documents come from
**DocMagic**, signatures from **DocuSign**, fee estimates from **ClosingCorp SmartFees**.

`WireStatus` runs `StatusWireRequested` → `StatusWireApproved` → `StatusFunded`, carrying
`federal_reference_confirmation_number` (the Fed IMAD), `wire_instructions_received`,
`settlement_agent_receipt`, `receipt_confirmed_by` and `authorized_by_id`. `WireDeduction`
items reduce the wire except those flagged `PaidOutsideClosing`.

## Warehouse

Loans are table-funded on warehouse lines and delivered as **tapes** — CSV files generated by
the operator, written to S3 and recorded as documents:

| Document type | Warehouse |
| --- | --- |
| `bank_united_warehouse_funding_tape` | **BankUnited** (`tapes/bankunited/`) |
| `origin_warehouse_tape` | **Origin Bank** (`underwriting/origin/`) |
| — (referenced in the MCT tape) | **Texas Capital Bank** |

`investor_details.warehouse_line` names the line per loan; `ContactRoleWarehouseLender` is a
first-class contact role, and MISMO `PartyRoleWarehouseLender` carries it into the disclosure.

Warehouse principal is **not** the note amount — `conformingWareHousePrincipal(loanAmount,
gainOnSale)` and `jumboWareHousePrincipal(...)` derive it from gain-on-sale, so the line
advances less than the note on a premium-priced loan.

Generating the Origin tape has a side effect: it stamps `UpsertWireAuthorizedBy(loan, operator)`
— **producing the warehouse tape is the act that authorizes the wire.**

> 🔑 **BBYS uses a different warehouse.** [[bbys-stage-progression]] records a **JPM warehouse
> line** on BBYS funding artifacts. BankUnited/Origin/Texas Capital are Rail A only.

## Investor delivery and gain on sale

`InvestorName` (14): BankUnited · Bayview · Caliber · Chase · **Eave** (retained) · Flagstar ·
FranklinAmerican · Lakeview *(legacy — now Bayview)* · **MandatoryPipeline** · **MaxEx** ·
NexBank · PennyMac · PHHMortgage · USBank.

**MaxEx** is an exchange, so it has nine sub-investors: A1 Arch Mortgage Funding · A2 Bank of
America · A3 Citi · A4 Goldman Sachs · A5 JP Morgan · A6 Morgan Stanley · A7 Northpointe Bank ·
A8 Onslow Bay Financial (Annaly) · A9 Principal Bank.

Economics (`InvestorDetail.Calculate`) — gain is quoted as **price**, revenue derived:

```
expected_revenue = loan_amount × (expected_gain − 100) / 100
actual_revenue   = loan_amount × (actual_gain   − 100) / 100
```

`GainOnSale()` prefers `actual_gain`, falling back to `expected_gain`. Also tracked:
`investor_loan_number`, `investor_purchase_date`, `loan_service_transfer_date`.
The `summary_report` extends this with `PrepaidInterestByBorrower`,
`SecondaryMarketAccruedInterest`, `SalesAdjustmentsAndOrPenalties`, `InterestIncome`,
`OriginationProceeds`, `Curtailments`, `HedgingGains` — a full loan-level P&L.

> ⚠️ **`MandatoryPipeline` is an investor value meaning "not yet allocated".** Counting it as a
> counterparty overstates investor concentration.

## Servicing and transfer

`InvestorToServicer(investor, sub, state)` resolves the subservicer, and it is **state-dependent
for Lakeview** — a 19-state West Coast list vs everything else, both landing on **LoanCare, LLC**
but with different remittance addresses. Servicers in the map: LoanCare · Select Portfolio
Servicing · Shellpoint · Fay Servicing · Specialized Loan Servicing · Northpointe Bank · Cenlar
(Central Loan Administration & Reporting) · Carrington · **Nationstar d/b/a Mr. Cooper** ·
Caliber · JPMorgan Chase · Flagstar · Citizens One · NexBank · PennyMac Loan Services ·
PHH Mortgage · U.S. Bank.

Each entry carries the **goodbye-letter** addresses (new servicer, overnight, client care
center hours) and the **HOI transfer-of-servicing / ISAOA mortgagee address** — i.e. the RESPA
servicing-transfer notice is generated from this table.

Interim servicing is modelled in `ServicingPayment` (check date/amount/document, coupon splits
for additional principal, additional escrow, late charge, fees, recoverable advances) and
`ServicingActualEscrow` (actual tax / insurance / MI / disbursement per period).
`SortedServicingPayments.CheckForPaymentPeriodGap` hard-errors on a missing month — HLHL
serviced loans in-house between funding and transfer.

## Compliance surfaces

| Surface | Implementation |
| --- | --- |
| **NMLS** | `EaveNMLSLicense = 1529229` on every MISMO/FNMA message; per-LO `hlhl_contacts.nmls_id` + `nmls_licensed_states` |
| **HMDA** | `db/hmda.go`, `underwriting/hmda.go`; ULI built from `LEI = 143800CDBLFYF82X274` |
| **TRID / RESPA** | CD wait period, change-in-circumstance re-disclosure, `trid_balancing`, fee tolerance levels |
| **ECOA / adverse action** | `adverse_action_notices` (`reasons` string array), `AdverseActionNotices.tsx` in the borrower app, mailed physically via **Lob** |
| **E-consent** | `homelighthomeloans.com/electronic-consent-agreement` + `/privacy-policy`, linked at account creation from homes-fe |
| **Soft-credit consent** | `eave-web` `soft-credit-terms-and-conditions` page |
| **Right of rescission** | 3-day, on HELOCs secured by a primary residence ([[heloc-product]]) |
| **Conflict of interest** | its own underwriting hurdle |
| **Fraud** | its own hurdle; `Accurate Group` callbacks |
| **State eligibility** | Rail B gates on `MarketplaceProgram.enabled_for_state_id?("heloc", …)`; Rail A on `states_and_territories.go` |

## How BBYS connects

- HAPI models HLHL as a **lender row**: `Lender::EAVE = "eave"`, and
  `lender_leads.eave_loan_application_id` is the join key into `eave-go`.
- `LenderViewModel::LENDER_PRIORITIES = ['eave', 'better', 'lenda']` — **"Eave is preferred"**
  (comment in code). HLHL sits at the top of lender ranking.
- Cash Close: eave-go pulls the HomeLight order
  (`GET /order-data-service/home-loans/orders/{id}`) and classifies it into eight
  `CashCloseType`s (`trade_in`, `cash_offer`, `cash_offer_express`, `trade_in_cash_offer`,
  `trade_in_plus`, `trade_in_plus_cash_offer`, `none`, `error`), carrying DR guaranteed sale
  price, cash-close fees, ownership expenses, transfer tax and listing-prep fee.
- Reverse: eave-go pushes mortgage leads via `POST /lead-data-service/home-loans/mortgages`
  and 13 `HLStage` values.
- Full integration inventory: [[bbys-integration-map]].

## 2026-08-29 — from the Data Bridge knowledge base (KB, source: `knowledge_documents`)

New DTI Drop / lien / VA facts from the compiled Data Bridge KB (see [[data-bridge-kb-index]] for
the full corpus). Stated (compiled ops answers), not code-verified here.

- 🔑 **DTI Drop can auto-select without operator input.** Add a confirmation step before
  accepting a DTI-Drop-flagged submission: does the borrower actually need equity access
  (→ Equity Unlock) or is this a purchase-before-sale bridge scenario (→ bridge loan)? DTI Drop
  and Equity Unlock follow different review paths and misrouting causes rework.
  ("DTI Drop Intake: Confirmation Step for Auto-Selected Submissions")
- ⚠️ **New York: the 0% bridge-loan / equity-unlock path is not offered at all.** DTI Drop *may*
  still be offered on an NY subject property if the borrower has sufficient down-payment funds
  and needs the departing-home PITIA removed from qualifying DTI. Don't conflate the two paths
  for NY files. ("New York: Bridge Loan Restrictions vs. DTI Drop Eligibility")
- **VA scenarios split into two distinct use cases** that get different treatment: (1) BBYS used
  for DTI/VA-entitlement relief (extra complexity, evaluate carefully), vs. (2) BBYS used only
  for equity access with no DTI/entitlement need — a VA departing lien + VA purchase loan in
  case (2) is **not** an automatic decline. ("VA BBYS Scenarios: DTI/Entitlement Relief vs.
  Equity Access")
- **DTI-omission ≠ payment waiver.** Leaving the departing-home PITIA out of qualifying DTI is an
  underwriting accommodation only — the borrower still owes the payments on that mortgage until
  the DH sells. Unused bridge proceeds / cash overage at closing may be disbursed to the borrower
  to help cover it (title/CD handling required). ("BBYS / DTI Drop: Borrower Payment Obligations
  and Cash Overage at Closing")
- 🔑 **DTI Drop fee wording trap:** contracts phrased "greater of 1% of final sale price **and**
  $5,000" are a floor/minimum-fee comparison (take whichever is larger), not additive — LOs
  sometimes read "and" as "+". ("DTI Drop Fee: Contract Wording FAQ")
- **Second-lien payoff presentation gap:** when HomeLight pays off a first mortgage + a
  HELOC/second lien at IR closing, ECON tracks them as separate line items but the BBYS agreement
  document may show one combined payoff figure. This is a document-presentation limitation, not
  data loss — cross-reference ECON if a client or LO questions the combined number.
  ("Second-Lien Payoff Presentation in BBYS Agreements")
- **DTI Drop mortgage-paydown workflow:** set the estimate calculator's lien balance to the
  post-paydown figure (Equity Unlock should then show $0), add Finance-addendum language via the
  worksheet, and log a Sales App exception note. Equity Boost may cover only part of the required
  paydown — the remainder is the client's out-of-pocket responsibility.
  ("DTI Drop: Mortgage Paydown Operator Workflow")
- **Payoff-info release requires signed borrower/seller authorization** before HomeLight will
  share payoff figures with a title company — route the title company into the payoff
  department's email chain to supply it. ("Payoff Information Release: Borrower Authorization
  and Title Company Coordination")

### 2026-08-29 addendum (LOOP-2 KB sweep — remaining ~290 title-only rows read via the compiler's
own `summary` column, not full content_text; see [[data-bridge-kb-index]] for method)

- 🔴 **FHA-to-FHA is a hard boundary, not an edge case:** when BOTH the departing-home mortgage
  and the incoming purchase loan are FHA, operators must NOT promise the DTI-drop/liability-
  omission benefit — it doesn't apply. The file may still qualify for Equity Unlock support on
  its own merits. ("FHA-to-FHA Loans: DTI Drop / Liability Omission Boundary")
- 🔑 **DTI Drop's liability offset is scoped to ONE property** — the departing primary residence
  designated as the primary/sale property in the file. Don't extend the same relief to other
  properties the borrower wants to sell; route those through equity-unlock proceeds or rental-
  income DTI treatment instead. ("DTI Drop: Property/Liability Boundary and Eligibility Guidance")
- **DTI Drop cancellation/extension fee mechanics:** fee-free cancellation if the borrower sells
  the departing home before closing without HomeLight's help. After closing with HomeLight
  support, standard expectation is sale within 120 days; extensions beyond that carry extension
  fees, and payoff-only scenarios are handled as change-of-circumstance exceptions.
  ("DTI Drop Fee Guidance: Pre-Close Cancellation, Extensions, and Payoff Boundaries")
- ⚠️ **BBYS cannot free up VA entitlement**, even though it removes the home-sale contingency and
  guarantees the departing-home sale — the departing home is not purchased *before* the new home
  closes, so entitlement stays tied up until the DH actually sells. LOs relying on BBYS to restore
  VA entitlement for the new purchase need this corrected explicitly. ("BBYS and VA Entitlement:
  Key Limitation for LOs") — distinct from the eligibility question covered below.
- **VA/government-loan eligibility is not one rule:** an existing VA mortgage on the departing
  residence does NOT disqualify a client from BBYS bridge support, but the DTI-drop mechanic
  specifically requires the *incoming* purchase loan to be conventional. Two different gates —
  don't conflate them when advising an LO. ("BBYS Eligibility: VA/Government Loans and DTI-Drop
  Mechanics")
- 🔴 **Texas land purchases can be fully ineligible, not just CLTV-capped**, due to Texas
  50(a)(6) homestead-equity-loan constraints — a stricter bar than the general Texas 50A6 CLTV cap
  documented in [[bbys-buy-box-and-eligibility]]. Escalate before presenting as viable.
  ("Texas BBYS: Land Purchase Scenarios and 50(a)(6) Eligibility")
- **Out-of-footprint state routing:** before declining a BBYS scenario for a state outside
  HomeLight's lending footprint, separate the two asks — if the client doesn't need HomeLight-
  funded equity and the file otherwise qualifies for DTI Drop, route it as a DTI-only review
  rather than a flat decline. ("Handling Out-of-Footprint States: Separating Equity Access from
  DTI/Contingency Relief")
- 🔑 **Three BBYS-approval concepts LOs commonly conflate:** the opinion of value is distinct from
  the HomeLight purchase price; the fallback/backup purchase amount is tied to the Loan Payoff
  Value, not the opinion of value; and BBYS funds are wired to **title**, never directly to the
  client. ("BBYS Approval: Value, Funding Flow, and Common FAQs")
- 🔑 **DTI Drop-only pricing (1% of final sale price, $5,000 min) and full BBYS bridge-loan
  pricing (2.4%, $9,000 min) must be quoted and documented separately** — confirm which product is
  actually in use before quoting a fee. Matches the numbers already in
  [[bbys-buy-box-and-eligibility]]'s pricing section; flagged here as independent corroboration,
  not a new number. ("DTI Drop vs. Full BBYS Bridge Loan: Fee Structure Guide")
- ⚠️ **No 21-day rescission period exists on the BBYS bridge loan** — final docs are signed 2–3
  days before closing, not held open for a statutory rescission window some LOs assume applies.
  Maintenance reserve is scoped to major issues affecting sale value, not cosmetic repairs.
  ("BBYS Program: Rescission Period & Maintenance Reserve Guidance")

## Open questions

- **Is Rail A still originating anything?** No commit since 2023-12-11 and a 2022 conforming
  limit strongly suggest no new first-lien production. If true, HomeLight's "we are a real
  lender" position rests entirely on the BBYS lien + HELOC, not on mortgages.
- Which capital partners appear in `calculations[].partner` besides `"default"`, and who
  decides the ordering? HLHL is running a multi-partner eligibility engine that HAPI ignores.
- What backs the BBYS lien and the HELOC on the balance sheet? No warehouse or investor
  plumbing for Rail B exists in any repo read here — it is presumably contractual, not coded.
- Who services the HELOC today? [[heloc-product]] says HLHL services in-house and is "actively
  working on a subservicer partnership"; the Rail A `InvestorToServicer` table is the obvious
  place to look for that counterparty but is mortgage-only.
- Is `blend-encompass-poc-enabled-loan-app-ids` dead, or is an Encompass migration pending?
- Where do BBYS payoff/reconveyance mechanics live in code? [[bbys-stage-progression]] shows
  Simplifile + MeridianLink close-out operationally; `WriteHLPaidOffDate` and
  `FetchReconveyanceLoanFile` in HAPI's MeridianLink interactors are the code entry points but
  were not read in depth here.
- **Who owns NMLS #1529229 compliance? 2026-08-29 partial answer (Slack, no single owner
  found):** no message names a formal compliance/registration owner. What Slack does show: (1)
  `#hlhl-processing`, Derek Lupien, 2024-06-24 — last full state-licensed list found for general
  HLHL lending: MS, AR, AZ, CA, CO, CT, DC, FL, GA, IA, IN, KS, KY, MN, OK, OR, PA, SC, AL, TN,
  TX, ID, WA, SD, WY (25 states; likely stale, 2+ years old). (2) `#lab-rats-lp-sales`, Marc
  Kaplan, 2026-07-31 — a **separate, narrower HELOC *servicing*-license list**: FL, MI, MT are
  currently blocked for HELOC specifically (general lending license ≠ servicing license); the
  optimizer silently declines to issue a prequal for a blocked purchase-state rather than
  erroring, per Marc Kaplan. (3) Marc Kaplan currently drives HELOC state-gating
  *operationally* — the monthly `#lab-rats-lp-sales` state-list update, and the `hlhl_lead`
  new-home-state licensing-gate discussion in `#proj-bbys-heloc-flow` (2026-03-16) — but nothing
  found indicates he owns *compliance* (legal registration/renewal) rather than *product
  gating*. Compliance ownership for entity #1529229 itself remains unresolved. See also
  [[repo-eave-hlhl]] 2026-08-29 update.
