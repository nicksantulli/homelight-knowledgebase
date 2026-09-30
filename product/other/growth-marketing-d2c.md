---
last_updated: 2026-08-29
status: current
source: Slack channels (growth/marketing/D2C), 2026-03-01 to 2026-08-29
scope: D2C consumer funnel, paid acquisition, SEO/content, email infra, webinar/B2B LO motion, experiment culture
source: HomeLight-Vault/context/growth-marketing-d2c.md
imported: 2026-09-29
---

# Growth, marketing, D2C

The vault had no dedicated growth/marketing doc before this one — greenfield. Built from a
6-month sweep (Mar 1 – Aug 29, 2026) of `#growth-*`, `#d2c-*`, `#hotjar-*`, `#equity-boost-*`,
`#webinar-*`, and adjacent channels. Read with [[bbys-overview]], [[hubspot]],
[[data-bridge-automation-inventory]], [[tools-and-stack]].

> ⚠️ This is almost entirely **stated** (Slack chatter), not **observed** (code/data). Metrics
> below are what people posted in-channel, not verified against Redshift/Periscope directly.
> Treat exact numbers as directional.

## Org shape (inferred from channel activity)

- **Sumant Sridharan** — runs Growth/D2C; the dominant voice in `#growth`, `#growth-sem`,
  `#growth-seo`, `#proj-d2c-heloc`, `#proj-listing-boost`. Reviews ad creative, SEO PRs, and
  product UX personally, down to CTA copy.
- **John Van Slyke (JVS)** — SEM/paid lead; owns the monthly D2C MBR deck and channel.
- **Felipe Martinez** — analytics/ops across SEM, paid social, partners; runs the AI post-call
  qualification pipeline.
- **Randy Ginsberg, Ping Tsai** — SEM buyers (Google/Bing).
- **Reynold Krieg (Infinity Media LA)** — external paid-social agency contact (Facebook/Meta).
- **Ralph Tumaneng** — sole SEO engineer; posts daily technical SEO metrics.
- **Bill Raney, Leti Cabrera** — affiliate/partner channel management and weekly reporting.
- **Sue Suanco, Christina Farley, Alexandra Lee, Andrew Soss, Mette Adams** — B2B/LO-facing
  lifecycle email, webinars, Elite Lender Program (`#growth-email`, `#lifecycle-team`).
- **Alma Chen, Verónica Aguilar, Anirudh Bhutani, Ashwin Dayal, Sai** — D2C HELOC and Listing
  Boost product builds (home value quiz, Replit prototypes).
- 🔴 **`#equity-boost-launch` and `#content-team` had zero human messages in the 6-month
  window** — either the work moved elsewhere (DMs, other channels) or these initiatives are
  dormant. Worth confirming directly rather than assuming activity from channel presence.

## 1. The D2C funnel — how consumer leads arrive and where they go

### Acquisition channels (all feed `hl_all_leads`, tagged by `marketing_source`/`utm_source`)

| Channel | Slack home | Who runs it | Notes |
| --- | --- | --- | --- |
| SEM (Google/Bing) | `#growth-sem` | JVS, Randy, Ping, Felipe | Daily RoAS/CPL/lead-count posts |
| SEO (organic) | `#growth-seo` | Ralph Tumaneng | Daily technical metrics |
| Paid social (Meta/TikTok) | `#growth-paidsocial` | JVS, Reynold Krieg (agency) | Weekly report bot (Leti) |
| Display | `#growth-paidsocial` | Felipe Martinez | Rolled into same channel |
| Partners/affiliates | `#growth-partners` | Bill Raney, Felipe, Leti | ~15 named partners, weekly RoAS |
| Content Cognition (co-reg) | `#growth-partners` | Bill Raney | Marked as partner `cc`, not a lead form |

🔑 **Persona split at intake is UTM-driven.** Facebook/TikTok lead ads set `utm_term` to
`simplesale`, `agent`, or `hv` depending on ad creative — this becomes the
`marketing_persona`/persona-definition attribute downstream
(`#growth-paidsocial`, 2026-03-31, Felipe Martinez).

### Lead → referral funnel (D2C consumer leads)

Leads land as `hl_all_leads` rows with `marketing_channel`/`marketing_persona`, then flow
through **auto-intro** logic to agents and/or investors:

1. Lead created (SEM/SEO/paid social/partner form).
2. Phone verification gate — **as of 2026-03-11, Google conversion value only fires on
   phone-verified leads** (18% of Simple Sale leads were unverified and worth ~1/10th the
   value; JVS, `#growth-sem`, 2026-03-11).
3. Persona/UTM routing decides agent-only, investor-only, or dual-path treatment.
4. Auto-intro to agent(s) and/or ping-post investor network (rev-share vs PPL —
   pay-per-lead — investors are tracked separately).
5. Referral claim → SRM/LSM/CA follow-up → meeting scheduled (MS) → conversion.

Weekly reported metrics across every paid channel: **gross leads, referrals, 3-day referral
rate ("Ref Rate 3D"), RPL (revenue per lead), RPR (revenue per referral), contact speed
<5min%, and profile mix (% Agent Only / % Investor Only / % Dual Path)**. These are the
standard KPI set — any channel report not using this vocabulary is nonstandard.

⚠️ **"Referral" here means agent/investor referral, not the BBYS/HubSpot "referral" concept**
— confirm which system before comparing numbers across [[hubspot]] and D2C dashboards.

### D2C → BBYS/agent-referral conversion

- D2C leads that don't want an agent match or aren't selling get funneled toward **HELOC
  Card** or **agent referral** as fallback monetization (see §HELOC below).
- `sumant`, `#proj-d2c-heloc` (2026-05-18): *"if a consumer is not interested in an agent or
  an investor, we try to pitch them the HELOC card (it represents no risk for us)."*
- No evidence found in-channel of a formal, documented D2C→BBYS handoff SOP; the mechanism
  appears to be ad hoc pitch-if-nothing-else-works, run through Slack coordination rather than
  a system-enforced flow. **(inferred)**

### MQL / SQL definitions — 🔴 institutionally fuzzy

- Andrew Soss (marketing), DM to Nick Santulli, 2026-03-27: *"I should know this lol, but what
  actives are considered MQL and not SQL lol"* — Nick's reply: *"Shit dude I can't remember lol
  I know like registration, education I think, maybe a few others."*
- By 2026-08-18, Guilherme Batista was actively building **new MQL trigger workflows** in
  HubSpot (`#the-boyz`: *"I believe the workflow will trigger new MQL events today"*), and a
  Aug 24 HubSpot ticket references *"mqls using ai"* — suggesting MQL scoring/events were being
  rebuilt from scratch as of Q3 2026, not maintained from an older definition.
- A recurring **"Demand Gen & Sales Activity Review"** biweekly meeting (Andrew/Mette/Marc/
  Wei/Gui) reviews MQL/SQL volume + conversion for leadership reporting (daily-prep note,
  2026-05-26) — the definitions exist somewhere in that meeting's materials, not in Slack.

⚠️ **No canonical MQL/SQL definition doc was found in Slack.** If one exists it's likely in
HubSpot itself (lifecycle stage criteria) or a deck, not searchable here. Flag for direct
confirmation with Andrew Soss/Mette Adams.

## 2. Marketing campaigns, Mar–Aug 2026, and measured results

### SEM (Google + Bing)

- Daily cadence: Felipe Martinez posts Overall SEM RoAS/leads/spend/CPL/RPL for Google and
  Bing non-brand campaigns every morning.
- **RoAS ranged ~0.5–1.8x through March**, trending toward ~1x by mid-year; team explicitly
  treats **RoAS ≥ 1.0 as the bar**, with escalation ("this thread can't be this quiet given
  where sem is right now" — sumant, 2026-03-13) when it dips.
- Major structural change: **JVS's Competitor Campaign Brief (2026-03-06)** proposed
  consolidating 9 fragmented competitor campaigns (43 ad groups) into 1 campaign / 5 ad groups
  by product line (Home Value, Cash Offer, Agent Match, BBYS, FSBO) — 30-day baseline was
  $10.4K spend, 319 conversions, 1.20x RoAS.
- **Bid caps** rolled out account-wide starting ~2026-03-12 (sumant pushed for an $8 max after
  seeing $85 CPCs) to control runaway CPCs on intent campaigns.
- **Phone-verification gating** for Google conversion values (2026-03-11) was a direct fix to
  inflated/misleading conversion signal.
- Search Partner Network test launched 2026-03-11 on two campaigns (Simple Sale High Value /
  BBYS), 14–30 day test window.
- Monthly **SEM MBR** (Ping Tsai's notes, 2026-03-05) flagged: reporting-accuracy concerns,
  rising Google CPCs, Bing RoAS declining YoY, meeting volume up but RPL down 53% YoY (lead
  quality/value concern), and a stated $18K/day revenue target vs. ~$12K/day actual.

### Paid social (Meta/TikTok)

- **UGC (user-generated-content) ad creative consistently outperformed static ads.** Spike to
  **3.61x total RoAS** (2026-03-30) after shifting spend to UGC; "Lead form" campaigns
  separately ran ~0.96–1.47x RoAS through Q2/Q3.
- Facebook policy constraints shaped product: **can't ask for address directly** (must use
  Meta's autofill or ask "what property are you looking to sell") due to housing-category ad
  policy.
- Recurring reliability failures: Meta lead-ad Zapier integration broke multiple times
  (2026-04-01 duplicate leads, 2026-07-21 all zaps disabled for days by a Meta-side outage).
- Weekly "Paid Growth" report format (Leti Cabrera) — gross leads, referrals, Ref Rate 3D,
  RPL/RPR, contact speed, profile mix — was consistently 25–45% RoAS from June onward, with
  volatility partner-to-partner.

### Partners/affiliates

- ~15 named partners tracked weekly: `gvg_capital`, `we_buy_houses`, `real_estate_bees`,
  `us_news`, `property_leads`, `zeel_media`, `credit_karma`, `realty_com`, `maven`,
  `interest_media`, `cc` (Content Cognition), `benefithub`, `bonus_homes`.
- Aggregate channel RoAS ranged **1.25x–1.92x** across the period; individual partners swung
  wildly week to week (e.g., `real_estate_bees` RoAS moved -80% → +630% → -94% across
  consecutive weeks — small-sample noise, not necessarily signal).
- **AI-based lead-quality scoring** (post-call analysis) is used to diagnose partner RoAS
  drops — e.g., `bonus_homes` investigation (2026-06-19) found 70.1% of AI-analyzed outbound
  calls were outright rejections ("Not Interested" or "Unsubscribe"); only 7.5% were genuinely
  interested. This model migrated from **Gemini 2.0 → Qwen3.5 Omni Flash** in May 2026 when
  Gemini 2.0 was deprecated, and the migration itself was suspected (unconfirmed) of causing a
  spike in leads flagged "not interested" for some partners.
- Partner API data-quality issues recur: malformed `supplimental_data` payloads (Maven,
  Interest Media, March 2026), missing property-type fields (`bonus_homes`), address
  deliverability issues (`cc`).
- 🔑 Bill Raney (partner channel owner) grew partner gross profit from **$10K/month to
  $47K/month** over his tenure (as of 2026-03-03), scaling toward a $60K target.

### D2C Monthly Business Review (MBR) — the standing measurement structure

`#d2c-biweekly-updates`, hosted by JVS, is where D2C-wide results roll up monthly. Recurring
sections (evolved over the period): **Financials/win-rate, SEM, MLS (data), Aged leads, SaaS
revenue (US News, Offers), PPL, `d.io` (see Listing Boost below), Voice AI, HC (HouseCanary)
conversion test, Lead Decline reason launch.** This is the closest thing to an official D2C
scorecard — cross-reference future D2C questions against these MBR decks rather than
individual channel chatter.

⚠️ March MBR reconciliation issue: a PPL number moved from **>$140K to $70K** in the deck the
day before the meeting (Justin Tran flagged it), later clarified as an Accounting-vs-Collections
number mismatch — a live example of the "many numbers disagree across systems" pattern
documented in [[bi-metrics-definitions]].

## 3. Webinar / B2B motion for LO acquisition

- **UWM AE Q&A webinar series** (`#webinar-uwm-ae-qa-series`) — weekly, LSM-hosted, rotating
  pairs of LSMs. Registrants were UWM AEs/LOs. **Sunsetted by John Labrada in June 2026** in
  favor of 1:1 demo links per AE (*"we're sunsetting that webinar. Any chance you can send them
  a demo link instead?"*) — signal that the group-webinar format wasn't converting well enough
  to justify the ops overhead. At least one session had zero attendees (2026-06-04).
- **Elite Lender Program** (`#webinar-elite-lender-program`, owned by Mette Adams) — a distinct
  LO-tier enablement track with a 1-pager asset; very low Slack traffic (2 messages in 6
  months), so most of this program's activity likely happens outside Slack (email, live calls).
- **"Marie Webinar"** and **"Power Calculator Webinar"** — recurring named webinar programs
  referenced in Sue Suanco's weekly email-ops updates (`#growth-email`), each with dedicated
  RSVP, reminder, and two-variant follow-up emails (attended vs. registered-but-no-show,
  and "no contract" vs. "on platform" segments). These appear to be the actual active B2B
  webinar cadence, run through HubSpot email, not Slack coordination.
- 🔑 **HELOC/Equity Boost prequal rate ~86.4%** was reported as a headline LO-channel metric in
  a Marc Kaplan HELOC EoW report (referenced 2026-05-26 daily prep) — worth pulling that report
  series directly if pursuing LO-side HELOC performance.

## 4. Experiment culture — A/B tests observed

**No dedicated experimentation platform (Optimizely, GrowthBook, LaunchDarkly, etc.) was
found in any channel.** Every test observed was either:

1. **Traffic-percentage feature flags shipped directly in HAPI/sales-app PRs**, e.g.:
   - BBYS HELOC application quiz: **3 variants (AB1/AB2/AB3)** shipped by Thiago Manfrin
     Casagrande (PR #18713, ~2026-04-25) to find the best conversion path.
   - Home-val quiz HELOC-interest question: launched at **10% traffic**, with 30% of that 10%
     shown the HELOC question (Alma Chen, 2026-05-29).
   - BBYS consent-flow experiment: **three-way split — Control 70% / "0313" variant 20% /
     consent variant 10%** (`#d2c-epd`, 2026-03-24).
   - SS Quiz social-proof/brand addition: launched at 10% traffic (2026-04-01).
2. **Ad-platform native experiments** (Google Search Partner Network test, Bing phrase-match
   test, Facebook UGC-vs-static creative tests) — tracked via Periscope dashboards, not a
   shared experiment registry.
3. **Email subject/CTA A/B tests inside HubSpot** — e.g., Lender Insight Survey email 2 CTA
   test (Sue Suanco, 2026-06-13).

⚠️ **Sample-size skepticism is explicit and recurring.** Andrew Soss, on an LSM-facing asset
test: *"It's not a statistically significant population to A/B test"* (`#the-boyz`,
2026-06-03) — followed by an informal show of hands instead ("the guys like mine and the
females like Mette's"). This suggests **rigor varies a lot by team** — engineering-run product
experiments use real traffic splits and dashboards; some marketing/creative decisions are
settled by informal preference polling.

🔑 **`AbTest` is one of HAPI's 77 mounted engines** ([[repo-hapi]]) and HAPI also ships a
`Split` A/B dashboard — but no Slack channel referenced using either for a D2C growth test in
this window. The infrastructure may exist without being the tool growth actually uses.
**(inferred — worth confirming directly against the `Split` dashboard.)**

## 5. Email marketing infrastructure

**Three systems, split by audience, with active consolidation pressure:**

| System | Audience | Owner | Status |
| --- | --- | --- | --- |
| **SendGrid** | Transactional/operational (password resets, deal-stage notifications, referral alerts, comms-journey emails) | HAPI engineering (Joel Shurtleff, Faaz Shaikh, Jose Herrera, Taylor Wong) | Active, triggered from HAPI code |
| **Iterable** | B2C/consumer lifecycle — described by Christina Farley as *"vast majority of what's live on the B2C side of the biz is SMS flows... no dashboards where we've been keeping tabs on them"* (2026-03-19) | Unclear single owner; `IterableMarketing` is a HAPI engine | Active but under-instrumented |
| **HubSpot** | B2B/LO-facing lifecycle (GCI reports, Elite Reports, webinar RSVP/follow-up, lender surveys, sales sequencing) | Sue Suanco (execution), Andrew Soss/Mette Adams (strategy), Christina Farley/Alexandra Lee (prior owners, handed off ~May 2026) | Active, growing |

🔑 **There is an explicit (half-joking) intent to migrate SendGrid → HubSpot and retire
SendGrid entirely.** Guilherme Batista, DM with Nick Santulli, 2026-03-12: *"I feel we can
migrate them to HS too and remove sendgrid"* — Nick: *"Oh 100%, my ideal would be moving them
all over."* No formal project/timeline found for this migration as of Aug 2026; treat as a
stated intent, not a committed roadmap item.

⚠️ **SendGrid click-tracking rewrites URLs** (`equity.homelight.com` → `url3132.homelight.com`)
and serves a `sendgrid.net` TLS cert on the rewritten link — this has caused recurring LO
support tickets (password-reset flow perceived as broken/suspicious) since at least 2025,
unresolved as of 2026-08-04 (`#centralized-homes-epd-support`). A fix (disabling SendGrid
click-tracking) was proposed but not confirmed shipped.

🔴 **Comms Service race condition** caused a production incident (2026-08-11): SendGrid
webhooks tried to create Comm Events before async Comms Service entities existed, triggering
Sidekiq retries that amplified an 8,000+ email batch job (`ReminderToUpdateAgentLeads`,
weekly Monday 9am job) into a HAPI outage. Fixed same-day by Jose Herrera (incremental backoff
+ case-insensitive email matching fix).

**Google Workspace mail is also part of the outbound surface** — `casteam@homelight.com` was
being rate-limited by Google due to high-volume CC/BCC from automated SendGrid sends
(2026-08-28), resolved by removing the CC/BCC rather than fixing the volume.

## 6. SEO / content ops

**Ralph Tumaneng is effectively a one-person SEO engineering function**, posting daily metrics
in `#growth-seo`: blog indexed-page count, organic clicks/impressions/ranking position,
Core Web Vitals (LCP/CLS/FCP), crawl-request failure rate, and a running list of PRs against
`homelight/homelight`.

- **Blog indexed pages grew from ~4,992 (early March) to 5,597 (late March)** — steady,
  incremental growth, not a step-change event.
- Recurring technical SEO fixes: meta description rewrites, page-title dedup, `noindex` on
  thin `/homes/county` pages, WebP image conversion, explicit image width/height, priority
  loading for above-fold images, internal-linking strategy (route authority from
  top-performing blog posts to underperforming/non-indexed pages).
- 🔴 **Top-agent city pages flagged as "soft 404s" by Google Search Console** despite HTTP 200
  — pages with no agents listed read as thin content. Ralph's fix-in-progress (2026-03-20):
  internal links to nearby state/county pages rather than hard-404ing (which would block
  future indexing once agents populate). **This is the agent-directory page problem** — no
  evidence in-channel that it was fully resolved by end of window.
- 🔴 **Content team reports posts going invisible from Google search post-migration** — Richard
  Haddad (content), 2026-03-26: multiple blog URLs that should rank for their target queries
  don't appear even on direct `site:homelight.com` searches. Framed as ongoing content-team pain,
  not resolved in-window.
- **IP blocklisting** (Fastly) used repeatedly to fight bot/scraper traffic inflating crawl
  stats — 50 IPs at a time, multiple rounds through March.
- Google algorithm updates (spam update + core algo update, both March 2026) tracked for
  before/after impact but no full readout found in-channel.

### Listing Boost — new agent-marketing product (not primarily an SEO play)

`#proj-listing-boost` (started 2026-04-01, still active as of 2026-08-26): an **agent-facing
marketing-asset generator** — social media posts, postcards, flyers, feature sheets — built by
Alma Chen on Replit (`listing-marketers.replit.app`), championed by Sumant Sridharan, branded
by Verónica Aguilar. Feature list tracked in a shared spreadsheet; Stripe + SSO login being
wired for production readiness as of April 2026.

- 🔑 **As of July 2026, the plan is to fold Listing Boost into `D.io`** ("the biggest
  conclusion I'm coming to is we can just combine the listing boost product into D.io to
  create a universal listing platform rather than 2 separate products" — sumant,
  `#project-dio20`, 2026-07-21). `D.io` recurs in D2C MBR agendas (roadmap-to-wider-usage,
  AWS cost savings) — it's a live, budgeted product, not a prototype. **What `D.io` actually
  is was not established in this sweep — flag for direct follow-up.**
- Handed off to Angelina Michadjaja for continued ownership as of 2026-08-26 (Replit account
  access requested/granted).
- Lauren Garnel raised a real gap during ideation: agents are being pushed toward more social
  video but are "uncomfortable/embarrassed about recording videos of themselves" — proposed
  AI-video generation (Synthesia-style) as a Listing Boost feature; not confirmed built.

## Open questions

1. ~~What is `D.io`, concretely...~~ **Resolved 2026-08-29 — see addendum below.**
2. ~~Is there a canonical MQL/SQL lifecycle-stage definition...~~ **Resolved (as "no") 2026-08-29
   — see addendum below.**
3. Was the SendGrid→HubSpot migration ever scoped formally, or does it remain an informal
   intent between Gui and Nick?
4. ~~Did the SendGrid click-tracking / TLS-cert LO support issue ever get fixed?~~ **Partially
   resolved 2026-08-29 — see addendum below.**
5. What killed `#equity-boost-launch` and `#content-team` channel activity — did the work
   move elsewhere, or did these initiatives stall?
6. Is `AbTest`/`Split` (HAPI's in-repo A/B infrastructure, see [[repo-hapi]]) actually used by
   growth, or is every observed test a bespoke traffic-percentage flag?
7. What is the actual, current MQL/SQL definition set referenced in the 2026-08-18 "new MQL
   events" workflow rebuild — and does it supersede whatever existed before? **Partially
   resolved 2026-08-29 — see addendum below: the 8/18 change is a workflow trigger, not a
   definitions doc.**

## Gap-chase addendum — 2026-08-29

> Source: targeted Slack search/read, following up specific open questions this doc and a
> companion sweep left unresolved. Read with [[team]] and
> [[projects/2026-08-29-slack-product-pods-6mo]].

### 🔑 `D.io` = Disclosures.io, confirmed

The hypothesis was right. `D.io` / `d.io` is universally used in Slack as shorthand for
**Disclosures.io** (`disclosures.io`, internally "HLM" — HomeLight Marketplace-adjacent, the
agent-facing disclosure-package/e-signature product HomeLight acquired). Confirmed multiple
ways:
- Nick Santulli himself, `#hubspot-help`, 2026-08-24: *"Short hand for disclosures.io lol"* and
  *"given all the d.io merges"* (re: HubSpot contact-record merges from the product).
- Pooja Mithani (D2C product), `#d2c-epd`, 2026-07-28/07-31: *"we need to update D.io's Terms
  of Service"* — links resolve to `disclosures.io/terms`.
- `#project-dio20` (`C0B5GD8MG3V`) is the live product-eng channel for it — security fixes,
  domain migration (`app.disclosures.io` → `hlm.homelight.com`, cut over 2026-08-19), a July
  2026 PLG (product-led-growth) push with a 14-day free Pro trial, Stripe-adjacent billing.
- Engineering side: a July 15–16 2026 security remediation (`#eng-security`, Jose Herrera)
  fixed a **public-read S3 bucket exposing user-uploaded PII documents** and upgraded deprecated
  NodeJS v10 Lambdas — this is the same incident hapi's `DisclosuresIoService` hypothesis
  pointed at, confirming code and Slack agree on ownership. (Link syntax fixed 2026-08-29 —
  was a malformed wikilink with backticks inside `[[ ]]`; no vault doc actually documents
  `DisclosuresIoService`, so this is left as inline code, not a link. See
  `working/2026-08-29-critic-technical.md` §1.8.)
- **Owner:** Sai (`sai@homelight.com`) is the de facto product lead posting most updates and
  driving the 2026-08-25 CDN cost-fix conversation; Átali Silva is lead eng; Angelina Micha
  Djaja (D2C PM) runs city-rollout and MLS-SSO-usage coordination; Jose Herrera owns
  security/infra.
- 🔑 Confirms the prior open question's framing: `D.io` is a **live, actively developed,
  budgeted product** (security fixes, PLG features, cost optimization, TOS updates all
  happened within the last 5 weeks of this sweep), not a stalled line item. The planned
  "combine Listing Boost into D.io to create a universal listing platform" (Sumant,
  `#project-dio20`, 2026-07-21) had not visibly started as of this pass — no Listing Boost
  code/product references found in `#project-dio20`.
- **Not yet found:** an explicit AWS-cost-savings figure or roadmap doc tying back to the D2C
  MBR's "d.io savings" line item — the 2026-08-25 CDN routing fix (`~$880` in avoidable
  `DataTransfer-Out-Bytes` cost since 2026-07-16) is the closest concrete number in-channel and
  is plausibly what that MBR line refers to, but no MBR deck itself was located to confirm.

### 🔴 MQL/SQL: no canonical definition exists — confirmed tribal, and a known, named gap

This was **not previously codified** as of this sweep, and multiple people have said so
explicitly:
- Andrew Soss's own meeting notes (`#the-boyz`, 2026-05-07): *"There's significant confusion
  around MQL, SAL, and SQL terminology, everyone seems to have a different definition. We need
  a simple, dictionary-style reference document that defines each term clearly."* His to-do
  list includes "Define each lifecycle stage clearly for the LSM team" and "Build out the
  funnel stage reference document" — **no evidence in Slack that this doc was ever built.**
- Guilherme Batista (DM, 2026-01-30): *"We have a field called 'Lifecycle Stage', but since our
  business model is not a SaaS one, one contact can become a MQL/SQL more than once"* — i.e.
  HubSpot's out-of-box Lifecycle Stage semantics (monotonic, SaaS-funnel shaped) don't cleanly
  map to HomeLight's repeat-eligible LO contacts. This is a structural reason a canonical
  definition is hard, not just a documentation gap.
  ⚠️ **Contradicts the standard HubSpot mental model** most non-technical readers of this file
  will have (Lifecycle Stage as a one-way funnel) — treat any MQL/SQL count pulled from
  Lifecycle Stage with that caveat.
- The closest thing to a working definition is **stated, informal, and from one person**: Jake
  Vogel (DM, 2026-06-24) describes an SQL as an LO showing "meaningful engagement" (email
  opens/clicks, site/tool usage — e.g. Purchase Power Calculator, webinar registration/
  attendance, repeated content interaction) combined with program fit (broker/retail, Modex
  score) — **not** formally written down anywhere in HubSpot or a shared doc, per this search.
- The 2026-08-18 "new MQL events" change (Guilherme Batista, `#the-boyz`) is a **workflow
  trigger going live**, not a definitions document — it operationalizes *an* existing scoring
  logic but doesn't answer what that logic actually is in plain language.
- **Owner (inferred):** Guilherme Batista appears to own the marketing-automation/scoring side
  (workflow builds, "Marketing Score" definition requests directed at him); Jake Vogel owns the
  sales-side qualification judgment for LSM/LO outreach. No single named owner of a written,
  cross-functional definition.
- 🔴 **Net: if any report or deck cites "MQL" or "SQL" counts, ask what field/workflow it's
  actually pulling from** — there is no single source of truth, and the team has known this
  since at least January 2026 without resolving it.

### SendGrid TLS-cert / click-tracking issue: fixed for the reported case, not globally

The specific complaint that reached engineering (`#centralized-homes-epd-support`,
2026-08-04/05) was an LO-portal **password-reset email** showing a `sendgrid.net` cert instead
of `homelight.com` because SendGrid's link-rewriting (click tracking) swaps the real domain for
a `sendgrid.net`-backed tracking link. Root cause confirmed by Jose Herrera: *"SendGrid...
rewrites links from equity.homelight.com... to url3132.homelight.com... that URL returns a cert
from sendgrid.net."*
- **Fix shipped and confirmed same day (2026-08-05):** click tracking disabled specifically for
  the password-reset email template. Jose: *"Just tested the change, confirmed that the
  password reset email no longer comes with click tracking... As far as the password reset
  email goes, we are good to go."*
- ⚠️ **This is a point fix, not the systemic one.** Jose flagged in the same thread that a
  broader "click tracking solution... talked about several weeks ago" (referenced Slack link
  from mid-June 2026, channel now inaccessible to re-verify) would fix this class of issue for
  *all* HomeLight-sent email, not just password resets — Mike Abner was asked to help implement
  it but no confirmation found that the broader fix shipped. Joel Shurtleff separately noted
  this exact cert error has been hitting one partner (CCM) LOs "intermittently since last
  year." **If any other transactional email type (not password-reset) still uses SendGrid
  click tracking, the same cert-mismatch symptom likely still reproduces.**

## ⚠️ "US News" is two systems (2026-08-29)

The name collides: (1) the **organic co-brand lead-gen site** documented here and in
[[repo-homelight-growth]] (Jekyll → S3), and (2) a **Stripe-backed paid agent sponsorship
product** built in HAPI May–Aug 2026 (Emmanuel Diaz; Production Index ≥1.0 qualification,
<10-transaction pre-filter). Ask which before answering any "US News" question.
