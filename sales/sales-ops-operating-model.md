---
last_updated: 2026-09-11 (+pointer to [[payments-ops-bible]])
threads added under §4: the mandatory AM 4-business-hour intro-call SLA, the ad hoc reward-split
policy, an RWM back-pay sign-off precedent, and the uncentralized-partnership-agreements gap.)
status: current
source: Slack channels (6-month sweep, 2026-03-01 to 2026-08-29) — see channel list below · DMs D051H6JJTK3 (Jake Vogel), D04LLU0URSR (Sarah Jaka), 2026-08-29
scope (updated 2026-08-29 — §8 added from Google Drive): how the sales/ops org actually moves deals — routing & escalation, payment ops, valuations/approvals workflow, listing & closing ops
source: HomeLight-Vault/context/sales-ops-operating-model.md
imported: 2026-09-29
---

# Sales & ops operating model

Durable, observed operating patterns for the BBYS sales/ops org — not the product spec, the *how work actually
flows day to day*. Built from a 6-month Slack sweep (2026-03-01 → 2026-08-29) of `#hubspot-help`,
`#hs-contact-assignments-and-merges`, `#no-lsm-lrm-fallback-alerts`, `#ongoing-payments-requests`,
`#homes-payment-requests`, `#proj-lo-payments`, `#proj-payments-to-loan-officers`,
`#trolley-hubspot-integration-errors`, `#approvals-valuations-team`, `#utilities--listing-ops`,
`#proj-notary-scheduling-flow-revamp`, `#retail-channel`, `#proj-retail-wholesale`. Companion to [[hubspot]],
[[bbys-routing-and-ownership]], [[bbys-lifecycle-operations]], [[deal-operations]],
[[2026-08-28-vault-refresh-slack-findings]] (do not duplicate that file's reorg/DTI-Drop/pricing findings — this
doc is the routing/payments/valuations/closing layer it didn't cover).

## 1. How LO/company ownership actually gets assigned (observed)

`#hs-contact-assignments-and-merges` is a live bot log of every automated LSM assignment and contact merge.
Sampling 2026-08-27–28 gives the **actual precedence order** the assignment engine runs, in priority order:

| Order | Source (bot's own label) | Behavior |
| --- | --- | --- |
| 1 | `company_lender_sales_manager` | Contact inherits the company-level LSM directly, or round-robins among **multiple** LSMs listed on that company |
| 2 | `partnership_lender_sales_manager` | Round-robins among the partnership's LSMs, gated on partnership **Stage = Launched** |
| 3 | `weighted_round_robin` | Fallback when no company LSM, no pod, and no partnership match exists — "No company LSM, pod, or partnership found. Round-robin to Pod N." |

🔑 Observed pod round-robin memberships from the same log (2026-08-28 sample): **Pod 1** = Tierney Izar, riché-la
(Richie Helali) · **Pod 2** = Tejas Narkhede, TJ Sims, Brian Banes · **Pod 3** = Marisa Drake, Kara Kleingarn.
These are the actual pairs the round-robin cycles between — not just static pod lists.

A **separate automated contact-merge bot** runs in the same channel, matching duplicate contacts by NMLS ID and
tagging a confidence score (`Confidence: high` on every observed merge). This explains a split pattern: routine
NMLS-matched duplicates (e.g. two emails for the same licensed LO) merge silently via the bot, while
**cross-domain duplicates without a shared NMLS ID on file** (e.g. `jfigueroa@nexamortgage.com` vs
`jfigueroa@nexalending.com` — same person, company rebranded/added a domain) fall through and generate a manual
`#hubspot-help` ticket instead. See § 2.

### 🔴 The routing failure mode, live and high-volume

`#no-lsm-lrm-fallback-alerts` fires an automated post every time a new BBYS lead submission can't find an LSM
pod and falls back to LRM. Sampling 2026-08-24–28 shows this firing **multiple times per day**, often with `LO
HubSpot Contact: Not available` / `LO Name: Not available` — meaning the lead came in with data the router
can't even look up. This is the live, quantified confirmation of the data-model gap Jake Vogel flagged in
[[2026-08-28-vault-refresh-slack-findings]] ("we don't have a clear way to identify which AM/LRM actually owns a
particular LO").

- **2026-08-26, Jake Vogel:** *"can we remove Lennar from these please?"* — Lennar-mortgage-domain LOs
  (`@lennarmortgage.com`) keep triggering fallback alerts despite Lennar being a dead partnership (team no
  longer works Lennar, per [[2026-08-28-vault-refresh-slack-findings]]). The routing rules haven't been updated
  to reflect the partner exit.
- **2026-08-26, Jose Herrera** joined the channel "as context while you rework lead routing" — active engineering
  work on this exact problem is underway as of this sweep, no completion date observed.

### AM (formerly LRM) intro-call SLA — made mandatory 2026-08-06

Jake Vogel to Tulli (DM D051H6JJTK3, 2026-08-06): *"Can you build me something that tracks the
AMs (old LRMs) intro calls on LOs first and second deals submitted. I would like to see the %
outreach, what deals aren't called. Basically called within 4 business hours Monday to Friday.
**We are making this mandatory.**"* 🔑 **Stated policy, not yet observed in code or a dashboard**
— no vault doc previously captured a numeric SLA for AM/LRM first-touch outreach; this is the
first. Same thread (2026-08-06/12) also specs conversion-rate reporting requirements for both
AEs and AMs (App→Approval / App→IRUC / Approval→IRUC, monthly cohorted) — unclear whether either
the SLA tracker or the cohort dashboard has shipped; not found elsewhere in the vault as of this
pass.

### Ad hoc reward-split requests — HomeLight pays the company, not the individuals

Sarah Jaka DM (D04LLU0URSR), 2026-08-10: a company asked HomeLight to split a $7k "Summer of
Savings" pool across 5 named individuals directly. Tulli declined the per-person disbursement
and gave two options instead — pay the company (John) or pay the sponsoring lender (Fairway) and
let them disburse internally: *"We shouldn't need to worry about this... it's hard enough to
wrangle LOs to fill out their info... I'm not trying to have us do the same and then keep track
of that for 5 people for ad hoc payments."* This generalizes the same pattern already documented
above for Thrive (paid to the company, not the individual LO) — HomeLight's payment ops will not
stand up ad hoc multi-recipient disbursement, on precedent now confirmed twice.

### Back-pay for a pre-payment-plan deal needs VP sign-off

Sarah Jaka DM, 2026-07-02: a 2024 deal (LO on the RWM comp plan) came in asking for back-pay from
before the RWM payment plan existed. Tulli: *"I'm good with it but he'll need Nick's sign-off
since I think that's well before the start of the RWM payment plan."* 🔑 New approval precedent —
back-pay requests predating a partner's current payment-plan start date route to Nick (Friedman)
for sign-off, not to Tulli/Sarah alone.

### Executed partnership agreements are not centralized

Sarah Jaka DM, 2026-06-23: Legal (Pooja) needed all executed partnership agreements urgently
(Fairway and Cross Country first) and Sarah could not find them in the partners' own Slack
channels — *"It may take some time... as we didn't collect them in one place."* No central
repository for signed partner agreements is documented anywhere else in the vault; this is a new
gap, not previously flagged. See also [[partners]] for individual partner terms, which does not
include a document-storage location.

## 2. Escalation map — who approves what

| Decision | Approver | Evidence |
| --- | --- | --- |
| Fee exceptions (DTI Drop pricing, program-fee overrides) | **Jake Vogel** | [[2026-08-28-vault-refresh-slack-findings]] §2 (pre-existing finding, not re-derived here) |
| Variable pricing (per-deal exception) | **Jake Vogel** | [[2026-08-28-vault-refresh-slack-findings]] §3 (pre-existing finding) |
| Retail partner-level comp plans (e.g. CMG, Luminate) | **External, partner-side**: Nadine Rocha (Product Strategy Mgr) and Eric Lovins (President of Retail Lending) — HomeLight-side sign-off still pending as of 2026-08-20 | `#retail-channel`, 2026-08-20 (Kim Tanner) |
| HELOC Boost conditional-agreement send/hold | **Automated threshold**, not a human — Equity Unlock ≥90% of Target Equity auto-generates and sends; EU >10% below Target Equity auto-generates but withholds send pending review | `#approvals-valuations-team`, 2026-08-06 (Ciro Affronti, on Marc [Kaplan]'s change) |
| Market-driven RAV (valuation) risk adjustments | **Jason Smith** (valuations team lead) issues standing directives by market (e.g. "5% RAV reduction in FL if active:pending ≥3:1") that PH analysts apply | `#approvals-valuations-team` |
| Notary/closing flow state-gating rules (which states allow RON) | **Zahedi Aquino** (eng) implements on direction from **Derek Lupien** (Closing Signing Ops) | `#proj-notary-scheduling-flow-revamp` |

⚠️ None of these approval paths are enforced in the system — they are social/Slack-channel conventions. Ian
Pardo's 2026-08-27 comment in [[2026-08-28-vault-refresh-slack-findings]] ("I don't recall asking for variable
pricing... was it even approved?") generalizes: **approval state for fee/pricing exceptions lives only in Slack
channel history, not in HubSpot or Sales App**, for every approval type in this table.

## 3. HubSpot pain points and recurring data issues

Full detail and dated log now lives in [[hubspot]] § Repeat Requests / § Common Queries (appended 2026-08-29
from this sweep). Headline pattern from a full `#hubspot-help` read-back to 2026-03: **the single largest
ticket category, every month, is `Fix (reassign, reassociate, etc.)`** — company/LO ownership fixes, contact
merges, DBA/JV structuring, and duplicate-email splitting, overwhelmingly requested by Tejas Narkhede, Tierney
Izar, riché-la, and Brian Banes. See [[hubspot]] for the specific recurring bugs (Executive contact-type
misclassification, non-marketing workflow no-op, Aircall outcome inaccuracy, LSM-attribution gap on new
contacts) and the DBA/merge/reassignment request patterns in detail.

## 4. Payment ops — LO payments, Trolley, referral credits, commission

> 🔑 **2026-09-11:** Living deep-dive lives in [[payments-ops-bible]] (systems, 13-field plan model, HAPI/sales-app code path, redirect policy, open loops). Keep §4 for observed Slack/KB patterns; prefer the bible for Payments Ops day-to-day.


### Mechanics (from `#proj-lo-payments`, project kickoff 2024-10-18, still the operating structure)
- **Payout structure:** TLS LOs — higher of $1,500 flat or 10bps of final DR sale price on a cash-transaction
  referral · any-lender cash-transaction referral — 10bps of final DR sale price · Thrive (Lower.com subsidiary)
  — $1,500 flat, paid to the company not the individual LO.
- **Collection flow:** W-9 + ACH Authorization via a DocuSign power form, sent once a file is payable (current
  trigger: IR Contract / clear-to-fund). Payment recipient defaults to the transaction-file LO **unless** the
  lender company has told ops to pay the company directly (Thrive does this by standing instruction; TLS
  brokers are handled case by case because the broker relationship, not HomeLight, decides payee).
- **Disbursement rail:** Trolley. LOs must complete a Trolley payment profile (bank info + tax form) before any
  disbursement can process — this is the single biggest bottleneck (below).

### 🔴 The Trolley profile backlog is not shrinking

`#ongoing-payments-requests` gets a weekly automated **"BBYS Payment Profile Report"** from the "HomeLight Sales
App" bot, breaking incomplete-profile LOs down by LSM. Trend across four consecutive weeks in August:

| Report date | LOs with incomplete profile | Leads affected | All-time failed LO-payout payments |
| --- | --- | --- | --- |
| 2026-08-05 | 618 | 723 | 1,982 |
| 2026-08-12 | 624 | 729 | 2,035 |
| 2026-08-19 | 625 | 726 | 2,105 |
| 2026-08-26 | 615 | 709 | 2,180 |

The weekly incomplete-profile count holds roughly flat (615–625, ~15 LSMs), but the **all-time failed-payment
counter climbs every week** (+198 over 3 weeks) — meaning failed payments are accumulating faster than they're
being resolved, not clearing out. **~309–312 of the incomplete LOs every week are `Unassigned`** (no LSM at
all) — the single largest bucket, larger than any individual rep's book, and the same underlying ownership gap
as § 1.

Companion weekly bot posts in the same channel: **"Trolley Inactive Recipient Report"** and **"Trolley Offline
Check Payments Pending"** — the latter requires a human to manually mark checks paid in "payments operations."

### Other observed payment-ops friction (all `#homes-payment-requests` / `#ongoing-payments-requests`, Aug 2026)
- **Payment plans are attached manually, per deal, on request** — ops (Sarah Jaka) attaches the correct
  lender-specific plan (TLS / Barrett / Edge Home Finance / UWM / VIP Mortgage / Arbor / Fairway) to a Sales App
  lead only when a rep asks in Slack. There is no default/automatic plan assignment.
- **Wrong-plan bug (2026-08-28):** a deal auto-attached the **X2** payment plan instead of TLS, even though the
  LO has no X2 affiliation — the Lender Company field read "X2," and the plan-selection logic appears to key off
  that field rather than the LO's actual payment relationship. Sarah Jaka had to manually swap it.
- **Bulk payment tool broken (2026-08-24):** a bulk payment to Barrett Financial Group covering 5 payments only
  populated 1; the other 4 had to be entered manually into the invoice.
- **Zapier↔Trolley↔HubSpot sync drops Lead IDs**: `#trolley-hubspot-integration-errors` shows a recurring
  Zapier error, *"No Lead ID was found on the Payment ..."* — a payment record arrives without the join key
  needed to tie it back to a Sales App lead.
- **LO-side confusion is a recurring theme**: "when do I get paid — bridge funding or DR sale?" (Fairway LO,
  2026-08-14); "I was never paid for this file" tickets that turn out to be an incomplete Trolley profile, not a
  missed payment; requests for HomeLight's EIN because it's masked on a prior-year 1099 copy (referencing the
  **24% backup-withholding** default noted in [[2026-08-28-vault-refresh-slack-findings]] §15); Trolley 2FA
  verification codes not arriving at all (Fairway LO, 2026-08-25, "checked spam, checked company-held email").

## 5. Valuations / approvals team workflow

- **Org:** US lead Jason Smith + Preston/Scott Farress/Brian Karch/Hugh Rodman/Jarrod Johnson, plus a
  **Philippines-based analyst team** (Angie Paguidopon, Kimberly Delmonte, "Jepoy_PH," rey membreve) doing the
  bulk of valuation execution, PH-hours with a US handoff window.
- **Task flow:** valuation tasks are filtered in Sales App by the tag **`LPVAL`**; on completing a valuation the
  analyst creates a follow-on **"Conditional Approval"** or **"Refresh Approval"** task due 4 hours later — a
  manual two-step handoff, not a single automated task chain.
- **RAV discretion is directive-driven, not systematized**: Jason Smith periodically issues market-specific
  guidance in-channel (e.g. *"active:pending ratio ≥3:1 in these FL metros → apply a 5% RAV reduction"*) that
  analysts are expected to apply by hand on affected valuations. This is a standing instruction living only in
  Slack, not a configured rule.
- **2026-08-06 process change (still rolling out as of this sweep):** the "Generate Documents" toggle for BBYS
  deals with a HELOC Boost application had been grayed out entirely; that's being reverted. New behavior: the
  system will auto-preset the send/no-send toggle based on Equity Unlock vs Target Equity (see § 2 table) and
  surface a pop-up explaining the preset — approvals staff no longer manually change disposition, just confirm
  or override the toggle.

## 6. Listing ops and closing ops

### Listing ops — utility setup/cancellation is fully manual, per property
`#utilities--listing-ops`: Evan Jaquias-Johnson (and Candice Jenkins) post per-property utility account details
(company name + phone number, sometimes account numbers) for **EJ Cendana** to call and set up ahead of COE, and
to cancel at CTC/close. Every property is a fresh manual lookup — there is no utility-provider database keyed by
address/ZIP; the same providers (e.g. TXU, Atmos, various municipal water depts) recur constantly across Texas
deals, an obvious candidate for a lookup table if volume justifies it.

### Closing ops — notary/signing flow, multiple live bugs (`#proj-notary-scheduling-flow-revamp`)
- **RON (Remote Online Notarization) is state-gated**, restricted to a maintained list; Delaware was explicitly
  added to the "no RON" list 2026-05-14 on Derek Lupien's request. ⚠️ The gating list lives in a PDF, not a
  queryable config — engineering (Zahedi Aquino) has had to ask "should UT also be excluded, or was it another
  state?" when re-implementing (2026-06-10).
- **Hybrid + Split signing is force-converted to Wet Signing** as of 2026-06-23 — the system silently overrides
  the user's Hybrid selection whenever Split signing is also selected, with an explanatory note added to the
  confirmation screen and the ops Slack post.
- **Weekend-signing bug (2026-06-04):** a client was able to book a Saturday signing despite weekend signings
  supposedly being blocked.
- **COE-proximity bug (2026-06-02):** the notary link let a client schedule a signing for the same day as COE,
  when the intended rule is no earlier than 2 days before COE (rush files excepted).
- **Non-Borrower Spouse (NBS) fields** added 2026-05-28 to populate Client 2 data for community-property-state
  signings (AZ, CA, ID, LA, NV, NM, TX, WA, WI). ⚠️ Even after this ship, a 3rd title holder correctly listed
  in Sales App still failed to appear on the notary scheduling link for at least one file (2026-06-23) — the
  NBS/multi-signer plumbing is not fully reliable.
- **Repaper (re-execution) reschedule confusion (2026-08-18):** the reschedule email tells the client to
  "contact your loan officer," but LOs are not briefed on the repaper process and get blindsided by the call —
  an unresolved comms gap flagged by Cheryl Funk.
- **Snapdocs webhook noise reduction (2026-05-27):** Slack notifications from Snapdocs were filtered down to
  only comment events, "closed" status, and document-processing status — a direct response to alert fatigue
  from the closing ops team.

## 7. Sep 1 Wholesale/Retail reorg — delta beyond `2026-08-28-vault-refresh-slack-findings`

That file is the authoritative reorg record; this section only adds what a live read of `#retail-channel` and
`#proj-retail-wholesale` surfaced beyond it.

- **Angelica Espinosa's onboarding to Retail is now directly observed**, not just reported: she joined
  `#retail-channel` 2026-08-20 with a team welcome from Kim Tanner and Deborah, resolving the 8/03-vs-8/21
  Retail-AM-roster discrepancy already flagged in the existing findings doc in Angelica's favor.
- **Barb Griego left `#retail-channel` 2026-08-24** — consistent with her being dropped from the 8/21 Retail AM
  list, now confirmed by an actual departure event rather than just a roster edit.
- 🔴 **Partner-level comp plans are still unresolved for at least two major retail accounts while LO training is
  already underway.** Kim Tanner's 2026-08-20 recap: a CMG (Kris Nelson team) training drew 50+ attendees and a
  Luminate training drew 35+, with a note attached to each: *"in case LO asks about comp"* — comp plans for both
  accounts were still being chased with the partner's own leadership (Nadine Rocha at CMG's parent, Eric Lovins
  at Luminate) at time of training, not before.
- **Kim Tanner is running dedicated HELOC-review alignment sessions** with AEs plus Ashwin [Dayal], Marc
  [Kaplan], and Sean, because partner-side underwriting/sales-manager contacts are surfacing HELOC questions the
  Retail team isn't yet uniformly prepared to answer (2026-08-20).
- **First evidence of a physical-presence sales motion under the new Retail structure:** Brandi Cirell drafted a
  target list of geographically clustered LOs (Phoenix/Scottsdale, San Diego) for in-person lunch-and-learns,
  cross-referencing which offices already have active users (2026-08-20).
- `#proj-retail-wholesale` itself carried no new substantive findings — it's a thin channel with links to
  Tulli's own Google Sheet analysis and a Granola meeting note, not discussion.

## 8. The wholesale/broker allocation — the actual numbers (added 2026-08-29, from Drive)

**Sources:** `Broker/Retail Sync Prep — Aug 19 2026 (FINAL)` (Google Doc, owner
nicholas.santulli@, Drive modified **2026-08-19**) · `Aug 13, 2026 | Broker/Retail Transition
Working Session` (Google Doc, owner kim.tanner@, modified **2026-08-17**).

§7 above covered the Slack-visible layer. These two docs carry the **allocation arithmetic and
the named blockers** that never reached Slack.

### Book sizes before the split (Aug 3 company-grain snapshot)

| Current AE | Broker cos | Onboarded LOs | Producing LOs | Apps | Broker IR closes |
|---|---|---|---|---|---|
| Marisa | 394 | 1,529 | 443 | 3,768 | 731 |
| **Tejas** (retail stripped) | 404 | 1,250 | 449 | 3,201 | 743 |
| TJ | 215 | 726 | 226 | 1,788 | 395 |
| Kara | 88 | 123 | 47 | 305 | 84 |
| Brian | 34 | 66 | 9 | 114 | 10 |
| Tierney | 17 | 24 | **2** | 41 | 7 |

🔑 **The spread is ~64:1 on onboarded LOs and ~220:1 on producing LOs.** Tierney enters the
reorg with 2 producing LOs; Marisa has 443. Any "balanced" allocation is really a rebuild.

### Proposed ending state (Marisa → Tierney/Kara, TJ → Brian, Tejas held)

| Proposed AE | Companies | Onboarded LOs | Producing LOs | Apps | IR closes | Largest acct |
|---|---|---|---|---|---|---|
| Tierney | 211 | 838 | 238 | 2,127 | 400 | **41.1%** 🔴 |
| Kara | 288 | 838 | 254 | 1,987 | 422 | 14.7% |
| Brian | 249 | 792 | 235 | 1,902 | 405 | 10.0% |

Spread: 5.5% onboarded, 7.5% producing. Balanced on **ending** state, allocated at
**company-family** level. File: `wholesale_allocation_2026-08-19.csv`, 1,013 rows, HubSpot-ID keyed.

🔴 **NEXA concentration.** 344 onboarded LOs = **41% of Tierney's proposed book**, given to the
AE with the smallest starting book (2 producing LOs). Three options were tabled (Tierney takes
it as named primary / it goes to Kara / named-account coverage with a defined backup outside
the split). **No decision is recorded in either doc.**

### 🔴 Data blockers that gate the cutover

**14 companies would split across two AEs** because duplicate CRM records sit in different
books: NEXA (352 LOs, Tierney+Brian), C2 Financial (68), Loan Factory (36), The Loan Store/TLS
(21, three-way), First Class Mortgage (14), plus Equity Smart, Swift, CTC, Anchor, Crosspoint,
Greenlight, Pacific Community, JTS, Home Financing Direct.

> **Rule needed, unowned:** *one real-world company = one AE, regardless of how many CRM records
> exist.* Without it, two LOs at the same shop route to different AEs depending on which record
> their application attached to.

**Live HubSpot findings (read-only, 2026-08-19):**

| Company | LOs | Finding |
|---|---|---|
| **CMG Mortgage** | 89 | **No HubSpot company record exists.** Nearest is CMG Financial `51219869366`, typed Lender. Also missing from the Aug 13 pipeline extract. |
| **Turnkey Foundation** | 64 | Record `55511822705` — no pod, no company type, no partner type. **UNROUTABLE.** |
| First Coast | — | 4 records. `18344779607` = Pod 2 / Mortgage Brokerage. `19507767060` = Pod 1 / Lender. Same name, **opposite classification AND opposite pod.** |
| Coast2Coast | 19 | 9 records (was 4 on Aug 17), incl. unrelated businesses auto-created from domains. Real record `50740887391`. |
| Loan Factory | 34 | `863857775` = Pod 2; `46565812141` = Pod 1. Split-brain routing today. |
| Barrett, Xpert, Movement, APM, Equity Smart | — | All have branch/duplicate records. APM has 4, Xpert has 3. |

**250 of 1,013 allocation rows have no HubSpot company ID** — almost all 1–4-LO tail records,
but none can be enabled for company-grain routing until matched or retired.

🔑 **Root cause, named:** *"HubSpot auto-creates companies from email domains, so every LO with
a personal site spawns a branch record. That's why duplicates grow month over month — this
won't stop on its own."* This is the mechanism behind the duplicate-company pain in §3 and
[[hub-mirror-gotchas]]; it is a **standing generator**, not a backlog.

### Hybrid classification — why no single field can route

The Aug 17 rule was **primary operating behavior, not license type**. Live field values show why:

| Company | HubSpot ID | Type field | Partner type | Pod | Proposed |
|---|---|---|---|---|---|
| American Pacific Mortgage | 51032050828 | Lender | Brokerage Partnered w/ Wholesaler | 2 | RETAIL |
| The Mortgage Calculator | 19507731524 | Lender | Brokerage Partnered w/ Wholesaler | 3 | Retail-leaning |
| Change Home Mortgage | 6691484571 | Lender | *(blank)* | 1 | needs call |
| Coast2Coast Mortgage | 50740887391 | Lender | Brokerage Partnered w/ Wholesaler | 2 | BROKER |
| Equity Smart | 4569305159 | Mortgage Brokerage | Brokerage Partnered w/ Wholesaler | 2 | BROKER |
| Northwest Funding Group | 15607908380 | Mortgage Brokerage | Brokerage Partnered w/ Wholesaler | 2 | BROKER |
| GoRascal | 15607413399 | — | — | — | BROKER (lender license, wholesale behavior) |
| Loan Factory | 863857775 | Mortgage Brokerage | Brokerage Partnered w/ Wholesaler | 2 | BROKER |
| First Coast | 4 records | **CONFLICTING** | *(blank)* | **1 AND 2** | resolve identity first |

⚠️ **Explicit instruction in the source:** *"store the result in one canonical company-level
field. Do not rebuild routing on `hl_lender_sales_enterprise_partner_type` — it's blank on
Change Home and First Coast."*

**Fuller hybrid lists from the Aug 13 session** (leans-Retail: Allegiance Home Lending, City
First, **CMG Mortgage/Home Loans — 75% retail in 2025**, The Gibraltar Group *(CMG JV)*, GO
Mortgage, Gold Star, Fidelity Direct, OneTrust, Lower.com, The Mortgage Calculator;
leans-Broker: Loan Factory, Change Home, Coast2Coast, Ease, Equity Smart, First Coast,
GoRascal, Lions Capital, LoanHaus, Northwest Funding). Three of the leans-Retail set (CMG,
Gibraltar, OneTrust) sit **under TLS today** and are flagged as needing to switch.

### Account pass-off list (Aug 13)

| Direction | Accounts |
|---|---|
| Labrada → Kim (Retail) | APM · RWM *(done)* · Envoy · PRMI |
| Kim → Labrada (Broker) | NEXA · Barrett *(done)* · GoRascal · Loan Factory · LoanHaus · Orion Mortgage *("small but ELITE")* |

Excluded from the questionable-accounts pull: **builders, UWM, TLS, Orchard**.

### 🔴 The Tejas residual gates the date

Residual broker book: **1,250 onboarded / 449 producing LOs.** Three scenarios, none chosen:

| Scenario | Result |
|---|---|
| A — 4th primary AE | ~49% larger by onboarded, ~83% larger by producing, than the others |
| B — redistributed across the three | Each lands at ~1,250 onboarded / 393 producing |
| C — overlay | Needs **3 separate fields**: routing owner, relationship owner, comp credit |

**Scenario B raises every proposed book by ~50%** — i.e. the allocation table above is not final
if B is chosen. The doc states plainly: *"This gates the Sept 1 date, not just the org chart."*
Tejas was **deliberately not invited** to the Aug 19 sync, "per Aug 17 decision, hold until Tate
confirms his role."

### 🔴 Open decisions carried into the cutover week

1. **Unlisted retail lender owner — Nick vs. Kim.** Granola says Nick; the working doc says Kim.
   **Data Bridge routes a Nick-owned company directly to Nick** — live routing behavior, not a
   cosmetic field. (The Aug 13 session recorded Kim as company owner/AE for unlisted retail.)
2. **AM scope is internally contradictory.** Brandi/Deb/Barb cover ~53% of 2026 Retail
   application rows; the other **47.4%** sit with Ian, Kyle, Ashlee, Angelica, Carter. *"'Three
   Retail AMs' and 'preserve relationships' are incompatible — pick one."*
3. ~10 hybrid lenders need Retail→Broker recategorization.
4. Portal contact card — add Closing Manager, define blank-role fallback.
5. 🔴 **Comp-period locks are NOT deployed** (branch `reorg/comp-period-lock`). Until shipped,
   any repod can **destructively recompute closed comp periods**. Named a *"hard no-go gate for
   automated cutover."* → [[comp-plans]] §7, [[risk-register]].

### The calendar problem, stated in the source

Sept 1 was 12 days out at the Aug 19 sync; Tulli was OOO 8/20–8/27, back 8/28 — **~two working
days between return and go-live.** Options offered: decisions land that day, a named owner per
open decision, or the date slips. Also recorded: *"Reading John: UWM is at 9 contracts (would be
about 31 at last month's pace) and he's being asked to double channel contract volume."*

## 2026-08-29 — from the Data Bridge knowledge base (KB, source: `knowledge_documents`)

New payment-ops and listing-agent facts from the compiled Data Bridge KB (see
[[data-bridge-kb-index]] for the full corpus) — companion detail to §4's Slack-sourced Trolley
findings, not a replacement. Stated (compiled ops answers), not code-verified here.

- 🔑 **LO payment cadence mechanics (adds to §4):** LOs self-enter tax/ACH info in the Trolley
  portal; the payment service **reviews submitted tax info on Tuesdays** and **sends payments on
  Thursdays**. Eligibility depends on payment plan + deal milestone (commonly triggered at **DR
  Closed**). Company-paid plans (e.g. Edge) pay the **company** first, which then distributes via
  payroll — not a direct Trolley payout to the LO. ("LO Payment Cadence via Trolley")
- 🔑 **Payment-redirect verification is a two-step gate**, not a courtesy check: default routing
  follows the LO named on the transaction file, and *any* request to redirect payment to someone
  else must be (1) verified as authorized with the BBYS Payments team **and** (2) confirmed with
  the listed LO — before any change is promised or made. ("LO Payment Recipient Verification
  Guidance")
- **Backup-withholding pre-activation check (adds detail to the 24% mention in §4):** the
  BBYS Payments team should check the Trolley backup-withholding flag on a partner's profile
  *before* activation, not after. If flagged, confirm with the partner whether it's intentional
  (payouts cut 24% while active); if it was a mistake, hold activation and route the partner back
  through the LO/Trolley portal to correct tax info first. ("LO Payment / Trolley: Backup
  Withholding Guidance")
- **Partner/company payment arrangements require a signed partnership agreement first** (e.g.
  Barrett) — payments do not start automatically on deal closings. Once signed, prior eligible
  closings from the negotiation window can be backpaid. Never promise a disbursement date to a
  partner without checking with payment operations first. ("Partner Payment Arrangements:
  Partnership Agreement Prerequisite and Backpay Process")
- **LO referral-credit payout sequence** (distinct from referral-credit *eligibility*, which is a
  separate rule set): the departing-home closing must be recorded first, reconciliation runs
  **Thursdays**, and payment follows **the week after**. ("LO Referral Credit Payout Cadence &
  Reconciliation Timing")
- 🔑 **Listing-agent boundary (new topic, not previously in this doc):** HomeLight does not
  mediate listing-agent disputes or give listing-agreement advice — direct the borrower to their
  agent's broker. The only BBYS requirement is that the departing home stays MLS-listed; the
  specific agent is secondary. If the borrower switches agents, confirm the prior relationship is
  resolved, collect the new agent's contact info, and update Sales App immediately.
  ("Listing Agent Changes & Dispute Boundaries – BBYS Guidance"; "Handling Listing Agent Changes
  for BBYS Borrowers")
- 🔑 **Listing-agent compensation when the DH doesn't sell in the standard period:** a 60-day
  extension (+1.2% program fee) may be granted; if still unsold by day 121 and HomeLight
  acquires the property, **the listing agent is NOT paid at acquisition** — the listing agreement
  survives, the agent keeps marketing the home for HomeLight, and is paid at the **eventual
  resale**. Worth adding to agent-facing BBYS FAQ materials per the source article.
  ("BBYS: Listing Agent Compensation When Departing Home Doesn't Sell in Standard Marketing
  Period")

### 2026-08-29 addendum (LOOP-2 KB sweep — remaining ~290 title-only rows read via the compiler's
own `summary` column, not full content_text; see [[data-bridge-kb-index]] for method)

- 🔑 **HOI declarations page ≠ Evidence of Insurance (EOI).** These are distinct documents;
  operators must request the full declarations page when coverage detail is needed for review.
  Any prior one-off EOI acceptance was an **exception**, not a reusable precedent — a recurring
  LO confusion point worth flagging in operator training. ("HOI Documentation: Declarations Page
  vs. Evidence of Insurance (EOI)")
- **Departing-home inspection gating (adds to §5/§6):** the inspection link only activates after
  BOTH an executed purchase contract AND a signed HomeLight agreement are in place; clients
  generally pay nothing upfront (HomeLight recoups from DR sale proceeds). Full inspection
  reports are required — third-party summaries and appraisals are NOT acceptable substitutes.
  Recency window for prior inspection evidence is 60 days, exceptions only via Approvals; the
  inspection itself must complete at least 21 days before closing to avoid closing-timeline risk.
  ("Departing Home Inspection: Timing, Costs, and Repair Classification"; "Departing-Home
  Inspection Reports: Full Report Required"; "Departing Home Inspection Requirements: Appraisal
  vs. Inspection & Recency Window"; "Departing Home Inspection Timing Guidance"; "Inspection
  Payment Guidance: No Upfront Cost for Clients")
- 🔑 **Backup-purchase reimbursement mechanics:** if HomeLight ends up buying the departing home,
  carrying and resale costs (taxes, insurance, utilities, HOA, closing costs, etc.) are reimbursed
  from DR sale proceeds **before** the client receives any remaining profit. Frame the normal
  market sale within the program window as the goal — the backup purchase is a rare safety net,
  not a routine outcome. ("Backup Purchase: Reimbursement of Carrying and Resale Costs")
- **BBYS fee taxonomy clarified:** the 5% default/late fee applies only on borrower default or bad
  faith — it is NOT an automatic consequence of HomeLight buying the departing home at day 121.
  Extension fees and backup-purchase fees are the ordinary post-120-day mechanisms and should not
  be conflated with the default fee in LO-facing FAQ materials. ("BBYS Fee FAQs: Distinguishing
  Default/Late Fees from Extension and Backup-Purchase Fees")
- **HomeLight's BBYS program fee must never appear in incoming-residence lender disclosures** —
  it's collected after the DR sells and is unrelated to the IR closing/loan-amount mechanics.
  Flagged as needing explicit LO training to prevent a compliance-adjacent disclosure error.
  ("BBYS Program Fee: Exclusion from Incoming-Residence Lender Disclosures")
- **Fee true-up vs. Sales App recalculation are different events:** the final fee true-up happens
  at payoff; updating the DR final sale price mid-file can prematurely trigger a Sales App program-
  fee recalculation and force document regeneration. Avoid updating the final sale price unless
  approval/document teams have intentionally decided to reissue. ("BBYS Fee True-Up vs. Sales App
  Recalculation: Closing Document Guidance")
- **Cash-overage exceptions above the normal threshold require title-company feasibility
  confirmation first**; if a program fee increases after the borrower has already signed,
  operators must verify the LO prepared the borrower before applying the updated fee. ("Cash
  Overage Exceptions and Fee Changes: Operator Guidelines")
- **LO referral credit is $1,500**, paid after the departing residence sells, but ONLY when the
  purchase loan is brokered through a HomeLight partner channel (The Loan Store or UWM) — a
  narrower eligibility gate than the payout-cadence mechanics already documented above (Thursdays/
  next-week). Don't conflate "eligible" with "will be paid this cycle." ("LO Referral Credit:
  Eligibility and Payment Conditions")

## Open questions

- Is the `weighted_round_robin` fallback logic (§1) the same mechanism that produces `#no-lsm-lrm-fallback-alerts`,
  or are these two separate systems that happen to fail on the same underlying gap (missing company LSM data)?
  Not confirmed from Slack alone — would need a HAPI/Data Bridge code check.
- What is Jose Herrera's routing-rework scope and timeline (§1)? Only a channel-join and one contextual mention
  were observed; no design doc or PR was surfaced in this sweep.
- Is there a target date to fix the Trolley bulk-payment tool (§4) or is manual invoice entry the accepted
  workaround indefinitely?
- Does the RAV market-adjustment directive pattern (§5) get captured anywhere durable (a shared doc, a
  Periscope note) or does it only exist as scrollback in `#approvals-valuations-team`? If the latter, new PH
  analysts have no way to discover standing guidance.
- Who owns closing the comp-plan gap for CMG/Luminate (§7) — Kim Tanner, or does it sit with a HomeLight
  partnerships/legal function not visible in this channel set?
- **Was the NEXA concentration decision ever made?** (§8) Three options tabled 2026-08-19, none
  recorded as chosen. 41% of one AE's book hangs on it.
- **Which Tejas scenario (A/B/C) was chosen?** (§8) B invalidates the published allocation.
- **Was the "one real-world company = one AE" rule ever given an owner?** (§8) It was explicitly
  asked for in the room and left unassigned.
- **Does CMG Mortgage (89 LOs) now have a HubSpot company record?** (§8) It had none on 2026-08-19
  and is one of the largest books in the split.
- **Has the mandatory AM 4-hour intro-call SLA (§4) shipped a tracker?** Jake Vogel asked for one
  2026-08-06; no dashboard or workflow surfaced anywhere else in the vault as of 2026-08-29.
- **Where would executed partnership agreements actually live if centralized?** (§4) Legal's
  2026-06-23 urgent ask surfaced the gap; no owner or target system was proposed in the thread.
