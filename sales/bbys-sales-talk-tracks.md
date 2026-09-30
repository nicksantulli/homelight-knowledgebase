---
last_updated: 2026-08-29
status: current
source: Data Bridge production Supabase (project reqtzlabsmuqywvxluqb), table `aircall_calls`.transcript_text — read-only sample of ~55 calls (longest-duration + keyword-matched), March–August 2026
scope: how LSMs/AEs and LRMs actually pitch, explain, and defend BBYS on live Aircall calls — talk tracks, objection handling, FAQ as delivered
source: HomeLight-Vault/context/bbys-sales-talk-tracks.md
imported: 2026-09-29
---

# BBYS sales talk tracks — from the call transcripts

Built from real Aircall transcripts, not scripts or decks. Quotes are verbatim (transcription
artifacts like broken turn-taking left in where relevant to show real-call texture; cleaned up
only for readability of the point being made). Every claim below is **observed** (transcript
text) unless marked otherwise. Read with [[aircall-system]] (the scoring pipeline that produces
this data) and [[2026-08-29-aircall-mining]] (corpus stats, friction findings).

## Two different calling populations, two different call shapes

The Aircall roster splits into roles with materially different call content (mapped via
[[team]]):

| Role | Reps sampled | What their calls sound like |
| --- | --- | --- |
| **LSM/AE** (pre-sale, LO-facing) | Richie Helali, TJ Sims, Tejas Narkhede, Marisa Drake, Kara Kleingarn, Brian Banes | **Pitch calls** — product explanation, terminology, pricing, competitor comparison, objection handling. Longest calls run 60–85 min and are effectively live spreadsheet walkthroughs with an LO. |
| **LRM** (post-signature, deal-servicing) | Ian Pardo, Barb Griego, Deborah Shutt, Kyle Bradish, Evan Jaquias-Johnson, Carl Giordano, Brandi Cirell, Angelica Espinosa, Carter Marks, Ashlee Kim | **Deal-mechanics calls** — funding timing, payoff math, extensions, maintenance reserve, repairs. Talk about a specific live deal, not the product in the abstract. |
| **Listing Ops** (a subset also tagged LRM in Aircall) | Evan Jaquias-Johnson, Candice Jenkins, Christy Meek, Carl Giordano | Property-condition/repair-vendor calls (Orchard Concierge scope, inspection findings). |

🔑 This split is why "how BBYS is pitched" and "how BBYS funding/payoff actually works" show up
as different call populations, not different sections of the same call — LSMs rarely explain
payoff mechanics in depth, LRMs rarely re-pitch the product from scratch.

## The core pitch structure

Consistent across LSM reps (Tejas, Richie, TJ), the standard explanation follows this order:

1. **Baseline approval** — "we're gonna land around 70% on the house itself, sometimes 75%"
   (Tejas Narkhede, call `3609942284`, 2026-03-19).
2. **Two ways past baseline to 90%** — asset-based (retirement, brokerage, savings, gift funds
   held as reserves) *or* a second-position HELOC on the new home. Reps explicitly name both as
   **"Equity Boost."**
3. **DTI drop as the lighter-weight fallback** — offered when the client doesn't need cash out,
   only needs the departing home excluded from DTI: *"There's DTI drop, which is just the DTI
   solution... All that does is it helps you omit [the debt] of their current property from
   their debt to income ratio... it's just 1%."* (Tejas, `3609942284`)
4. **Variable pricing** as the fee lever for expensive/fast-moving markets (below).
5. **Competitor differentiation**, usually only if the LO raises it.

### Variable pricing pitch (near-identical talk track across reps)

> "With variable pricing... if you sell your old home within the first sixty days, the fee will
> be discounted down to 1.5%... For every day it goes on after sixty days, it'll go up [~15bps/day
> toward 2.4% cap]." — Richie Helali, `3715124831`, 2026-04-24

> "The other alternative is a variable pricing model... they don't wanna pay 2.4% because they
> think they're gonna sell in a week. So for those folks, we can do a fee that starts lower at
> 1.5%..." — Tejas Narkhede, `3609942284`

Barb Griego frames eligibility explicitly as a **negotiation lever gated by home value**: *"the
only time... that fee is negotiable is when the value of the home goes... over a million
dollars."* (`4072653069`, 2026-08-21) — consistent with the **$425,000 eligible-minimum-valuation**
gate on variable pricing already documented in [[bbys-pricing-engine]], though reps talk about it
as a "million dollar" rule of thumb, not the $425K floor.

### Competitor comparison talk track

Richie Helali's stock opener when an LO brings up competitors: *"there are other companies that
offer something similar... Fly Homes, Knock... There's also one called Cal[que]."* (`4065357448`,
2026-08-19). The differentiation he leads with is **extension flexibility**:

> "NOC [Knock], initially, they give six months before they step in and buy it. But the
> difference is they're not gonna give an extension on six months... Whereas with us, we're
> talking to the agent. [We can] extend it. Not a big deal. NOK is a strict six months. If you
> don't sell in six months, NOK will buy at a predetermined price." — Richie Helali, `4065357448`

⚠️ **"Knock" is transcribed inconsistently as "NOC"/"NOK"** across calls — an artifact of
speech-to-text on an unfamiliar proper noun, not a real product name variant. Grep transcripts on
`nok|noc|knock` together if searching for competitor mentions.

Calque appears in only 2 of ~37K transcribed calls — essentially never comes up live, despite
being in the official competitor picklist ([[repo-hapi]] § competitor picklist). Homeward appears
53 times but nearly always as a **mis-hearing of "HomeLight"** by the transcription engine, not a
genuine competitor mention (see samples in [[2026-08-29-aircall-mining]]) — treat raw "Homeward"
hit counts as unreliable.

### Credit and program-access qualification

> "What's the credit score? ... 620. And here's the funny thing about our qualification on your
> credit score. For a normal bank... they're looking for a 620 mid, and then all of a sudden
> there's all these other credit score requirements... For us, we just need to see that they have
> a score that is 620." — Richie Helali, `4021747368`, 2026-08-04

> "Do the clients need a VA, FHA... government? ... No. There's — you can't do VA and FHA." — TJ
> Sims call, LO asking directly, `3858084784`, 2026-06-11

**No appraisal is required** — reps state this as a differentiator repeatedly and directly:
*"Do you need an appraisal? ... No appraisal needed... on yours right now... we're gonna need
some photos of the home"* (Richie Helali, `4021747368`). Approval is desktop/photo-based, not
appraisal-based.

## Objection handling, as actually run

| Objection | How reps respond | Example |
| --- | --- | --- |
| **"That's too expensive" (2.4% fee)** | Pivot to variable pricing, or note internal fee cuts already made for lighter products (DTI-drop-only cut to ~1%: *"why should we charge somebody the full 2.4% if we're just... providing them literally a piece of paper?... we can get this done at 1%"*) | Richie, `3732875760`, 2026-04-30 |
| **"[Originating LO is] not interested in that part of it"** | Firm, not soft: *"if he's not interested in home life [HomeLight], he's not gonna be able to do the mortgage period"* — program participation is bundled with the purchase-side loan, not optional | Marisa Drake, `3836269678`, 2026-06-04 |
| **"They already have a lender"** | Never treated as disqualifying. Reps ask for the lender's name to check onboarding status, or offer to route to a partner LO if the existing one won't onboard | Evan Jaquias-Johnson `3644165871`; Brian Banes `3714863921` |
| **Agent-stripping fear ("nobody told me")** | Reps proactively name the failure mode and draw a hard boundary before it happens: *"the agent's always gonna be the borrower's choice"* — framed as an automatic non-negotiable, not case-by-case | Richie Helali, `3715124831` |
| **Fee/cost surprise late in the deal** | Reps clarify mechanics reactively (maintenance reserve refund logic) but **do not always get ahead of disclosure timing** — see friction pattern below | Evan Jaquias-Johnson, `4077923012` |

## FAQ, as answered live

| Question | Answer as delivered | Source |
| --- | --- | --- |
| What's the minimum credit score? | 620 mid FICO; no extra credit-history overlays like a bank runs | `4021747368` |
| Is an appraisal required? | No — photo + desktop valuation only | `4021747368` |
| Equity unlock vs. Equity Boost — same thing? | **No.** "Equity unlock is our internal verbiage for the bridge loan." Equity Boost is the *mechanism* to increase how much of that bridge loan you can access (asset-based to ~85%, or HELOC-based to ~90%) | TJ Sims, `3858084784` |
| Can VA/FHA borrowers use it? | No — explicitly excluded | `3858084784` |
| What if the home doesn't sell in the program window? | Extension available (negotiated case-by-case), vs. a hard buyout deadline for at least one named competitor | `4065357448` |
| When does HomeLight actually send money? | Wired to escrow/title company the day of or day before closing — **not** disbursed at approval. No cancellation fee if client backs out before that point | Tejas Narkhede, `3609942284` |
| What happens to unused maintenance reserve? | Refunded/credited back against the payoff statement if not used | Evan Jaquias-Johnson, `4077923012` |
| Does the LO still have to be involved for DTI-drop-only (no bridge loan)? | Yes — LO still coordinates the purchase-side loan regardless of which BBYS variant is used | multiple LRM/LSM calls |
| How is the HELOC-based Equity Boost priced so it doesn't wreck the new-purchase CLTV? | The HELOC is originated at **$0 draw** at closing, so it counts as LTV not CLTV on the purchase loan until actually drawn | TJ Sims, `3858084784`; Tejas, `3609942284` |

## Lending mechanics explained on calls (practical, not marketing)

- **Funding is not disbursement.** Approval → signed agreements → *no money moves* until the
  wire to escrow "the day of or the day before closing." Cancel any time before that with no fee.
  (`3609942284`)
- **Guaranteed backup offer = sum of two payoffs.** *"What determines our backup offer is gonna
  be the loan payoff values... their borrower's first mortgage plus our bridge loan."*
  (TJ Sims, `3858084784`) — matches the model in [[bbys-unit-economics]].
- **Second-lien HELOC mechanics**: originates in second position behind the new-home purchase
  loan, "no interest, no monthly payments for six months" teaser, $0 draw at origination so it
  doesn't count against CLTV until used. (Richie Helali, `3769597201`; TJ Sims, `3858084784`)
- **Payoff-at-close waterfall** (LRM call walking a client through the actual numbers): equity
  unlock amount + HELOC draw = cash to close on the new property; separately, program fee +
  maintenance reserve + inspection fee are rolled into what's withheld/settled at the DR sale.
  (Barb Griego, `3880334399`)
- **120-day clock, then two paths**: extension, or HomeLight buys the departing residence
  outright at the guaranteed price. Reps are explicit that this is **not** the same as a
  competitor's fixed-window forced buyout — extension is discretionary and routinely granted.
  (Evan Jaquias-Johnson, `4077923012`)

## 🔴 Recurring confusion and complaint patterns (product friction signal)

- **Equity Unlock vs. Equity Boost naming collision is a real, recurring problem — not just a
  first-time-caller issue.** In one 77-minute call, an LO who has clearly worked multiple BBYS
  deals states plainly: *"I still don't know what Equity Boost is and when that kicks in,"* then
  later, discussing the HELOC variant specifically: *"I'm just gonna put next to it confusing as
  of right now."* (TJ Sims call, `3858084784`, 2026-06-11) This is not an isolated case — see
  matching confusion in `3609942284` and `4022529178`. **This is the single most repeated
  friction point across the sample.**
- **Cost-disclosure timing complaint.** A client explicitly states a cost impact (needing
  $26,000 for deferred property taxes at closing) *"wasn't disclosed until literally the week of
  closing"* and that the arrangement they were promised (using unlock proceeds to pay property
  taxes) changed without earlier warning, calling it *"a key factor in us doing this in the first
  place."* (Evan Jaquias-Johnson, `4077923012`, 2026-08-24) Full context in
  [[2026-08-29-aircall-mining]].
- **Massachusetts LO compliance friction.** On an MA-related call, the loan officer refuses to
  discuss "rate" or "loan" specifics with the client directly, citing compliance constraints
  tied to referral/dual-role rules. (Marisa Drake, `3836269678`, 2026-06-04) — corroborates the
  code-level MA carve-outs already documented in [[bbys-buy-box-and-eligibility]]
  (`DTI_DROP_ONLY_STATE_CODES`, `EXTENSION_EMAIL_REGULATORY_BLOCKED_STATES`), from an independent
  source (live call, not code).
- **Term overload generally.** A typical 60+ minute pitch call introduces 5+ named terms (equity
  unlock, equity boost, DTI drop, HELOC Equity Boost, variable pricing) in sequence, and LOs
  interrupt mid-explanation to ask for simplification at a noticeably high rate in the sample —
  consistent with the naming-collision finding above rather than a separate issue.

## 🔴 "Orchard" appears active in live calls — contradicts team.md's "no longer works Orchard"

An August 2026 LRM call (`4042268953`, Evan Jaquias-Johnson) references an **active, current**
operational relationship: *"we have a relationship with them [Orchard]... specifically for... their
concierge program... Greg Lehman is their lead project manager, and so we utilize them for our
in-house... repair work."* This is **Orchard Concierge**, a repair/vendor-coordination service
used on live BBYS deals — not the sales/lead-referral relationship. It is consistent with, and
corroborates, the still-active Orchard integration already documented in code
([[bbys-comms-journeys]] § "Orchard is a hybrid, deliberately" — live `orchard_builder_comms_enrollee?`
logic, `equityadvance@orchard.com` / `contracts@orchard.com` CC rules as of 2026-08-18; and
[[bbys-buy-box-and-eligibility]]'s `RAPID_ORCHARD_BYPASSED_CHECKS`). **This directly contradicts**
the "known context" note in this operation's brief ("The team no longer works Orchard") — see
[[2026-08-29-aircall-mining]] for the full flag. Not corrected here since `team.md` is out of this
doc's scope; flagged for the vault owner to reconcile.

## Open questions

- Is the 620 mid-FICO floor a hard system gate or an LSM rule-of-thumb? Not confirmed against
  HAPI eligibility code in this pass — cross-check against [[bbys-buy-box-and-eligibility]].
- How widespread is the cost-disclosure-timing complaint beyond the one call sampled? Worth a
  targeted pull of all calls where `ai_overall_score` is low *and* transcript contains
  "disclosed"/"surprise"/"didn't know" to size the pattern.
- Is there a standard internal reference (battlecard, wiki) LSMs are trained from? Calls suggest
  reps individually reconcile Equity Unlock/Equity Boost terminology on the fly rather than
  reading from a shared script — worth confirming whether `talk_tracks` /
  `value_propositions` / `success_stories` tables in Data Bridge (currently all 0 rows) were ever
  meant to hold this and are simply unpopulated.
- Variable pricing eligibility: reps describe it informally as a "$1M+" rule; [[bbys-pricing-engine]]
  documents a $425,000 minimum valuation gate. Are these the same rule described at different
  precision, or two different thresholds (soft LSM guidance vs. hard system gate)?
