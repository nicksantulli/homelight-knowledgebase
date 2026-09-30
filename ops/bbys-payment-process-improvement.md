<!-- source: HomeLight-Vault/projects/bbys-payment-process-improvement.md | imported: 2026-09-29 -->

# BBYS Loan Officer Payment — Process Improvement Analysis

> Related: [[bbys-overview]] · [[homelight-homes]] · [[team]] · [[hubspot]] · [[BBYS-LO-Payment-FAQ-DRAFT]]

**Prepared by:** RevOps (Tulli / Claude)
**Date:** April 15, 2026
**Data Sources:** Slack (#homes-payment-requests, #ongoing-payments-requests), Gmail (BBYSpayments@homelight.com — ~65+ threads), HubSpot tickets

---

## Executive Summary

After a comprehensive review of payment-related communications across Slack (#homes-payment-requests, #ongoing-payments-requests), Gmail (BBYSpayments@homelight.com, ~65+ threads), and HubSpot tickets, **17 recurring themes** emerged across two analysis passes. The dominant pain points are **payment timing confusion**, **incomplete Trolley profiles**, and **missing/incorrect payment plans**. Together, these three categories account for an estimated 70-80% of all inbound payment questions from LSMs on behalf of their loan officers.

A deep review uncovered critical systemic issues including the **Thursday cleanup script deleting payments** rather than holding them, **dual Trolley profiles** from email mismatches, and **portal status sync bugs**. Gmail data revealed that **80% of email volume is internal coordination** (not direct LO inquiries) and that **W-9 withholding validation** is a surprisingly large manual workload for Tara Edwards.

The highest-leverage improvements are: (1) a proactive LO-facing FAQ sent at onboarding, (2) changing the Thursday script from "delete" to "hold" for incomplete profiles, (3) automated Trolley profile status alerts, and (4) clearer payment plan visibility in the Sales App.

---

## Theme Analysis (Ranked by Frequency)

### 1. "When Will I Get Paid?" — Payment Timing Confusion
**Frequency:** Very High (appears in nearly every thread)
**Who asks:** LSMs on behalf of LOs, occasionally LOs directly

**What's happening:** Loan officers close a deal and expect payment within days. They don't understand the batch cycle, the Thursday processing schedule, or the dependency on Trolley profile completion. LSMs field these questions constantly and often don't know the answer themselves.

**Common sub-questions:**
- "The deal closed 2 weeks ago, where's the payment?"
- "When does the next batch go out?"
- "Why hasn't my LO been paid yet?" (answer is almost always: Trolley profile incomplete)
- Confusion about monthly vs. weekly batch timing for certain partners

**Root causes:**
- No proactive communication to LOs about payment timeline at deal close
- LSMs aren't trained on the payment lifecycle
- The Thursday batch + Tuesday tax review cycle isn't documented anywhere LOs can see
- Monthly batch partners (first Thursday after first Monday) is confusing even internally

**Recommendations:**
- **Quick win:** Create a one-page "BBYS Payment Timeline" visual showing the lifecycle from deal close → Trolley setup → batch processing → payment received. Distribute to all LSMs and include in LO onboarding.
- **Medium-term:** Trigger an automated email/SMS to the LO at deal close explaining the payment process and linking to Trolley setup.
- **Long-term:** Add a "Payment Status" widget to the LO portal showing real-time status of their pending payments.

---

### 2. Trolley Profile Incomplete / Not Set Up
**Frequency:** Very High
**Who asks:** LSMs, Sarah Jaka, Tara Edwards

**What's happening:** LOs receive an invitation to set up their Trolley profile (homelight.portal.trolley.com) but don't complete it. Without a completed profile, the Thursday cleanup script deletes their payment from the batch. This is the #1 reason payments are delayed.

**Common sub-issues:**
- LO never received the Trolley invite (email went to spam, wrong email on file)
- LO started the profile but didn't complete tax document upload
- Tax documents submitted on Wednesday — miss Tuesday review, delayed a full week
- LO completed profile but it's under a different email than what's in HubSpot
- Company-level Trolley profiles vs. individual LO profiles causing confusion

**Root causes:**
- No follow-up sequence when Trolley invite goes unopened
- No visibility into Trolley completion status from HubSpot or Sales App
- Tuesday tax review + Thursday batch creates a tight window that isn't communicated
- Email mismatch between Trolley and HubSpot isn't caught automatically

**Recommendations:**
- **Quick win:** Create a Slack alert (or n8n workflow) that flags LOs with pending payments but incomplete Trolley profiles 48 hours before each Thursday batch. Sarah/Tara can proactively reach out.
- **Medium-term:** Build an n8n workflow that sends reminder emails to LOs at day 1, day 3, and day 6 after Trolley invite if profile remains incomplete. Include the direct portal link and a short video walkthrough.
- **Long-term:** Surface Trolley profile status in the Sales App so LSMs can see it without asking payments team. Consider a HubSpot property that syncs Trolley completion status.

---

### 3. Payment Plan Missing or Incorrect on Lead
**Frequency:** High
**Who asks:** LSMs, Sarah Jaka

**What's happening:** Payments fail or route incorrectly because the payment plan wasn't applied to the lead in the Sales App, or the wrong plan was applied. This is especially common with newer partners, company changes, or when an LO moves between companies.

**Common sub-issues:**
- Lead created without a payment plan → payment can't process
- Wrong payment plan applied (e.g., standard plan instead of partner-specific plan)
- `needs_internal_review` status triggered by payment plan issues — fix requires removing and re-adding the plan
- New partner onboarded but payment plan not yet created in the system
- Confusion about which plan applies: TLS $1500, NFM, Orchard $400, Princeton (company-paid), US Mortgage $1000, Edge Home Finance $1500, etc.

**Root causes:**
- Payment plan assignment is manual and depends on LSM or ops knowing the correct plan
- No validation that a payment plan exists before a deal progresses
- Partner-specific plans aren't documented in a single, accessible reference
- When partners change plans or new partners onboard, there's no systematic update process

**Recommendations:**
- **Quick win:** Create and maintain a "Payment Plan Reference Sheet" listing every active partner, their plan name, amount, and whether payment goes to LO or company. Share with all LSMs.
- **Medium-term:** Add a HubSpot workflow validation — if a deal reaches "Closed Won" without a payment plan, trigger an alert to the payments team and the owning LSM.
- **Long-term:** Auto-assign payment plans based on the LO's company association in HubSpot, reducing manual selection.

---

### 4. LO Company Change — Payment Routing Confusion
**Frequency:** Moderate
**Who asks:** LSMs

**What's happening:** When a loan officer moves from one mortgage company to another, their payment routing may need to change. Deals that were originated under the old company may still need to pay under the old plan, while new deals should use the new company's plan. LSMs don't know how to handle this.

**Common sub-issues:**
- LO switched companies mid-deal — who gets paid?
- Old Trolley profile is under old company email; new company needs new profile
- Payment plan on existing leads still references old company's plan
- Some companies pay the LO directly, others pay the company — a switch changes the routing

**Root causes:**
- No documented process for handling LO company transitions
- HubSpot contact company association may not be updated promptly
- Trolley profiles are email-based, so company changes often require new profiles
- No automated detection of company changes

**Recommendations:**
- **Quick win:** Document a "LO Company Change Checklist" for LSMs: update HubSpot company association, verify/create new Trolley profile, update payment plan on active leads, notify payments team.
- **Medium-term:** Create an n8n workflow that detects when a contact's company association changes and triggers the checklist automatically via Slack or task assignment.

---

### 5. Payment to LO vs. Company Confusion
**Frequency:** Moderate
**Who asks:** LSMs (they often don't know the routing rules)

**What's happening:** Some partners (like TLS) pay the LO directly. Others (Princeton, some company-paid plans) route payment to the company. LSMs frequently ask "does this go to the LO or the company?" and sometimes submit incorrect information.

**Root causes:**
- Routing rules aren't centralized in a reference doc
- Rules vary by partner and sometimes by deal type
- LSMs learn through tribal knowledge, not training

**Recommendations:**
- **Quick win:** Include LO-vs-company routing in the Payment Plan Reference Sheet (recommendation from Theme 3). Add a column: "Pays To: LO / Company."
- **Medium-term:** Make this visible in the Sales App payment plan selector — when an LSM selects a plan, show "Payment routes to: [LO/Company]."

---

### 6. System Failures and Status Issues
**Frequency:** Moderate
**Who asks:** Sarah Jaka, LSMs

**What's happening:** Payments occasionally land in problematic statuses: `needs_internal_review`, `failed`, or `returned`. Each requires different remediation, and the process isn't well-documented.

**Common sub-issues:**
- `needs_internal_review` — almost always fixed by removing and re-adding payment plan
- `failed` — usually a Trolley/banking issue on the recipient's end
- `returned` — bank rejected the payment (wrong account info, closed account)
- Payments stuck in `pending` or `created` longer than expected

**Root causes:**
- No self-service documentation for common status fixes
- Sarah/Tara are the only people who know remediation steps
- No automated alerting when payments hit error statuses

**Recommendations:**
- **Quick win:** Create a "Payment Status Troubleshooting Guide" for Sarah/Tara (and eventually LSMs) documenting the fix for each status.
- **Medium-term:** Build an n8n workflow that monitors for `needs_internal_review` and `failed` statuses and auto-alerts the payments team in #homes-payment-requests with the lead details and suggested fix.

---

### 7. Portal/UI Visibility Issues
**Frequency:** Low-Moderate
**Who asks:** LSMs, LOs (via LSMs)

**What's happening:** LOs or LSMs check the Sales App or Trolley portal and see confusing information: $0 payment amounts, missing dollar signs, wrong status displayed, or payments that don't appear at all.

**Common sub-issues:**
- Payment shows $0 in the portal (usually means payment plan not yet applied)
- Missing "$" symbol making amounts hard to read
- Status in portal doesn't match actual payment status
- LO can't find their payment in Trolley portal

**Root causes:**
- Portal UI doesn't gracefully handle edge cases (null amounts, pending plans)
- Trolley portal and Sales App may show different statuses
- No "last updated" timestamp to help users know if data is stale

**Recommendations:**
- **Quick win:** File tickets for the $0 display and missing $ symbol issues — these are likely quick frontend fixes.
- **Medium-term:** Add a "Payment FAQ" link directly in the Sales App payment section so users can self-serve when they see confusing data.

---

### 8. Tax Document (1099/W-9) Requests & Withholding Confirmations
**Frequency:** Moderate-High (Gmail data shows ~23 threads in recent months — 46% of email volume; seasonal spikes in Q1 plus ongoing W-9 withholding validation)
**Who asks:** LOs directly to BBYSpayments@, company payroll departments, plus proactive outreach from Tara Edwards

**What's happening:** This theme is larger than Slack alone suggested. Gmail reveals two distinct sub-patterns: (1) LOs unable to find 1099s or needing corrections, and (2) a high volume of W-9 withholding confirmation emails where Tara proactively contacts LOs who indicated 24% IRS backup withholding on their W-9. The withholding confirmation emails follow a template ("Our team was reviewing the W-9 information you submitted recently through our portal and noticed you indicated being subject to 24% withholding...") and represent significant manual effort.

**Common sub-issues:**
- LO can't find 1099 in Trolley portal (e.g., Tyler Eads: "I am unable to find my 1099 for the $1500 I was paid when I login")
- W-9 withholding set incorrectly — requires manual outreach to confirm/correct
- Company payroll departments requesting payment breakdowns they can't see on dashboards

**Root causes:**
- No self-service access to tax documents in the LO portal (or LOs can't navigate to them)
- W-9 withholding validation is entirely manual — Tara reviews each submission individually
- Process for requesting 1099 corrections isn't documented for LOs
- Seasonal volume spike catches team off guard, but withholding confirmations are year-round

**Recommendations:**
- **Quick win:** Add tax document instructions to the LO FAQ (deliverable #2 from this analysis). Include: where to find 1099s, how to update W-9, who to contact for corrections.
- **Quick win:** Add a W-9 guidance note during Trolley profile setup explaining the 24% withholding field — most LOs select it incorrectly, triggering manual follow-up.
- **Medium-term:** Proactively send a "Tax Document Reminder" email to all paid LOs in early January with links and instructions.
- **Medium-term:** Explore whether Trolley's W-9 flow can validate or flag the withholding field automatically, reducing Tara's manual review burden.

---

### 9. Special/Exception Payment Requests
**Frequency:** Low
**Who asks:** LSMs, Jake Vogel (deal exception authority)

**What's happening:** Occasionally, good-faith payments, manual overrides, or one-off exception payments need to be processed outside the normal flow. These require special handling and approvals.

**Common sub-issues:**
- Good-faith payments for deals that didn't fully close
- Manual payment adjustments (overpayment, underpayment corrections)
- Rush payment requests for unhappy LOs
- Payments for deals with unusual structures

**Root causes:**
- No formal exception request process
- Approvals happen ad-hoc via Slack DMs
- No tracking of exceptions for audit purposes

**Recommendations:**
- **Quick win:** Create a simple exception request form (Google Form or HubSpot form) that captures: deal ID, LO name, exception type, amount, reason, and approver. Route submissions to #homes-payment-requests.
- **Medium-term:** Track exceptions in a Supabase table for reporting — useful for identifying partners or deal types that consistently require exceptions.

---

## Deep Review: Additional Findings

A second pass through all sources uncovered several additional patterns that enrich the themes above and reveal new sub-issues worth addressing.

### 10. Thursday Cleanup Script — Cascading Payment Deletions
**Frequency:** Moderate (root cause behind many "where's my payment?" threads)

The Thursday batch script deletes all payments where the Trolley profile is incomplete. This means if an LO completes their profile on Wednesday (after the Tuesday tax review cutoff), the profile won't be activated until the *following* Tuesday — but the script has already deleted their payment on Thursday. Sarah then has to manually re-create the payment for the next batch. This creates a full 2-week delay from what should have been a same-week payment.

**Verbatim from Sarah:** *"The script that runs on Thursdays deletes all payments with incomplete profiles, therefore there is no pending payment in the system. I am looking into this."*

**Recommendation:** Instead of deleting payments with incomplete profiles, move them to a "held" status so they automatically process once the profile is activated, without requiring manual re-creation.

### 11. Dual Trolley Profiles / Email Mismatch
**Frequency:** Moderate

When an LO's email in the Sales App differs from their Trolley profile email, the system creates a second Trolley profile. Sarah must then manually route the payment to the correct profile. This happened with Ron Roberts (ron@rjrteam.com vs ron.roberts@amerifund.com) and Ryan Nash (needed an account reset because the emails didn't match).

**Verbatim from Sarah:** *"The SA has the ron@rjrteam.com email address, therefore it created a new profile in Trolley under this email address. The LO has a complete Trolley profile under ron.roberts@amerifund.com."*

**Recommendation:** Add an email match validation check before payment creation. If the SA email doesn't match any existing Trolley profile, flag for manual review rather than silently creating a duplicate.

### 12. "Waiting for DR Sale" Portal Status Mismatch
**Frequency:** Low-Moderate

LOs see "waiting for DR sale" in their portal even when the Sales App shows the deal as DRX (DR closed). This causes unnecessary anxiety and triggers payment inquiries to LSMs.

**Verbatim from Tejas:** *"LO says his portal says 'waiting for DR sale' even though SA is DRX"*

**Recommendation:** This is a sync/data issue between the portal and SA. File as a bug — the portal should reflect current SA status.

### 13. Orchard Payment Complexity
**Frequency:** Moderate (concentrated in bulk batches)

Orchard payments involve unique complexity: the plan changed from $1,500 to $400 in January 2026, bulk batches can total $10,400+, data cleanup is occasionally needed before payments can process, and the team has discussed backdating payment plans. Zahedi (engineering) confirmed the plan logic is correct (pre-Jan 5 leads = $1,500, post-Jan 5 = $400) but the mechanics of creating bulk Orchard payments still require significant manual coordination between Sarah, Tulli, and engineering.

**Recommendation:** Document the Orchard payment process separately as an internal runbook. Consider whether Orchard payments can be automated more fully given their recurring nature.

### 14. "Failed" Status Has Same Fix as "Needs Internal Review"
**Frequency:** Low but important for process documentation

Sarah discovered that the fix for "failed" status is the same as "needs_internal_review" — remove the payment plan and re-add it. She asked Joel: *"Going forward anything with failed should be resolved in this manner?"* Joel confirmed and noted that `keygen` or `canceled_returned` commands work if normal steps error.

**Recommendation:** Add this to the Payment Status Troubleshooting Guide. Both `needs_internal_review` and `failed` → remove payment plan → re-add → status changes to `eligible`.

### 15. Company-Level Trolley Profiles (Not Just LO Profiles)
**Frequency:** Low-Moderate

It's not just individual LO profiles that can be incomplete — company-level profiles for partners like Princeton, Fairway, and APM also need to be set up and maintained. Sarah noted: *"The company has not completed their payment profile to receive payments."*

**Recommendation:** Maintain a tracker of company-level Trolley profile status for all payment plan partners. Flag incomplete company profiles proactively.

### 16. LOA (Loan Officer Assistant) Email Submission Issues
**Frequency:** Low

LO assistants sometimes submit BBYS applications under their own email rather than the LO's, creating mismatched records. Matt Leddy flagged: an LOA submitted under a new email, the LO wants accounts merged, and is asking if she'll get paid for prior deals submitted the same way.

**Recommendation:** Add guidance in the LO FAQ about ensuring the correct email is used for submissions. Consider a field-level validation or warning in the application flow.

### 17. Partner Company Dashboard/Spreadsheet Visibility
**Frequency:** Low-Moderate (Gmail)

Company payroll departments (Edge, Fairway) receive payments but can't see breakdowns on their dashboards. They email asking for updated spreadsheets or refreshed dashboard access. Edge payroll expressed frustration: *"Sorry but why are we suddenly having issues with every other file? Nothing has changed."*

**Recommendation:** Ensure partner dashboards (Periscope) are auto-refreshed and include all recent payments. Consider proactive notification to partner payroll when a batch processes, with a link to their dashboard.

---

## Impact / Effort Matrix

| Recommendation | Impact | Effort | Priority |
|---|---|---|---|
| LO-facing FAQ document (see deliverable #2) | High | Low | **Do Now** |
| Payment Plan Reference Sheet for LSMs | High | Low | **Do Now** |
| "Payment Timeline" one-pager for LO onboarding | High | Low | **Do Now** |
| Payment Status Troubleshooting Guide (internal) | Medium | Low | **Do Now** |
| LO Company Change Checklist | Medium | Low | **Do Now** |
| Trolley incomplete alert 48hrs before Thursday batch | High | Medium | **Next Sprint** |
| Trolley reminder email sequence (n8n) | High | Medium | **Next Sprint** |
| HubSpot workflow: alert if deal closes without payment plan | High | Medium | **Next Sprint** |
| Auto-alert on error payment statuses (n8n) | Medium | Medium | **Next Sprint** |
| Automated email to LO at deal close with payment process | High | Medium | **Next Sprint** |
| W-9 withholding guidance note during Trolley setup | Medium | Low | **Do Now** |
| W-9 auto-validation to reduce Tara's manual reviews | Medium | Medium | **Next Sprint** |
| Surface Trolley status in Sales App | High | High | **Roadmap** |
| Auto-assign payment plans from company association | High | High | **Roadmap** |
| Payment Status widget in LO portal | High | High | **Roadmap** |
| Exception request form + Supabase tracking | Low | Medium | **Roadmap** |
| Document "failed" status fix (same as needs_internal_review) | Medium | Low | **Do Now** |
| Orchard payment internal runbook | Medium | Low | **Do Now** |
| Company-level Trolley profile status tracker | Medium | Low | **Do Now** |
| Email match validation before payment creation | High | Medium | **Next Sprint** |
| Change Thursday script to "hold" instead of "delete" | High | Medium | **Next Sprint** |
| Fix "waiting for DR sale" portal sync bug | Medium | Medium | **Next Sprint** |
| Partner dashboard auto-refresh + batch notifications | Medium | Medium | **Roadmap** |

---

## Key Insight from Gmail Data

Gmail analysis (~65+ threads) revealed an important pattern: **80% of BBYSpayments@ email volume is internal coordination** — HomeLight staff (LSMs, Tara, Sarah) looping in the payments team, not LOs writing directly. Only ~18% of threads are direct LO inquiries. This means most LO questions flow through intermediaries, suggesting that empowering LSMs with better self-service info could dramatically reduce payments team workload.

Gmail also elevated the **tax/W-9 theme** significantly — it accounted for 46% of email volume in the analyzed batch, driven by Tara's manual W-9 withholding validation process.

## Data Gaps & Next Steps

1. **HubSpot tickets were mostly PPL/referral agent payments** (PPLClosings pipeline), not BBYS LO comp. The BBYS-specific payment issues live primarily in Slack channels and email, not in HubSpot tickets — this is itself a finding worth noting for process design.
2. **Quantitative frequency data:** Slack search doesn't provide exact counts. The frequency rankings above are qualitative based on thread volume and recurrence. For exact numbers, consider tagging payment requests in a structured system going forward.
3. **Gmail pagination:** Analysis covered ~65+ most recent threads. Older threads may contain additional patterns, particularly around seasonal tax document spikes in January-February.

---

*This analysis covers data through April 2026. Review quarterly to assess whether implemented changes are reducing ticket volume.*
