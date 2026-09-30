---
last_updated: 2026-08-29
status: current
source: slack (homelight.slack.com) 2026-03-01 → 2026-08-29; ~35 channels + targeted search
scope (updated 2026-08-29 — §5 Agent Success MBR added from Google Drive): Company-wide picture of HomeLight beyond Homes/BBYS — business lines, leadership map, Mar–Aug 2026 narrative, and how decisions actually get communicated
source: HomeLight-Vault/context/homelight-company-overview.md
imported: 2026-09-29
---

# HomeLight — Company Overview (beyond Homes/BBYS)

> Tulli's vault is deep on Homes/BBYS and near-silent on the other ~three-quarters of the
> company. This doc is the outside-the-Homes-division picture, built from six months of Slack.
> Everything here is **stated** (someone said it in Slack) unless marked `(inferred)` or
> `(observed)`. Related: [[homelight-homes]] · [[team]] · [[bbys-overview]] · [[partners]]

> 🔑 **HomeLight is four businesses, not one.** (1) A consumer→agent **referral marketplace**
> monetized by success fee; (2) an **investor/iBuyer lead network** (Simple Sale) monetized by
> pay-per-lead and rev share; (3) **agent SaaS** (HLM, HomeLight Offers, soon HomeLight Pro);
> (4) **balance-sheet lending + title/escrow** (Homes/BBYS, HLHL, HLCS). The Homes division Tulli
> runs is only #4's lending half.

---

## 1. Business lines — what's alive, what's sunset

| Line | Product / surface | Monetization | Status as of 2026-08-29 | Evidence |
|---|---|---|---|---|
| **Agent referral marketplace** | `sales-app.homelight.com`, agent portal, HL mobile App, quiz funnels | Referral success fee on closed sides | **Core, alive.** Handmatch team + automated waterfall both live | #cas-handmatching, #srm-hl-support, #centralized-epd-support passim |
| **Elite Agent tier** | Badging, "platform utilization" tracking, SRM ownership, Elite Agent Spotlight social | Retention/upsell vehicle for the marketplace | Alive; SRM-owned, Periscope "Marketing — Elite Agents by SRM" dashboard | Gian Duque #marketing-analytics 2026-07-07; Sarah Jayne Olan #homelight-booster-club 2026-07-13 |
| **Investor network / Simple Sale** | `simplesale.com`, investor portal, buy boxes, PPL bidding | **Migrated rev share → PPL (pay-per-lead)**; PPL takes priority over rev share | Alive but churning on lead quality | Jon Peacock #proj-cerberus-sales 2026-07-17 |
| **HLM (HomeLight Listing Management)** | Was `disclosures.io`; cut over to `hlm.homelight.com` **2026-08-19** | Subscription (Net MRR) | Alive + actively invested; PLG launched 2026-08-05 | Ätali Silva #project-dio20 2026-08-19; Angelina Micha Djaja #project-dio20 2026-08-05 |
| **HomeLight Offers** | `offers.homelight.com` | Agent subscription (~$99/mo per HL Pro spec) + exclusive referrals/appointments | Alive; added as a collections line item in the D2C MBR June 2026 | Justin Tran #d2c-biweekly-updates 2026-06-04; Sumant #growth-sem 2026-07-01 |
| **HomeLight Pro** | Bundle above Offers | ~$1,000/mo per agent (explicitly "subject to change") | **Pre-launch.** Target: end of October 2026 | Justin Tran #homelight-pro 2026-08-14 |
| **Listing Boost** | `listing-marketers.replit.app` — agent marketing-asset generator | Stripe subs planned | **Prototype, not shipped.** Built by Sumant on Replit; handed to angelina 2026-08-26 | #proj-listing-boost 2026-04-01 → 2026-08-26 |
| **HomeLight Convert** | Referral/lead cleanup + conversion feedback | TBD | **Concept only**, resurfacing inside HL Pro scope | Justin Tran #homelight-pro 2026-08-27 |
| **HLCS (title & escrow)** | Qualia-based closings; branches CA, AZ, TX, FL, IL, WA | Title/escrow fees | Alive; own all-hands + MBR cadence | #hlcs_allhands, #hlcs_*_wires channel family |
| **EVA** | AI escrow agent (repos `eva-ai`, `eva-api`, `eva-ui`) | Cost-out inside HLCS; publicly positioned as a platform | **Launched 2026-04-27** — the year's flagship announcement | Alexandra Lee #homelight-news 2026-04-27 |
| **HomeLight Home Loans (HLHL)** | Originates + **services** the HELOC leg | Interest/fees | Alive | Marc Kaplan #tls-irx-strogus 2026-08-24: "we are serving it (HomeLight Home Loans)" |
| **"The Card"** | HELOC-backed card product; ACH drawdown via Pesto; own hotline | Consumer credit | **In build.** Two logos in trademark as of 2026-07-17 | Karen De Leon #design-requests 2026-07-17; #homelight-pesto-ach-drawdown 2026-07-27 |
| **Homes / BBYS** | Bridge + Equity Boost + HELOC Equity Boost + DTI Drop | Program fee (2.25–2.4%, variable pricing) | Core — see [[homelight-homes]] | — |
| **Cash Close / Trade-In / Cash Offer 2.0** | `#d2c_co`, `#d2c_ti`, `#simple-sale_cash-close_support` | — | **Sunset.** No 2026 traffic in any of these channels | (observed) channel silence Mar–Aug 2026 |

> ⚠️ **"Cash Offer" in 2026 Slack almost never means HomeLight's legacy Cash Offer product.** It
> means (a) a competing all-cash bid on a client's departing residence, (b) the Simple Sale
> investor cash offer, or (c) a third-party platform ("Cash Offer Pro"). Do not read those
> mentions as the old product being alive.

### 1a. The referral marketplace mechanics worth knowing

- **Two match paths:** an automated **waterfall** (buy-box / zip / preference matching) and a
  human **handmatch team** (Danny's org; e.g. Nicole Benjamin). Handmatch overreach is a real
  failure mode — 6 agent intros on one lead flagged as excessive (Felipe Martinez
  #proj-cerberus-sales 2026-07-16).
- **Warm Transfer (WT)** and a **voice-AI** front end run on inbound D2C leads (Twilio
  recordings). As of 2026-08-10 the bot was opening agent-first on cash-offer leads and
  suppressing the cash-offer path entirely — Felipe's V2 critique in #d2c-voice-ai-live-leads.
- **Rev-share collection is leaky.** #revshare-investor-collection-audit-2026 (opened Aug 2026,
  Lauren Garnel + Jon Peacock + Justin Tran + Espen) is auditing ~**280k rev-share referrals
  2023→PPL migration** for closings HomeLight was never paid on; ~10% need manual review.
  Endurance alone has ~115k referrals and is "the investor that generates most of our rev share
  revenue" (Lauren Garnel, 2026-08-18).
- **U.S. News partnership** is a live agent-facing rev-share product (`#usnews-srm-sales`);
  HL Pro proposes bundling "$2,500 off US News subscription."

### 1b. HomeLight Pro — the proposed bundle (2026-08-14, Justin Tran)

HL Homes placements · $2,500 off US News · custom CRM integration · warm-transfer priority ·
Pro badging on all public agent surfaces · $99/mo Offers (existing Offers agents "graduate") ·
souped-up agent profile/portal/emails · HLM included · **agent-branded HELOC Card** · reduced
rev share or credit · title & escrow credit attached to client · HomeLight Convert · priority
support · extended claim window (reserve a referral).

> 🔴 HL Pro is the first product that **bundles all four HomeLight businesses into one agent
> subscription** — marketplace + SaaS + lending (the Card) + title. If it ships, it changes the
> RevOps object model materially (subscription state has to gate referral routing, HLM access,
> and a lending product). Payment rails are unsettled: Sai is evaluating checkout.com against
> Stripe depending on negotiation outcome (#homelight-pro 2026-08-21).

---

## 2. The Mar–Aug 2026 company narrative

| Date | Event | Source |
|---|---|---|
| 2026-03-02 | **Ankur Jain** named a 2026 HousingWire Finance Leader | Alexandra Lee, #homelight-news |
| ~2026-03 | **Rippling** replaces prior HRIS (Deel for LatAm contractors). Announced at a company all-hands; LatAm staff unenthused | Basem Bader #hl-mexico 2026-03-11; Pedro Oliveira #hl-brazil 2026-05-28 |
| 2026-03-12 | Annie Dreshfield named **Inman Marketing All-Star, 4th consecutive year** | #homelight-news |
| 2026-03-19/20 | **FinCEN reporting rule enjoined** — HLCS halts fee collection and reporting mid-flight | Debbie Schoenborn, #hlcs_allhands |
| 2026-03-24 | Wei Wang introduces himself workspace-wide as **"the new Head of Growth… I manage the entire sales team at Homes"** | #homelight-x-aircall |
| 2026-03-25 | **Q2 2026 Lender Insights Report** published (`active.homelight.com`) | #homelight-news |
| 2026-04-08 | **Global Gmail outage** — incoming mail dead ~7am–5pm; HLCS ran closings by phone and Qualia Connect | #hlcs_allhands, #general |
| 2026-04-14 | Nick Friedman named a **Next Gen Leader** (Progress in Lending) | #homelight-news |
| **2026-04-27** | **EVA launch + $40M BlackRock financing** — "industry's first AI-powered escrow agent." BusinessWire, Inman, Phoenix Business Journal, TMCnet, WRE News | #homelight-news, #eva-epd |
| 2026-05-13 | AZ (Scottsdale) office moves — first all-hands in the new space | Javy Morales, #az_office |
| 2026-05-15 | **Q2 Top Agent Insights** (950 agents): 82% report price cuts; 61% say monthly payment is the #1 buyer factor | #homelight-news |
| 2026-05-19 | Simple Sale Facelift A/B launches (80/20), immediately paused for broken investor intros | #proj-simple-sale-facelift |
| 2026-05-21 | **HomeLight AI Explorers** launches — company-wide AI enablement program, Bruno + Culture Program Team | Doug Santos, #homelight-ai-explorers |
| 2026-06 | **RIF / departures wave.** Wei Wang, Mette Adams, Matt Leddy out 6/12; Suzanne Krause + Veronica Aguilar out ~6/15–16 | [[team]] § Recent Departures; corroborated #the-boyz 2026-06-17, Andrew Soss 2026-06-25 ("now that Wei isn't here") |
| 2026-06-07 → 06-30 | **2026 Annual Performance Review cycle** runs in Scrappy360 (see §4) | Mary Remillard, #peoplemanagers |
| 2026-07-01 | Comp/title changes from the review cycle take effect — **including Homes** | Mary Remillard, #peoplemanagers 2026-06-22 |
| 2026-07-23 | **Q3 2026 Lender Insights Report** — 57 lending companies surveyed. Headline: 46% of LOs say starter-home shortage is the top first-time-buyer barrier vs. 14% citing rates | #homelight-news |
| 2026-07-26 | HomeLight at **Inman Connect** | Sumant, #homelight-booster-club |
| 2026-07-31 | **HELOC pulled from FL, MI, MT** pending servicing licenses | Marc Kaplan, #lab-rats-lp-sales |
| 2026-07-31 | Homes closes July at **295 IRUCs — best month ever** (best week ever: 74) | Jake Vogel, #pursuit-of-950 |
| 2026-08-04 | **Board meeting** | Andrew Soss, #the-boyz |
| 2026-08-05 | **HLM PLG launch** (Bay Area + Sacramento): agents create listing packages from the agent portal | Angelina Micha Djaja, #project-dio20 |
| 2026-08-05 | Nick Friedman on the **Chrisman Commentary** mortgage podcast — Q3 Lender Insights | #homelight-news |
| 2026-08-14 | **HomeLight Pro** kicked off, targeting end-October launch | Justin Tran, #homelight-pro |
| 2026-08-19 | **`disclosures.io` → `hlm.homelight.com`** domain cutover | Ätali Silva, #project-dio20 |
| 2026-08-19 | Q3 Lender Insights picked up by Inside Mortgage Finance, National Mortgage News, Crowdfund Insider | #homelight-news |
| ~2026-08/09 | **Company retreat** — Tulli skipping (baby due) | #the-boyz 2026-06-23; Sarah Jaka 2026-08-18 |

> ⚠️ **The $40M is venture debt, not equity.** Public framing is "$40 million in financing from
> BlackRock." Internally Mike Abner described it flatly as "a venture debt refinance deal with
> blackrock" (#eva-epd, 2026-04-27 14:49). Do not model it as a priced equity round or as new
> growth capital. The vault's "IPO in ~18 months" note (stated by Drew, 2026-03-26) is unchanged
> by it, and no fundraise, acquisition, or layoff announcement appeared in any company channel
> Mar–Aug 2026.

### Strategy shifts visible in the data

1. **Rev share → PPL** across the investor network. Investors bid for leads instead of paying on
   close. Consequence: rev-share receivables from 2023–2025 are still being chased (§1a), and
   quality complaints migrated from "these leads don't close" to "these leads get spammed."
2. **Expected-revenue haircuts.** 2026-07-28, Sumant told channel owners two haircuts went into
   effect: GVG capital early-stage connected leads cut ~30%, investor-only rev-share early-stage
   leads cut ~35% — net 10–15% impact on GVG and ≤3% overall (#growth). **Any D2C revenue series
   spanning 2026-07-27 has a methodology break at that date.**
3. **SaaS as a second P&L.** A dedicated Periscope model, "SAAS (HLM & HL Offers) — Data Source
   for MBR," now tracks Net MRR, activation, reactivation and churn per agent
   (Michaela Miranda, #consumer-analytics 2026-08-19).
4. **AI as the org's stated identity.** EVA is the public proof point; internally "Rise of the AI
   Builder," AI Explorers, Scrappy360, and 350+ non-engineers shipping code are the internal
   proof points. Codex and Claude are both in daily production use across EPD.

---

## 3. Leadership map (company-wide)

> Cross-checked against [[team]] and [[homelight-homes]]. Titles are as stated in Slack; where
> no one has said the title out loud it is marked `(inferred)`.

| Scope | Person | Role as evidenced | Notes |
|---|---|---|---|
| Company | **Drew Uher** | CEO | Referenced as final approver on pricing rollouts (Alex Kwan, #proj-dynamic-pricing-bbys 2026-08-28: "pending approval from drew"). Quoted in the Inman EVA piece. Rarely posts in Slack — a `from:drew` search over six months returns nothing. Working email `au@homelight.com`. |
| Company | **Sumant Sridharan** | Senior executive over **Agent/D2C + BI + growth product** `(inferred: COO or President)` | The single most operationally present exec in Slack: runs #growth, #proj-listing-boost, #proj-cerberus-sales, #proj-d2c-heloc, #homelight-pro, #project-dio20, #growth-sem/-seo/-affiliates, #usnews-srm-sales. Sets revenue-recognition policy. Prototypes products himself in Replit. |
| Company | **Mike Abner** | CTO / EPD | Also the org's Slack-connector/tooling gatekeeper. Created #general in 2014. |
| Company | **Ankur Jain** | Finance leadership `(inferred: CFO)` | 2026 HousingWire Finance Leader. ⚠️ Distinct from **Ankur Bansal**, former head of HLCS, now at DOGE/DOT (Nick Santulli, #the-boyz 2026-06-08). |
| Homes | **Nick Friedman** | President/GM of Homes `(inferred; called "El Presidente" by Tejas, 2026-08-20)` | Signs all partner agreements (Barrett, Neighborhood Loans). Approves pricing/LTV exceptions (90% LTV, 2.35% flat fee). External spokesperson for the Lender Insights reports. |
| Homes | **Jake Vogel** | Head of Lender Relations | Runs #pursuit-of-950 and #lab-rats-lp-sales; day-to-day sales leadership after Wei's exit. |
| Homes | **Marc Kaplan** | Head of Business Operations; owns **The Card** incl. capital markets + bank compliance | #design-requests 2026-07-17; DM 2026-08-18. |
| D2C/Growth | **John Van Slyke (JVS)** | Runs the **D2C MBR** (monthly business review) and paid/organic growth | #d2c-biweekly-updates; joined #peoplemanagers 2026-06-12. |
| D2C/Growth | **Sai** | Engineering leader for D2C / agent platform | Owns quiz infra, payments provider choice, HLM. |
| D2C/Growth | **Justin Tran** | Product — collections, HomeLight Pro, Offers | Started #homelight-pro. |
| D2C/Growth | **Lauren Garnel** | Investor network / agent field ops | Drives the rev-share collection audit. Left #peoplemanagers 2026-06-18. |
| D2C/Growth | **Jon Peacock** | Investor sales / account management | Owns investor churn conversations. |
| D2C/Growth | **Felipe Martinez** | D2C analytics + alerting | Owns #d2c-911-alerts, voice-AI QA. |
| D2C/Growth | **Alma Chen** | Product/design — Simple Sale facelift, Listing Boost | Joined ~Apr 2026. |
| D2C/Growth | **Angelina Micha Djaja** | Product — HLM / Project DIO 2.0 | Led the 2026-08-05 PLG launch. |
| D2C/Growth | **Michaela Miranda ("Mic")** | BI / analytics lead post-Alex Chun | Builds the SaaS + waterfall exec dashboards; also handles international contractor comp. |
| D2C/Growth | **Danny** | Handmatch / agent-matching operations | |
| HLCS | **Mark Kyser** | HLCS leadership — hosts the HLCS all-hands | Org chart lives in a Lucidchart linked from #hlcs-highfives topic. |
| HLCS | **Debbie Schoenborn** | Compliance/ops lead (FinCEN, wire fraud, Qualia) | The operational voice of record in #hlcs_allhands. |
| HLCS | **David Siegler** | Branch/ops leadership, San Diego + comms | |
| EVA | **Kelly Adu'Elohiym** | EVA PM — owns MBR / HLCS All Hands / Automation Weekly readouts | |
| EVA | **Jon Moubayed, Eric Kao, Irene Garcia Montoya, Pavel Savva** | EVA engineering | Repos `eva-ai`, `eva-api`, `eva-ui`. Landing AI for extraction, Qualia integration. |
| People | **Mary Remillard** | HR — owns the performance-review cycle | |
| People/Culture | **Doug Santos, Julia Gentry, Dafne Mata, Javy Morales, Todd Rhodes** | Culture Program Team (SF + AZ + remote) | |
| Comms/PR | **Alexandra Lee** (first press release Apr 2026), **Sarah Jayne Olan** | PR + employee advocacy | |
| Marketing | **Taryn Tacher** (agent PMM), **Richard Haddad** (content), **Karen De Leon / Soleil Ocampo / Ren De Leon** (design, PH) | | Design team reports to Guilherme Batista post-RIF. |
| Finance/Payments | **Tara Edwards**, **Sarah Jaka**, **Jon Geraci**, **Jhen Saplada**, **Leticia Cabrera** | Payments, partner agreements, MBR financials | |

> 🔴 **Correction to [[homelight-homes]] (dated 2026-04-24).** That doc lists Wei Wang as
> "Former Head of Strategic Growth." His own workspace-wide introduction on 2026-03-24 said
> **"Head of Growth… I manage the entire sales team at Homes."** Trust the self-introduction for
> the title. Departure date (2026-06-12) is unchanged and independently corroborated: Andrew Soss
> wrote "now that Wei isn't here" on 2026-06-25, and Data Bridge PR #608 revoked
> `wei.wang@homelight.com` (admin) and `mette.adams@homelight.com` (manager) on 2026-07-09.

> 🔴 **Refinement to [[team]] on Annie Dreshfield.** [[team]] records a "~late March" departure.
> Slack shows she was still active on 2026-03-12 and that by **2026-06-17** she was back as a
> **contractor** ("Make that new contractor annie do it," #the-boyz). As of 2026-07-07 there was
> still no confirmed creative-asset turnover (Carlo Mangoba, #homelight-agency: "it might also be
> in her turnover files if she left any"). Treat late March as the FTE end date and mid-June+ as
> a contractor re-engagement.

---

## 4. Culture and process — how decisions actually get communicated

**The formal channels are dead.** `#company-announcements` (created 2022) contains exactly one
message: its creator joining. `#general` is a culture/social channel — trivia, brackets, office
lunches — and its newest substantive post in the sweep window is 2026-04-24. `#okrs` has no 2026
traffic. **Do not look for company decisions in the channels named after company decisions.**

Where things actually land, in descending order of signal:

| Mechanism | Cadence | Who runs it | What lands there |
|---|---|---|---|
| **All-hands** (Zoom + in-person in SF and AZ) | Monthly-ish, 11am PT | Sandy Liao-Martin schedules; Drew presents | Company strategy, Rippling migration, EVA. Announced via `#general`, `#sf_office`, `#az_office` — never a dedicated channel |
| **MBR (Monthly Business Review)** | Monthly, per division | JVS (D2C), Kelly Adu'Elohiym (HLCS/EVA) | Financials, win rate, run rate vs. plan, named project owners. Deck-driven in Google Slides; the Slack thread is the agenda |
| **HLCS All Hands** | Monthly | Mark Kyser / Debbie Schoenborn | Compliance changes (FinCEN), Qualia process, branch news |
| **`#homes-product-announcements`** | Per release | Bruno Gonzalez, Faaz Shaikh, Dave Spivey, Joel Shurtleff | The best-run announcement channel in the company: what shipped, why it matters, a Loom demo, and an explicit "what didn't change" |
| **`#homelight-news` / `#homelight-booster-club`** | Per hit | Alexandra Lee, Sarah Jayne Olan | Press coverage + a request to like/share. This is where you learn what HomeLight is telling the market |
| **Deal channels** (`#<partner>-<stage>-<name>-<address>-<state>-<speed>`) | Per deal, auto-created | HomeLight Sales App bot | Where exceptions are actually granted — "Friedman approved 90%," "per :friedman:" |
| **Small private groups** (`#the-boyz`) | Continuous | — | Where org reality is discussed candidly and earliest. Not a source of record, but the leading indicator |

**Other process facts worth knowing:**

- **Scrappy360** (`scrappy360.homelight.com`, own repo) is HomeLight's in-house performance and
  peer-recognition app — self / peer / upward reviews, then manager "downward review," then a 1:1,
  then a shared written summary that HR uses to mark the cycle complete. 2026 General timeline:
  reviews due **June 7**, downward reviews + 1:1s due **June 30**, comp/title changes communicated
  to managers **June 22**, effective **July 1** (Homes included). Peer-review assignments arrive
  from `scrappy360@homelight.com`.
- **Thought leadership is a real GTM motion, not vanity.** Two quarterly primary-research reports
  — **Lender Insights** (surveys LOs; Q2 published 3/25, Q3 published 7/23 off 57 lending
  companies) and **Top Agent Insights** (surveys agents; Q2 off 950 agents, published 5/15). They
  reliably convert into Inman / Inside Mortgage Finance / National Mortgage News / HousingWire
  coverage, and Nick Friedman is the named spokesperson.
- **Geographic footprint is global and material.** Offices: SF and Scottsdale AZ (AZ moved to new
  space ~2026-05-13). Distributed teams with their own channels: **Brazil** (`#hl-brazil`),
  **Mexico** (`#hl-mexico`), **Uruguay** (Bruno), **Philippines** (BI + design), **Saint Lucia**
  (LRM assistants, live 2026-06-19 per [[team]]; a "St Lucia strategy and timelines" slide was on
  the D2C MBR as early as 2026-03-28). LatAm staff are paid via Rippling and openly prefer Deel.
- **AI enablement is programmatic.** `#homelight-ai-explorers` (launched 2026-05-21, Bruno +
  Culture Program Team, "no experience required"), an "AI Builders biweekly" spun out of an
  all-hands, and non-engineers shipping production code. Model allegiance is contested and openly
  argued about (#ai, #the-boyz).

---

---

## 5. Agent Success MBR — the marketplace/title side, quantified (added 2026-08-29, from Drive)

**Sources:** `Agent Success MBR` decks — 2025-05-01 (owner sumant@), 2025-09-01, 2025-10-01,
2025-11-01 (owner mic.miranda@). Drive modified dates 2025-06-04 → 2025-11-21.

⚠️ **These are 2025 figures.** No 2026 Agent Success MBR is in Drive under this account. Quote
with the as-of date attached or not at all.

This is the **SRM/Agent-Success** business — the referral-marketplace and HLCS-title side that
§1 describes qualitatively. It reports on escrow opens, title production, and BBYS
applications *sourced by SRMs* (a different population from the LSM channel this vault
otherwise covers).

### Escrow opens vs forecast, monthly totals

| Month (as-of) | Escrow opens | Forecast | % achieved |
|---|---|---|---|
| Apr 2025 | 369 | 396 | 93% |
| Jun 2025 | 362 | — | — |
| Jul 2025 | 387 | — | — |
| Aug 2025 | 384 | 420 | 91% |
| Sep 2025 | 323 | 354 | 91% |
| Oct 2025 | **278** | 344 | **81%** |

🔴 **The Agent Success channel was contracting through H2 2025.** Oct 2025 challenges named in
the deck: **−68% gross opens YoY (IL)**, **−64% YoY (Bay Area)**; Sep 2025: −57% YoY (AZ),
−51% YoY (IL), BBYS applications down ~40% MoM. The Aug-2025 deck's "all-time high" framing
(502 total escrows opened, 72% BBYS conversion) is two months earlier and does not survive.

### HLCS title production (monthly totals)

| Month | Title opens | Title closes |
|---|---|---|
| Jun 2025 | 136 | 95 |
| Jul 2025 | 157 | 103 |
| Aug 2025 | 211–212 | 83 |
| Sep 2025 | **249** (all-time record) | 112 |
| Oct 2025 | 225 | **146** |

Concentration: **Facio Palomino** alone runs 77–120 title opens/month — roughly 40–50% of
company title volume. "Diamond Bar" appears as a second named source from Aug 2025 (0 → 48/mo
in three months). → [[hlcs-title-escrow]] · [[top-producers]].

### Metric definitions stated in the decks

- **"% Elites Consistent"** = % of Elite agents who gave ≥1 escrow **each month for the last 3
  months**. **"% TAM Consistent"** = same test for TAM agents. These are *consistency* rates,
  not participation rates — Oct 2025 values run 12–27% for Elites and **0% for TAM across every
  zone**. → [[lo-elite-program]] · [[homelight-agent-marketplace]] · [[glossary]] (TAM collision).
- **"Zone slots"** — each SRM manages N zones × ~3 slots; "% Slots Used - Unique" runs 19–61%.
- ⚠️ Repeated deck footnote: **"BBYS Applications doesn't include lender activated
  applications."** Any BBYS number from an Agent Success MBR is the **agent-sourced subset
  only** — it is not comparable to the LSM-channel application counts in
  [[bbys-unit-economics]] §11. Add to [[numbers-that-disagree]].
- ⚠️ Vocabulary drift **within the same deck series**: the May-2025 deck says "BBYS
  Submissions / BBYS Leads"; by Sep-2025 the same rows are labelled "BBYS Subs / BBYS Apps."
  Same rows, renamed mid-series. → [[stage-vocabularies-master]].
- **IRCC** appears as a tracked line from the Sep-2025 deck onward (BBYS IRCC by SRM) — a term
  not otherwise defined in the vault.

### Historical: the Jan-2023 `200BBYS_MBR`

`200BBYS_MBR_Jan_22` (owner vanessa@, Drive modified 2023-01-27) — the "Path to 200 BBYS"
plan. 2023 North Star: **200 BBYS IR Contracts in June 2023**. Jan-2023 actuals vs plan: 165
leads (63% of plan), 127 pitches (77%), 57 conditional approvals (98%), **6 IR contracts**
(65%), 8 DPL funded, 1 DR sale. OKR owners named: Vanessa Famulener (overall), Devu/Viktoriya
(leads), **Nick** (conditional approvals + rejection rate), Taylor (agent retention). Included
a **$60M capital raise** OKR. Useful only as a scale marker: **6 IR contracts/month in Jan
2023 vs a 2026 plan of 500–950/month** — a ~100x trajectory.


## Open questions

1. **Who replaced Wei as Head of Growth for Homes?** No announcement exists in any channel. Jake
   Vogel appears to have absorbed day-to-day sales leadership by default, reporting to Friedman.
   Confirm whether the role was backfilled or eliminated.
2. **Is Sumant COO, President, or Chief Product Officer?** He behaves like all three. No Slack
   message states his title in the window.
3. **Ankur Jain — CFO or another finance title?** Only evidence is the HousingWire "Finance
   Leader" award and Tulli's parenthetical "(not the CFO…)" distinguishing him from Ankur Bansal.
4. **What are the actual revenue splits across the four business lines?** **PARTIALLY ANSWERED
   2026-08-29 from Drive** — the Homes/BBYS line now has an absolute plan number: **$55.49M
   planned 2026 program revenue**, against **$16.77M of 2025 LSM-attributed actual revenue**
   ([[bbys-unit-economics]] §11, from `Wei's 550 model - Homes 2026 Planning`, 2026-01-16).
   ⚠️ **The other three lines remain unquantified.** Drive holds no 2026 D2C MBR, Homes MBR, or
   board deck under this account — only **Agent Success MBR** decks (May/Sep/Oct/Nov 2025, owners
   mic.miranda@ / sumant@) and a Jan-2023 `200BBYS_MBR`. See §"Agent Success MBR" below and
   [[projects/2026-08-29-drive-mining]] for what was searched and not found.
5. **Does HLCS run title joint ventures?** Summit Title Group (College Station TX) celebrated a
   one-year anniversary with HLCS leadership attending (2026-04-03), and Irene GM references "JV
   automation." Structure unconfirmed.
6. **HomeLight Pro pricing sanity.** $1,000/mo/agent is ~10x the Offers price point and far above
   typical agent SaaS. Confirm whether that survived the August discussions.
7. **How large is HLCS and EVA's actual footprint?** Six branch states are visible from channel
   names (CA/AZ/TX/FL/IL/WA); order volume and headcount unknown.
8. **What was decided at the 2026-08-04 board meeting?** Only the fact of it is in Slack.
