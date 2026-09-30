---
last_updated: 2026-08-29 (correction: Builder & Growth pod channel renamed. Previous: 2026-04-24)
type: context
source: HomeLight-Vault/context/homelight-homes.md
imported: 2026-09-29
---

# HomeLight Homes — Division Overview

> High-level view of the HomeLight Homes division (lending products, LO-channel distribution, BBYS-centered). Read this for "who, what, why" before diving into any project note.

## Mission

Let homeowners move before they sell. The Homes division builds and operates HomeLight's owned-balance-sheet lending products — BBYS (Buy Before You Sell), HELOC Equity Boost, Asset Equity Boost — and runs the Loan Officer partner channel that distributes them.

## Company Context (as of 2026-04-24)

- **IPO timeline:** ~18 months (Drew Uher, 2026-03-26)
- **AI transformation company-wide:** 350+ non-engineers can ship code, 70% of commits AI-generated in Feb 2026. Drives the "Rise of the AI Builder" internal vision.
- **Recent org moves:** Annie Dreshfield departure (Marketing/Brand, late March) and Alex Chun (BI) caused temporary consolidation under Nick Friedman and Sumant respectively. Quiet layoffs in SF + Phoenix (Apr 10) spared revenue-generating roles.

## Leadership (relevant to Homes)

| Role | Person | Notes |
|------|--------|-------|
| CEO | Drew Uher | "Rise of the AI Builder" — sets product strategy, pushes deadlines, drives UWM daily updates |
| GM / VP | Nick Friedman | Temporary umbrella after Annie's departure — partner contracts, comp plans, legal |
| Senior Executive | Sumant Sridharan | Oversees BI (post-Alex Chun) — data infrastructure, AI efficiency |
| Former Head of Strategic Growth | Wei Wang | No longer with HomeLight as of Friday 2026-06-12; Nick Santulli now reports directly to Nick Friedman |
| Head of Lender Relations | Jake Vogel | Pipeline, LSM enablement, UWM forum attendee |
| Head of Business Operations | Marc Kaplan | HELOC Equity Boost, LO portal, co-pilot roadmap — weekly sync with Nick + Gui |
| Head of Revenue Operations | Nick Santulli ("Tulli") | BBYS + LO channel RevOps — vault owner |
| Head of Builder Relations | Nick Plamondon | Builder channel, NHC partnerships (DR Horton, Pulte, NVR, Lennar) — new-hire ~Apr 13 |
| Product (HELOC pod) | Ashwin + Marc Kaplan | Elite Lender Program, HELOC calculator, app flow. *Anirudh Bhutani held this until his departure 2026-08-07; Elite Lender Program product ownership is currently unassigned — see [[team]].* |
| Builder/Engineering pod lead | Joel Shurtleff | Builder opportunity workflows, econ model, data tape |
| Data Infrastructure | Alex Kwan | HubSpot-Redshift sync, channel org |
| Enterprise AEs | John Labrada, Kim Tanner | UWM sales, QBR presentations (NEXA, RWM, Envoy, etc.) |

See [[team]] for the full LSM/LRM roster, pod composition, and individual notes.

## Pod Structure (formalized 2026-03-24)

Three pods + a Builder sub-team, each with a public OKR channel in Slack:

| Pod | Slack channel | OKR (through EOY 2026) |
|-----|---------------|--------------------------|
| HELOC | `#homes-heloc-pod` | 50 HELOCs by May; 70% above target equity by EOY |
| Builder & Growth | `#homes-client-channel-and-growth-pod` (renamed from `#homes-builder-and-growth-pod` on 2026-08-19, per Slack sweep — scope broadened from "builder channel" to client-channel generally, i.e. banks and named partners) | 100 contracts from builder channel by May |
| Ops & Efficiency | `#homes-ops-and-efficiency-pod` | "One-click close" on all deals |
| Elite Lender Program | `#elite-lender-program` (private) | Top-tier LO retention + expansion |

Pod membership cross-cuts teams — an LSM can be embedded in HELOC pod while still carrying her normal book.

## Products

### BBYS (Buy Before You Sell) — flagship
Owners buy their next home before selling the current one, with HomeLight bridging the gap. HomeLight holds balance-sheet risk on the departure residence. See [[bbys-overview]] and [[bbys-edge-cases]].

### HELOC Equity Boost
Tap departure-residence equity as a HELOC to fund the incoming-residence purchase. Launched/expanding 2026-Q1. Formerly Anirudh's pod; now Ashwin (product) + Marc Kaplan after Anirudh's departure 2026-08-07. See [[heloc-product]] and [[dti-drop]].

### Asset Equity Boost
A companion solution paired with HELOC for clients above the equity threshold. Ashwin's directive (Apr 20): "never lose a deal without exploring HELOC and Asset Equity Boost."

### Cash Close (legacy)
Previous BBYS variant now mostly retired. Referenced in older workflows; do not build new logic against it.

## Distribution Channels

1. **Loan Officers (LOs)** — primary. ~35 LSMs/LRMs/AEs cover several thousand LOs organized under mortgage companies ("partnerships").
2. **Builder channel** — Nick Plamondon, Tiffany Traxler. New-home-construction partnerships; separate funnel.
3. **Enterprise partnerships** — UWM, TLS, Orchard, Fairway, Cross Country Mortgage, Lennar, APM, NEXA, Envoy, Barrett, Mortgage Pass, NFM Lending, GoMortgage, etc.
4. **Direct marketing** — Andrew Soss ("Sauce") runs HubSpot email sequences, UWM campaigns, DTI Drop awareness.

See [[lo-lifecycle]] for the full LO-stage funnel and qualification rules.

Top 10 partnerships by deal volume (2026-04-24):
`tls` (7,939) → `orchard` (2,976) → `fairway` (890) → `cross-country` (710) → `lennar` (561) → `apm` (341) → `envoy` (258) → `mortgage-pass` (191) → `nfm-lending` (150) → `go-mortgage` (87). Note: UWM attribution runs through a separate `wholesale_lender` / `intended_wholesale` codepath, not the primary partnership object. See [[partners]].

## Active strategic threads (2026-04-24)

- **UWM expansion** — Detroit training landed (Tejas, 80–100 attendees). May 14 "UWM Live" event: Marriott bar sponsorship, branded "DTI Drop" drink, billboard/geofencing, weekly webinars (~70 attendees, 700 AE emails). Weekly UWM squad sync Mondays.
- **Sales Blitz (Apr 22–23)** — Wei's first-ever cross-team gamified push ("The Business" vs "Standing on Business") to pull April IRUCs toward 330 target.
- **HELOC utilization directive** — Ashwin Dayal (Apr 20): HELOC + Asset Equity Boost must be explored on every deal before loss. Drives LSM/LRM training focus.
- **HLH Comp Reporting App** (shipped Apr 16) — canonical variable-comp tracker for LRM/LSM/Builder/RevOps. See [[2026-03-28-comp-app]].
- **Elite Lender Program** — top-tier LO retention framework, webinar series, ICP scoring project. ⚠️ Product owner unassigned since Anirudh's departure 2026-08-07 (Javy holds external events / partner conferences). ([[2026-04-16-lo-call-simulator]] feeds coaching loop).
- **Lead scoring (`proj-lender-lead-icp` — Feb 2026, Nick-owned)** — Gui's primary project; ICP classification for incoming LO leads.
- **Data Bridge** (Nick's main engineering project) — the integration platform connecting everything. See [[data-bridge]].

## KPI snapshot (as of 2026-04-24)

| Metric | April target | April pace | Notes |
|--------|--------------|------------|-------|
| IRUCs | 330 | 59 (pacing 195) | Wei pushing hard; blitz was designed to close the gap |
| Apps | 1,450 | 317 (pacing 951) | App cutoff ~Apr 22 to convert before month-end IRUC |
| Contracts | 200 | 69 (pacing 200) | |
| Weekly meetings | 85+ | Kara & Brian leading | First time above 85 since Jan |
| LSM weekly calls | 1,450 | Post-quota (started 2026-01-21) | Up ~80% from pre-quota baseline of 717 |

March closed at **216 IRUCs (+21% YoY)**, first time breaking 200.

## Meeting cadence

- **LSM & LRM Team Meeting** — weekly/bi-weekly (full Homes team)
- **Sales leadership sync** — weekly Mon (Wei, Jake, Nick)
- **UWM squad sync** — weekly
- **Sales performance dashboard review** — weekly
- **Bi-weekly Homes Exec Meeting: Sales & Marketing**
- **Tulli & Gui ↔ Marc Weekly** — BBYS / HELOC / copilot
- **Andrew ↔ Nick Weekly** — marketing campaigns
- **Sarah ↔ Nick** — payments operations
- **Mette ↔ Nick** — lead scoring / methodology deep dives
- **Lead Scoring Sync** — active project cadence
- **LS/RM Performance Evaluation** — monthly (Wei, Nick, Jake; 1st Thursday 1–1:30pm)

## Related context

- [[team]] — full roster with pod assignments + stakeholder notes
- [[bbys-overview]] — BBYS product details and terminology
- [[hubspot]] — HubSpot config, pipelines, owner IDs
- [[partners]] — partnership records, slugs, waterfalls
- [[tools-and-stack]] — tech stack inventory
- [[systems-map]] — how the tools fit together
- [[projects-active]] — current project status snapshot
- [[q2-priorities]] — Q2 2026 strategic priorities

---

## Company context update — 2026-08-29 (Slack sweep, 2026-03-01 → 2026-08-29)

> The "Company Context (as of 2026-04-24)" section above is now four months stale. Corrections and
> additions below. Full company-wide picture: [[homelight-company-overview]].

**Corrections to the 2026-04-24 block:**

- 🔴 **"Annie Dreshfield departure (Marketing/Brand, late March)"** — she was back as a
  **contractor** by 2026-06-17 (#the-boyz). Marketing creative production had no confirmed handoff
  as of 2026-07-07 (#homelight-agency).
- 🔴 **Wei Wang's title** was **Head of Growth**, per his own workspace-wide introduction
  (#homelight-x-aircall, 2026-03-24): "I am the new Head of Growth, and I manage the entire sales
  team at Homes." Departure 2026-06-12 confirmed. His scope was not backfilled by a named
  successor in any Slack channel; Jake Vogel absorbed day-to-day sales leadership, reporting to
  Nick Friedman.
- ⚠️ **"IPO timeline ~18 months" (Drew, 2026-03-26)** was not revisited in any Slack channel in
  the following five months. Treat it as an unrefreshed 2026-03 statement, not a current plan.

**What changed at the company level since 2026-04-24:**

| Date | Event | Why Homes should care |
|---|---|---|
| 2026-04-27 | **EVA launched** (AI escrow agent) + **$40M BlackRock financing** | This is the company's flagship 2026 story, not BBYS. ⚠️ Internally described as "a venture debt refinance deal with blackrock" (Mike Abner, #eva-epd) — **not** an equity round |
| 2026-06 | **RIF** — Wei, Mette, Matt Leddy (6/12); Suzanne Krause, Veronica Aguilar (~6/15–16) | Already in [[team]]; noted here for the company timeline |
| 2026-07-01 | Company-wide comp/title changes effective, **Homes included** | Driven by the Scrappy360 annual review cycle — see [[team]] § Performance review process |
| 2026-07-28 | **Expected-revenue haircuts** on D2C (GVG early-stage −30%, investor-only rev share −35%) | Methodology break in any D2C revenue series crossing 2026-07-27 |
| 2026-07-31 | July closed at **295 IRUCs — best month ever** (74 IRUCs in the best week) | Beats the 216 March record cited above; the March "first time breaking 200" note is superseded |
| 2026-07-31 | **HELOC unavailable in FL, MI, MT** pending servicing licenses | Directly affects LO/LRM eligibility conversations. Equity Boost also still not live in TX as of 2026-06-25 |
| 2026-08-04 | Board meeting | Outcome not in Slack |
| 2026-08-14 | **HomeLight Pro** kicked off — a ~$1,000/mo agent bundle targeting end-October | 🔴 Proposed scope includes **HL Homes placements**, an **agent-branded HELOC Card**, and a title/escrow credit. No Homes representative is in `#homelight-pro`. See [[projects/2026-08-29-slack-company-6mo]] |

**Products outside Homes that now touch Homes:**

- **"The Card"** — HELOC-backed consumer card. **Marc Kaplan owns it**, including capital markets
  and bank compliance (DM 2026-08-18). Two logos in trademark as of 2026-07-17; ACH drawdowns via
  Pesto; its own customer hotline. LO enthusiasm reported after Originator Connect
  (Ashwin, #proj-d2c-heloc 2026-08-20).
- **HomeLight Home Loans (HLHL)** is the **servicer** of the HELOC leg — clients get a payment
  portal at close (Marc Kaplan, #tls-irx-strogus 2026-08-24).
- **HLM** (formerly `disclosures.io`, now `hlm.homelight.com`) and **HomeLight Offers** are the
  company's agent SaaS P&L, tracked as Net MRR in a dedicated Periscope model.
