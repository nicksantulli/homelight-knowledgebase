---
last_updated: 2026-09-11 (+Elite cadence ops-vs-code clarification)
status: current
source: Payments Ops agent deep dig 2026-09-11 — Slack (#homes-payment-requests, #trolley-collab-and-notifications, #ongoing-payments-requests, #ent-21st-century, Sarah DM D04LLU0URSR), Gmail (bbyspayments / Fairway / Initiate Payments), vault [[sales-ops-operating-model]] §4 + KB addendum, code (homelight/hapi DisbursementService, sales-app PaymentOperationsPage, hl-fe-monorepo equity-app Trolley widget)
scope: living Payments Ops bible — LO/partner money movement, payment plans, Trolley, redirects, one-offs. Companion to [[sales-ops-operating-model]] §4 (does not replace it). Bill.com vendor/AP and Expensify receipts are out of scope.
owner_agent: Payments Ops (draft-only; never send Slack/Gmail, never approve Bill.com, never move money without Nick’s explicit confirmation)
source: HomeLight-Vault/context/payments-ops-bible.md
imported: 2026-09-29
---

# Payments Ops bible

How HomeLight Homes actually pays LOs and partner companies for BBYS (and related) referral fees — systems, people, plan model, code path, cadence, edge cases, and open loops as of 2026-09-11.

Related: [[sales-ops-operating-model]] §4 · [[lo-elite-program]] · [[comp-plans]] · [[dti-drop]] · map file on Payments Ops box `/workspace/payments-ops-lo-partner-map.md`

---

## 1. Systems of record

| System | Role |
| --- | --- |
| **Trolley** (`dashboard.trolley.com`, `homelight.portal.trolley.com`) | Recipient profiles (W9/ACH), invoices, batches, initiate/approve ACH |
| **Sales App** `/payment-operations` | Payment plans CRUD, Manual Payments, Admin Actions (assign/remove plan, offline checks), DBAs Admin |
| **HAPI DisbursementService** | Source of truth for plans, payments, Trolley client, Mon scheduled jobs |
| **LO / equity portal** (`equity.homelight.com/portal/payments`) | LO payment status + Trolley widget |
| **bbyspayments@homelight.com** | Shared inbox for LO/partner payout questions; “Initiate Payments…” mail |
| **#homes-payment-requests** `C0962C92RGQ` | Front door — is it paid?, attach/remove plan, Trolley setup, redirects, exceptions |
| **#trolley-collab-and-notifications** `C08HPJQKSTS` | “Good to approve” → Tara initiates; funding; Trolley support |
| **#ongoing-payments-requests** `C090ME95N3X` | Eng — auto-gen misses, wrong plan attach, profile report bots |
| **Partner `#ent-*`** (e.g. `#ent-21st-century`) | Plan questionnaire + activation |
| **Partner payment details dashboard** | Company-facing deal list keyed to **SA LO** (Fairway Debbie Dorn “refresh”) |
| **Bill.com** | Vendor/AP + TLS AE payouts — **not** BBYS LO referral Trolley |
| **Trackers** | Sarah payment-plan sheet; JE LO referral recon (Sarah Lim); Periscope plan dashboards |

**Not the payout engine:** Data-Bridge. No dedicated PRMI/Fairway “payroll dashboard” code in hapi/sales-app — partner reporting lives outside DisbursementService.

---

## 2. People & lanes

| Person | Lane |
| --- | --- |
| **Sarah Jaka** `U01N0R27GLC` | Day-to-day payments ops (plans, Trolley profiles, batch review, bbyspayments replies, questionnaire, tracker). **Not** Sarah Lim unless Trolley eng. |
| **Nick Santulli (Tulli)** | Policy (redirect, company vs LO, one-offs), funding cosign, 2nd-eye big batches, partner email when escalated |
| **Tara Edwards** | Fund Trolley, initiate approved batches, Bill.com/HLHL AP, bank/tax snags |
| **Kim Tanner** | Partner commercial terms (21st Century, Fairway/PRMI retail notes) |
| **Jake Vogel** | Exception authority (cash-deal / one-off pay) |
| **Brian Banes** | Field AE (often CC’d); not rail owner |
| **Eng** (Taylor Wong, Dave Spivey, Joel, Alex Kwan; Zahedi sabbatical) | Auto-gen, Payment Ops, batch membership |
| **Sarah Lim / Avery** | Accounting recon (JE LO referral fees vs Trolley) — separate from Sarah Jaka |
| **Revenue agent** | Flags pay-board vs warehouse mismatches only — does **not** draft disbursements |

**Payments Ops agent hard rules:** draft only. Never send Slack/Gmail. Never approve Bill.com. Never submit Trolley / move money / change recipient without Nick’s explicit confirmation in chat. Never invent amounts. Skip Trolley/Bill.com connectors unless Nick asks; work Slack + Gmail + vault.

---

## 3. End-to-end LO / partner payout flow

1. **Commercial** — DocuSign Services & Partnership Agreement → Sarah’s **13-field questionnaire** → plan created/activated in Sales App Payment Ops.
2. **Lead attachment** — Auto by lender/partner/source_partner eligibility; else Admin assign. Plan **start/eligibility date** must allow the lead (ops: cannot backdate start — creation day wins). Code skips auto-gen when rules fail.
3. **Earn** — Ops often says **DRX**; code has **no `DRX` string** — stages are `ir_closed` | `dr_closed`. UI default for new plans is **`ir_closed`** (can pay at IR closed if cadence/rules pass). Confirm per plan. Exceptions (cash) may need Jake + one-off.
4. **Recipient ready** — LO completes Trolley profile. No plan → often no profile invitation path. Incomplete → miss cutoff.
5. **Batch build** — Weekly and/or monthly by plan cadence. Sarah reviews; Nick 2nd-eyes heavy months.
6. **Fund** — If balance low, Sarah pings Tara (+ Nick for vis) in funding GDM. Credits vary (~$37k–$68.8k observed; Sep 2026 monthly need ~$102k–$106.7k).
7. **Approve / initiate** — Sarah “good to approve N batches” → Tara Done. “Initiate Payments to LO from Trolley” emails → bbyspayments@.
8. **Pay + confirm** — ACH to LO or company; Sarah confirms in `#homes-payment-requests`.

**Cadence layers (don’t conflate):**
- **Code jobs:** Mon 6am PT `ProcessScheduledPaymentsWorker`; weekly plan = Monday; monthly = first Monday; daily batch sync; Thu inactive payee check; Fri AP CSV.
- **Ops rhythm:** new profiles reviewed **Tue** → pending pays next **Thu**; company dashboard refresh often Monday.

---

## 4. Payment plan model

**Concept:** company-level commercial config (not a single-deal voucher).

### Sarah questionnaire ↔ Sales App / HAPI fields

| # | Ops field | Code / UI |
| --- | --- | --- |
| 1 | Standardized company name | `name` → slug |
| 2 | Trolley recipient ID / POC | `payment_recipient` (required for Corporate unless offline) |
| 3 | Plan start date | eligibility / start (cannot backdate) |
| 4–5 | Agreement type / Who HL Pays | `payout_target`: `LendingCompany` (Corporate) vs `LoanOfficer` (Direct) |
| 6–9 | Flat rate, total/lead, end recipient, LO cut | `payment_plan_disbursements` (≤4 rows): payee_type + flat/%/min/max/`borrower_paid` |
| 10 | Cadence | `weekly` \| `monthly` |
| 11 | Eligibility threshold | `payout_rules` / thresholds (often N/A) |
| 12 | Cohort stage | often N/A |
| 13 | Earned & paid stage | `payout_stage` (ops say DRX → code `dr_closed`; UI default often `ir_closed`) |

Also: `product` `bbys` \| `dti_drop` \| `both`; `add_on_enabled`; lender `owners[]` + fallback partner; `offline_check`.

**Owners (Lender/Partner) ≠ payee.** Payee is `payout_target` + disbursement lines.

**ALLOWED_PAYEES:** `LoanOfficer` \| `Branch` \| `AccountExecutive` \| `LendingCompany`.

**Common commercial:** flat **$1,500**/lead. Company-pay (Fairway, Orchard) vs direct-to-LO (21st Century weekly activated 2026-09-10). LO-direct → **no company Trolley profile**. Paper checks not offered (offline check path is separate).

**Auto-assign order** (`AssignPaymentPlanToLead`): LO’s lender plan → lender’s partner plan → BBYS `source_partner` plan. Product from CRO flag → `bbys` vs `dti_drop`. Skips if already in Trolley/paid.

---

## 5. Code path (SA LO → Trolley money)

Canonical: `homelight/hapi` DisbursementService ↔ `homelight/sales-app` `PaymentOperationsPage` ↔ `homelight/hl-fe-monorepo` equity-app Trolley widget.

**Double hop (why redirects matter):**
1. Sales App sets `loan_officer_id` on lead (**User**).
2. HAPI: `lead.loan_officer&.loan_officer` → **LoanOfficer** record.
3. DIRECT → that LO’s `payment_recipient` (Trolley). CORPORATE → plan’s `payment_recipient`.

**Amount calc** (`CreatePaymentForLead`): flat and/or percent → dollars; percent + LO add-on vs **departing residence final sale price**. Retail company payout: add-on may ride company line (provisional in code).

**Manual Payments:** Ad-hoc \| Exception · Regular \| Rollover · `recipient_source`: `loan_officer` \| `payment_plan` \| raw `trolley`.

**Separate rails (not BBYS plan path):** Elite/referral — see §11 and [[lo-elite-program]].

**Flagsmith mid-cutover:** `dynamic-pricing-payment-plan-disbursements-read` (JSONB `payout_details` vs table); `dynamic-pricing-partner-comp-ui`.

---

## 6. Redirect / wrong-LO policy

**Stated (Sarah 2026-09-10, `#homes-payment-requests` `1789072468`):** 2nd request for company to disburse to an LO other than SA LO. SA must be updated **before** these payments. Company can reach their own payroll; dashboard shows SA LO.

**Vault KB:** redirect = **2-step** (BBYS Payments auth + confirm listed LO).

**Working policy:**
1. SA LO is source of truth for HL attribution/dashboard.
2. Do not silently re-point Trolley payee for internal company commission splits.
3. If payee must change → update SA first, then pay / loop partner accounting for already-sent funds.
4. Internal reallocation = partner payroll’s problem.

**Open decision (2026-09-11):** PRMI $1,000 Shannon Clark → Ian Perry (Keith Anthony / 168 Union St Guilford CT). Gmail thread `1a08cb5c52bc0096`, draft `r416438614834882829`, Slack thread `1789072084.272929` / draft `Dr0C19G2PS1Y`. Nick **held** send ~12:00 MT.

---

## 7. Edge cases & pain patterns

| Edge | Note |
| --- | --- |
| Trolley profile backlog | Sep 9 report ~631 incomplete LOs / 732 leads; Unassigned ~half; all-time failed counter climbs |
| Wrong plan attach | Keys off **Lender Company** field (TLS↔X2, NEXA↔TLS) |
| Auto-gen misses | Plan eligibility vs submit date; Orchard monthly under-gen (Jul+Aug); C2 `14807325`, Lifestone `15144802` |
| Franchise Motto | Company plan exists ≠ auto company pay — BM decides |
| One-off / Jake | Manual Payment Ops; e.g. cash-deal $1,500 without full plan |
| Marketing prizes (SoS) | Prefer company pay; no ad-hoc 5-way splits; document LO vs company per winner |
| Incomplete profile pay | Deleted next day → Monday rollover batch |
| Bulk company invoice tool | Flaky (Barrett 5→1) |
| Fairway $8,250 “mystery” | Not Trolley LO batch — alternate HL system (thread `1a0871d98f2e7533`) |
| Backup withholding | Check **24%** default pre-activation |
| Company plan prerequisite | Signed partnership agreement first; backpay OK after (VP/Nick F. for pre-plan deals) |

---

## 8. Open loops (as of 2026-09-11)

1. **PRMI $1k → Ian Perry** — Nick yes/no held; drafts ready.
2. **Fairway Debbie $8,250** — confirm non-HL source closed (`1a0871d98f2e7533`).
3. **Orchard Aug 3/12** + C2/Lifestone auto-gen — eng (Taylor/Dave).
4. **Loan Depot** Services Agreement / 13-field answers — Tejas/Sarah open.
5. **~631 incomplete Trolley profiles** — structural.
6. Self-tracker “Debbie SoS reply” may be stale vs closed Gmail `1a017485b3ab2039`.
7. Bill.com **Opozee** device-verify — vendor AP, not LO Trolley.
8. Motto Secure company vs direct-LO path (Brenda Pope / lead `15269756`) — incomplete profile.

---

## 9. Ops cheat sheet

Requests → `#homes-payment-requests` → Sarah owns Trolley/plan/email → Nick owns policy + big-batch/funding cosign with Tara → eng in `#ongoing-payments-requests` when auto-gen fails → Bill.com is a separate AP lane → Revenue agent flags mismatches only.

**When drafting for Nick:** cite thread IDs, amounts, payees, dates from source. Speak as Nick only in outbound drafts. Counterpart for payments ops is **Sarah Jaka**.

---

## 10. Key citations (index)

Slack: `C0962C92RGQ` `1789072084` (PRMI redirect); `1788969750` (cash one-off); `C08HPJQKSTS` `1789060146` (batch approve); `C090ME95N3X` (Orchard/auto-gen); `C0AB5Q6KR51` `1775578073` (21st Century); `D04LLU0URSR` (Sarah DM batches); `C0C0V9VTATW` (Motto).

Gmail: `1a08cb5c52bc0096` (PRMI); `1a0871d98f2e7533` (Fairway $8250); `1a0457bbac0adc50` (Edge cadence); `1a017485b3ab2039` (Fairway SoS closed).

Code: `DisbursementService::PaymentPlan`, `AssignPaymentPlanToLead`, `CreatePaymentForLead`, `ProcessScheduledPaymentsWorker`, `PaymentOperationsPage.tsx`, `TrolleyClient`.

---

## 11. Elite / referral + partner-comp + offline_check

*(Code layer from Codebase Knowledge Bot, 2026-09-11 — `/workspace/payments-ops-lo-partner-map.md` Layer 2)*

### Elite / referral (separate from BBYS plan batches)

- **Flag:** Flagsmith `lo-elite-program`
- **Trigger:** BBYS disposition → `TriggerReferralPayment` → `HandleReferralPaymentWorker`
- **Who:** Invitee = lead’s LO; inviter = `invitee.inviter` (skip if blank)
- **Eligible if:** Fairway source partner **or** inviter `elite_member?`
- **Amount:** Fairway + non-elite inviter → **$1,000**; else **$1,500**
- **Stage gate:** hard **`dr_closed`** (cancels referral payment if not DR closed) — independent of plan `payout_stage`
- **Uniqueness:** one referral payment per inviter↔invitee pair
- **Create:** `CreateReferralPaymentForLoanOfficer` → `payment_source=elite_program`, `category=referral`; company vs LO follows lead plan (`pays_to_company?`)
- **Batch:** Mon `ProcessEliteProgramPayments` → batch name `"Referral Program Payments"`; regular plan path explicitly excludes elite (`not_elite_program_payments`)

### Partner-comp fee sync (on plan assign/clear)

- Soft-fail interactor `SyncPartnerCompensationOnPaymentPlanChange` → LeadDataService
- Rewrites on `BbysLeadFee`: `borrower_paid_plan_total_amount`, `total_customer_program_fee_amount`
- Formula (`CalculatePartnerCompensation`): HL base + **borrower_paid** disbursement lines (table, not JSON) + LO add-on ± discounts/surcharges → **cap $15,000**
- Does **not** mutate HL estimated/actual program fee columns

### `payout_stage` vs ops “DRX”

| Surface | Behavior |
| --- | --- |
| UI default | `ir_closed` |
| UI options | `ir_closed` \| `dr_closed` only |
| Gate | `Lead#reached_payout_stage?` (+ cadence + `payout_rules`); also blocks if lead is **past** DR closed |
| Ops “DRX” | No `DRX` string in disbursement code → map to **`dr_closed`** |

### Offline / check plans

1. Plan `offline_check` → payment created `OFFLINE_PENDING`, **no** Trolley recipient
2. Excluded from Trolley batches
3. Mon Slack CSV to `#ongoing-payments-requests` listing pending offline checks
4. Mark paid in Payment Ops Admin Actions → `PayOfflineCheckPayments`

### Elite ops reality (Slack/Gmail, 2026-09-11)

**Owners:** Sarah day-to-day Elite referral + plan payouts; **Taylor Wong** runs/creates Elite payments; Zahedi backfills inviter links / Referral tagging; Tara = 2nd Trolley approver / Bill.com — not Elite product owner.

**Amounts & cadence (ops speak):**
- Elite invite bonus: **$1,500** to referring Elite LO when invitee closes IR (ops); payee = referring LO **personal** Trolley (not company)
- Fairway invite-a-colleague legacy: **$1,000**, historically manual Trolley → Fairway corporate; later Referral tag on payments
- Standard $1500 plan “referral credit”: after DR closes; brokers ~2 weeks; retail/bankers monthly via payroll; Ashlee: **every Thursday we reconcile** → funds ~following week; Sarah aims mid-day Thu approve so Tara can second-approve
- **10bps cash** (rare): wait DR Closed, often manual Trolley; can top up when plan already paid $1500 (Payment App = one payment/lead → second payment goes straight to Trolley)

**Stacking:** Elite inviter bonus is separate from the invitee deal’s plan payment. Partner-comp UI is fee/pricing (Flagsmith + `add_on_enabled`), not Elite inviter payout.

**Open Elite/referral pain:**
1. Missed invite link → no auto referral; needs Nick/Zahedi + Taylor manual
2. Incomplete Trolley blocks Elite and plan pay
3. Payment App one-payment-per-lead → Elite/10bps/second pay must bypass to Trolley
4. 10bps cash plan rules still unfinished (Zahedi blocked Jun 2026)
5. Wrong bank can permanently lose $1500 (Wendy) — re-pay needs exec exception
6. Partner JE vs Trolley DR Closed mismatches (Sarah ↔ Sarah Lim)
7. Fairway invite-a-colleague still thin/v0 historically

Elite discussion concentrated in `#proj-elite-lender-launch` + `#homes-payment-requests` / `#ongoing-payments-requests`. Most Gmail “referral fee” = standard $1500 plan, not Elite invite bonus.

### Cadence: ops Thu vs code Monday (important)

Vault/Ashlee “DH close → Thu reconcile → pay following week” is **partner-facing ops language**, **not** a coded Elite pipeline.

| What | Reality |
| --- | --- |
| Elite create gate | Hard **`dr_closed`** on disposition |
| Elite Trolley push | Mon `ProcessEliteProgramPayments` via `ProcessScheduledPaymentsWorker` (`0 13 * * 1` UTC = Mon 6am PT) — batch `"Referral Program Payments"` |
| Thu jobs in code | Inactive payee check + Lower special $500 — **not** Elite referral reconcile |
| vs BBYS plans | **Stack, don’t exclude** — `lead.payment` hides elite; separate batches. Plan can pay at `ir_closed`; referral waits for `dr_closed`. Soft couple: referral payee can follow corporate plan |

Partner-comp UI: sales-app `PricingEngineModal` (flag `dynamic-pricing-partner-comp-ui`) shows LO add-on + borrower-paid total; edits BP discount. Same calc merged in `ApplyPricingEngine` / `SavePricingFees`.

