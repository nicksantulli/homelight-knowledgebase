---
last_updated: 2026-04-03
type: context
source: HomeLight-Vault/context/deal-operations.md
imported: 2026-09-29
---

# BBYS Deal Operations & Escalation Patterns

> How deals flow through Slack channels, who does what, and common escalation patterns.

## Deal Channel Lifecycle

Every BBYS deal gets a Slack channel, auto-created by Sales App. Channel names encode partner, stage, client, address, and state. **Renamed at every stage transition.**

| Stage Prefix | What Happens |
|-------------|-------------|
| new- | Lead submitted. TitleBot pulls title. Lead notification posted. |
| rev- | Property photos uploaded. "Tasked for valuations with X hour turnaround." |
| appr- | Conditional agreement generated. LO emailed approval. |
| as- | DocuSign completed. Auto credit check. @lender-ops-team tagged for HOA/title review. |
| iruc- | IR under contract. AI reviews contract, extracts fields. Inspection ordered. Real transaction work begins. |
| Closing | Wire instructions, EUIDs, Encompass setup, backup offer, notary, funding, wire release. |

## Who Does What in Deal Channels

| Role | People | What They Do |
|------|--------|-------------|
| LRM | Deborah, Ashlee, Kyle, Ian, Brandi | LO communication, relay deal context, request rush, flag issues |
| Lender Ops | Michelle Guarino, Jessie Guiang | HOA/title review, task for valuations, agreement generation |
| Closing | Brent Lehman, Derek Lupien, Cheryl Funk, Madeline Cypert | Wire instructions, EUIDs, Encompass, notary, backup offers |
| Property Condition | Hugh Rodman, Jarrod | Inspection review, repair requirements, escalations to Jason Smith |
| Listing Ops | Candice Jenkins, Carl Giordano, Christy Meek | Backup offer review, listing-side operations |

## Escalation Paths

### Fee Exceptions (Most Common)
- LRM or Brandi tags **Jake Vogel** in deal channel
- Typical: variable pricing, flat fee, capped percentage (1.7-2%)
- Jake evaluates: EU as % of valuation, LO relationship value, deal economics
- Trigger: LO fee pushback, competitive pressure (UpEquity, FlyHomes)

### HELOC/EB Confusion
- LOs apply for wrong product (HELOC vs asset EB)
- Asset upload blocked when both selected — workaround is manual upload
- EB approval fires before approval team review (known bug — Deb flagged as "very bad")
- **Jacquelyn Houston** handles HELOC eligibility and state restrictions

### Valuation/CLTV Disputes
- Closing specialists flag calculation errors
- **Nick Friedman** gets pulled in for backend data fixes
- Root cause often: econ model showing different balance than actual statement

### Property Condition Escalations
- Dedicated channel: #property-condition-escalations
- Common: foundation, water damage, mold, plumbing, roof
- Submitted to **Jason Smith** for review
- Can delay closing, change deal economics

### Rush Requests
- "Super rush" / "911" / "mad rush" in channel
- Triggers: imminent COE, expired financing contingency, competitive pressure
- Tasking speeds: 4-hour turnaround (rush) vs 24-hour (standard)

### Notary/Signing Issues
- Wrong package in Snapdocs, split signing scenarios, POA/trust handling
- RON being built into client portal (builder deals first)

## Operational Bottlenecks

1. **Photo submission** — 120 apps stuck. Building AI photo crawling as alternative.
2. **LO portal login friction** — multiple auth code requests per deal
3. **Agreement invalidation loops** — partner/DTI changes invalidate unsigned agreements
4. **Missing info at intake** — agent info, additional emails, DR agent consistently missing
5. **Wire instruction delays** — manual tracking of title company responses
6. **HELOC closeout docs** — additional friction (statement, closeout letter, 1003, credit report)

## LO Communication Patterns

- LRMs are primary LO touchpoint (call, text, email)
- Automated emails drive much of process communication
- First-time LO noted in channels: "LO's first BBYS — may need hand holding"
- LO sentiment tracked: "ecstatic", "freaking out", "upset", "confused with tech"
- Competitive dynamics surface: fee exception requests reference UpEquity, FlyHomes

## Exception Decision Authority

| Type | Decider | Frequency |
|------|---------|-----------|
| Fee exceptions | Jake Vogel | Very common |
| Rush tasking | Lender Ops (Michelle, Jessie) | Very common |
| Property conditions | Jason Smith | Common |
| HELOC eligibility | Jacquelyn Houston | Growing |
| CLTV corrections | Nick Friedman | Moderate |
| EB amount disputes | Scott Farress / approval team | Moderate |


---

## 2026-08-29 — The deal-failure / disposition taxonomy (from Drive)

**Source:** `Deal Failure / Deal Disposition Review - Optimization` (Google Sheet
`1htwzpbz69-_OxZYS9ycw1SdOd8QuwERS83YcSj6WKwc`, owner javy@homelight.com, Drive modified
**2026-08-26**; shared with Tulli 2026-03-27, referenced by Joel in Slack 2026-08-19).

> 🔑 **This is the live design of the failure-reason picklist rework** — not a description of
> today's picklist. It is the reconciliation of **two divergent vocabularies** (HubSpot's
> `Reason for Fail` and the Sales App's `Deal Disposition Reason`) down to a condensed set,
> per Sumant's direction. Treat the 13-reason table as **proposed**; the two raw lists as
> **current**. Another entry for [[stage-vocabularies-master]]'s "six+ vocabularies" problem.

### The problem it solves

| Vocabulary | Where | Shape |
|---|---|---|
| `Reason for Fail` | HubSpot deal picklist | 24 options, snake_case internal values (`approval_denial_acreage`, `canceled_dr_offer`, `equity_unlock_too_low`, `used_competitor_bridge_loan`, …) |
| `Deal Disposition Reason` | Sales App picklist | 26 options, prose labels ("Approval denial: Projected DOM too high", "Offer on DR (able to close prior to IR COE)", …) |
| Condensed target | Proposed | **13 reasons in 6 families**, per Sumant |

The two lists are near-parallel but **not** a clean mapping — the Sales App carries
`Canceled - Title issue/s` with no HubSpot equivalent, and HubSpot carries `Not Qualified for
IR Financing` / `Test Lead` / `Unresponsive` phrased differently. The sheet marks items in red
("remove — no actionable insight, forces LSMs to dig deeper") and yellow ("add to HS picklist").

### 🔑 The 13 condensed reasons, with owners

Each reason carries a definition, a "when to select this," and a coaching prescription — this
is a **coaching taxonomy**, not just a reporting one.

| Family | # | Reason | Sub-reason owner |
|---|---|---|---|
| **Client didn't need or want BBYS** | 1 | Client transacted without BBYS | Jake |
| | 2 | Client decided not to move | Jake |
| | 3 | **Lost to competitor** | Jake |
| **DR didn't qualify** | 4 | DR marketability concern | **Jason** |
| | 5 | DR property ineligible | **Jason** |
| **Economics didn't work** | 6 | Insufficient equity unlock | Jake |
| | 7 | Fee or cost objection | Jake |
| **Financing or timing** | 8 | Borrower didn't qualify for purchase financing | Jake |
| | 9 | DR sold during BBYS process | Jake |
| **Disengaged — last resort** | 10 | **LO stopped responding** | Jake |
| | 11 | Borrower unresponsive to LO | Jake |
| **Administrative** | 12 | Test Deal | n/a |
| | 13 | Duplicate | n/a |

### The distinctions that actually matter

- **#1 vs #9** — both end with the client not using BBYS, but #1 is *never committed* and #9 is
  *committed then overtaken*: "signed BBYS Agreement, there was an executed GBO," but the DR
  closed first. #9 is explicitly rated **low coaching priority** ("a timing outcome, not a
  pitch failure"); #1 is a pitch failure. Conflating them destroys the coaching signal.
- **#4 vs #5** — market conditions (DOM, comps) vs *what the property is* (type, size, zoning,
  condition). Both are "denial," different fixes: #4 → DOM/comp thresholds; #5 → property
  approval matrix.
- **#10 vs #11** — *who* went dark. #10 is a **relationship problem with the LO** and escalates
  to the Lender Relations Manager as a churn signal; #11 is the LO's client going cold and is
  coached with client-facing content. The sheet calls #10 "last resort" and instructs
  reclassification if the LO re-engages.

### Cross-reference rules baked into the taxonomy

- **#6 Insufficient equity unlock** must be cross-referenced against **Equity Boost / HELOC
  Boost application status**: *"If Boost NOT applied → LO likely doesn't know about Boost
  products"* (coachable). *"If Boost applied and still short → economics genuinely might not
  have worked"* (not coachable). 🔑 A raw count of reason #6 is **meaningless without that
  join** — see [[bbys-equity-boost]] · [[heloc-product]].
- **#7 Fee objection** names the fee as **2.4%** and prescribes the pricing-exception path
  ([[bbys-unit-economics]] §2, and Jake as fee-exception decider in the table above).

### Competitor set named in reason #3

Knock · UpEquity · Homeward · Calque · Lendsure · FlyHomes · Zavvie · Cash for Keys ·
Smart Move Guarantee · QuickBuy · Traditional Bridge Loan — **plus "loan officer lost client to
a different LO,"** which is categorised as a competitive loss. Assets requested but not yet
built: Knock and FlyHomes calculator links, trad-bridge-loan scripting/collateral (SOSS),
"calculator work from Ashwin / Marc." Compare [[competitors-bbys]] — this list is the
operational one reps actually pick from.

### ⚠️ Caveats

- The "Assets to surface" column is **mostly empty or written as questions to self** ("did you
  ask the LO if they are ok doing a standalone HELOC…", "ask jake / CA for verbiage"). The
  enablement content this taxonomy assumes **does not all exist yet**.
- Reason #9's sub-reasons list ("title issues discovered during full title/lien search,
  inspection issues") does not match its own definition (DR sold first). The sub-reason column
  looks like it drifted during editing. Do not build a sub-reason report off it as written.
- No effective date, no ship confirmation. Owner javy@; last touched 2026-08-26.

## Related Notes
- [[bbys-edge-cases]]
- [[stage-vocabularies-master]]
- [[competitors-bbys]]
- [[bbys-unit-economics]]
- [[projects/2026-08-29-drive-mining]]
- [[heloc-product]]
- [[team]]
