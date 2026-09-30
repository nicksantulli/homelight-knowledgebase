---
last_updated: 2026-09-01 (**Partner resource-link sync completed.** Marketing Hub is the partner-facing resource center: write it to Company `resource_center` and Partnership `marketing_hub`; write Landing Page to `landing_page` on both objects. Canonical sheet and local 270-row snapshot are documented in [[projects/2026-09-01-partner-resource-links-hubspot-sync]].) Previous: 2026-08-29 (**Nous archive-tail addendum: Zavvie warehouse-funding proposal (new
to this vault) and Luminate TPO warehouse-line mechanics ($5M line, $1M restricted deposit)
surfaced from a permission-restricted Nous collection, search-snippet only, mid-May 2026
snapshot — see §"2026-08-29 — Nous archive tail" below. Chase/JPM pre-pause history was searched
for and not found.**) Previous: 2026-08-29 (**DM sweep addendum: John Labrada's wholesale partner tracker built,
still Sheets-only; a new Property Radar data-access initiative floated.** Source: DM D04PPJA8GBF
(Labrada↔Tulli), 2026-08-28, read-only. (1) **Wholesale partner tracker built as the mirror of
Kim's already-HubSpot-synced Retail Partner Tracker** (see the 2026-08-28 entry below) — same
columns, pre-filled from HubSpot with the 91 partnerships where Labrada is Partnership Owner
(Lender/Tier/Rep/NMLS). **Unlike Kim's sheet, this one has not been synced back into HubSpot as
of 2026-08-29** — Status is 0/91 filled, Comp plan is a free-text field that exists only on the
sheet (not a HubSpot property), and 26/91 rows have no Rep assigned. Covers only the 91
*partnership* records, not the ~1,517 wholesale *company* records (of which the Aug-19 allocation
file resolves 1,013, with 404 historically marked `HOLD - Tejas decision`; that hold was superseded
by the Aug. 31 rule — Tejas keeps all verified Wholesale/Broker LOs plus the approved 76-contact
Retail top-20 keep set. See [[risk-register]] row 3.)
(2) **Rep-book concentration flagged, not yet acted on:** within Labrada's 91 partnerships, the
named-rep split is **Tejas 23 / TJ Sims 12 / Marisa Drake 6 / Richie Helali 1**, vs. **Tierney
Izar + Kara Kleingarn + Brian Banes 12 combined** — i.e. one rep (Tejas) alone holds roughly as
many partnerships as three others combined. Tulli flagged this as "might be exactly right if
legacy, but if not, Sept 1 is the moment to fix it" — no decision recorded. (3) **Property Radar
data-access idea, undecided:** Labrada's pitch (successor to an earlier "Cotality" idea) —
HomeLight buys ~100 Property Radar seats (~$100/seat/mo) for Elite LOs so they can identify
homeowners with high sell/buy probability and sufficient equity for BBYS targeting. Discussed
with Tulli 2026-08-28 as "theoretically... for lead packages to Elite LOs"; not committed, no
budget owner named. Time-bound detail: [[projects/2026-08-29-dm-decisions]].) Previous: 2026-08-29 (**Targeted follow-up: UWM root cause, TLS/Averra technical scope, hapi#19718
resolution confirmed.** Source: Slack, targeted `slack_search_public_and_private` across `#ent-uwm`,
`#ent-the-loan-store`, DM `D04PPJA8GBF` (Labrada↔Tulli), `#homes-ops-and-efficiency-pod`,
`#envoy-irx-barber-102-angus-ct-va-express`, read-only, 2026-08-29. Four updates to the entries
below. (1) **UWM root cause found** — a same-day DM from Labrada to Tulli (2026-08-25 10:45 MDT,
`D04PPJA8GBF`): *"Their marketing/compliance never actually told us we could use their name,
image, or nmls in anything we made lol."* Not a formal regulatory complaint — HomeLight never had
UWM's written consent for the co-branded page in the first place. State: **fully remediated same
day** (logo/NMLS/URL/link-preview all stripped by ~17:50 MDT; submission link deliberately
retained per Labrada). See updated UWM entry below. (2) **TLS→Averra technical scope confirmed
narrow**: `#ent-the-loan-store` thread (2026-08-18) shows Labrada explicitly choosing to **retain
the existing submission URL** (`equity-app.homelight.com/application-flow?partner=tls`) and swap
only the logo — no mention anywhere of `partner_slug`, pricing templates, keyword configs, or deal
channels changing. **(inferred)** the migration plan is marketing-asset-only; nothing found
suggests `partner_slug: "tls"` changes to `averra` or similar. (3) **hapi#19718 is RESOLVED,
same day**: Taylor Wong's fix thread (`#homes-ops-and-efficiency-pod` / the Envoy deal channel,
2026-07-31) ends with Taylor confirming *"backfill is complete. i ran this silently so we didn't
generate a bunch of slack notifications"* (~15:47 MDT) — the ~50 affected leads' `pricing_model`
was restored to `variable`, same day as Bruno's report. Taylor also flagged two follow-ups:
code fix so future backfills don't touch `pricing_model` pre-launch, and adding pricing templates
for the missing partners. Status was "unconfirmed" in the prior sweep entry — now confirmed
closed. (4) **Bay Equity→Rocket cleanup**: no new evidence found either way in this pass; the
prior entry's "ran through at least 2026-04" stands as the most recent confirmed status — treat
as **still open**, not confirmed finished.) Previous: 2026-08-29 (**Slack 6-month sweep (2026-03-01 → 2026-08-29) of #ent-* partner channels.** Six findings worth knowing before anything else in this file: (1) **Bay Equity is dead as a partner** — acquired by Rocket Mortgage (via Rocket's Redfin purchase, ~July 2025) and HomeLight cannot re-partner because Rocket conflicts with the UWM relationship; LO email-migration cleanup continued into 2026. (2) **TLS is renaming to Averra Financial, Inc.**, legal effective 2026-09-30, public go-live 2026-10-01 — a new `#ent-the-loan-store` channel was stood up 2026-08-18 to run this. (3) **UWM triggered a compliance-driven branding strip** on 2026-08-25 — UWM logo, NMLS number, and "uwm" removed from the co-branded landing page and its URL on short notice ("we got in a little bit of trouble with UWM"). (4) **The variable-pricing partner list has live drift**: prod Flagsmith `variable_pricing` was `[apm, envoy, arbor-financial, nfm-lending]` as of 2026-08-04/08-13 — **21st-century was NOT on it**, contradicting [[bbys-pricing-engine]]'s claim that the seed catalog already has 5 partners. A July 2026 backfill bug (hapi#19718) separately wrote `flat_fee` onto ~50 VP-eligible leads across tls, apm, nfm-lending, envoy, uwm. (5) **`#homes-partner-mismatch` has posted zero messages since 2026-02-05** — the Homie bot mismatch alert appears dead, not just quiet, for the entire sweep window. (6) SWBC's named-partner rollout (branded intake URLs) was still "not live" as of 2026-08-18 despite hitting Executive Meeting in March. Full detail, per-partner arcs, and channel-ID reference table: [[projects/2026-08-29-slack-partners-6mo]].) Previous: 2026-08-29 (**Gmail 6-month sweep — operational/payment-side facts for four partners, confirming and extending existing entries.** Source: gmail, <email-redacted>, 2026-03-01 to 2026-08-29, read-only. (1) **TLS AE Payout figures observed**: Aug–Dec 2025 = $8,012; Jan–Jun 2026 = $159,000 — both approved by Nick Friedman, submitted by Sarah Lim, processed via Bill.com under the HLHL entity (thread `19f4dcf981f720d2`). This is the first vault record of actual TLS payout dollar amounts. (2) **Orchard rev-share payment gap**: a missed payment (through March 2026 per HL's numbers) was still being reconciled as of 2026-03-30 between Nick Friedman/Tulli and Orchard's Michael Budlow — "my numbers align with yours through September [2025]." Confirms the Orchard relationship in this file is payment-fragile in practice, not just contractually described. (3) **Arbor Financial Group / KMC Financial (Jon Shrum, President) comp-plan change confirmed in writing**: flat **$1,500/deal**, replacing 10bps on departing-residence final sale price — negotiated directly by Tulli, March 2026 (thread `19d021ff0f8824ae`). This is a specific instance of an individual-partner comp override that isn't captured anywhere else in the vault; worth checking whether other partners have similar one-off flat-fee arrangements outside the standard TLS/Orchard rev-share model. (4) **CMG Financial referral-credit dispute** (bridge-lender deal, Request ID 801894, Jul 2026): CMG cannot add loan points to compensate an LO on a bridge product, but HomeLight's referral-credit program can still apply — resolved informally via Sarah Jaka + Tulli, no HubSpot/Partnership-object trace. (5) **Fairway Independent Mortgage is a high-volume operational relationship** not previously broken out with named contacts — see new [[vendor-and-external-relationships]] doc for the COI/payment contact roster (Lori Itrich, Debbie Dorn, and several LOs recurring on deal-closing threads). No contradictions with the existing content below; this is additive detail at the payment-operations layer, one level below the AE/partnership-object layer already documented here.) Previous: 2026-08-28 (**Partnership object field semantics corrected + Slack channel plumbing documented.** Three things this file previously got wrong or omitted. (1) **`lender_sales_account_executive` is the ENTERPRISE AE field**, not a rep field — portal-wide it only ever holds three values: Kim Tanner `589265219` (Retail), John Labrada `54109913` (Wholesale/Broker), Nick Plamondon `331990682` (Builder). The per-partner *rep* is `lender_sales_manager` (multi-owner checkbox). Kim's tracker sheet's "AE" column means the rep, so it maps to `lender_sales_manager`. (2) **`partnership_stage` is the live status field; the Partner pipeline is effectively dead** — `hs_pipeline_stage` is populated on **1 of 186** records, so writing to the pipeline is invisible to every existing report. (3) **`partnership_tier` has no "Tier 4" option** (Tier 1/2/3 only) even though 20 of Kim's 55 tracked lenders are Tier 4; adding it is a REST-only schema write. Also documented: the real 6-value `broker_retail` option list, the `#ent-<slug>` Slack convention, and the fact that **the channel topic carries the HubSpot record URL**, which is a better channel→partnership join key than `slack_channel_id`. See [[projects/2026-08-28-kim-retail-partner-sync]] and [[projects/2026-08-28-partner-channel-sync]].) Previous: 2026-04-09

> 🔴 **HISTORICAL — partnership ended.** Orchard volume below is legacy; the team no longer takes new Orchard business (RR removal 2026-08-21; existing post-Jul-2025 leads still serviced under special comms rules). Do not pitch or forecast Orchard.
type: context
source: HomeLight-Vault/context/partners.md
imported: 2026-09-29
---

# Partner Relationships

> Key partnerships and their operational details. Claude reads this for partner-related work.

## Partnership Custom Object (HubSpot ID: 2-48010378)

### Pipeline Stages
Prospect → Executive Meeting → Legal Review → Ready → Launched → Nurture → Failed

### Key Properties
| Property | Purpose |
|----------|---------|
| `partner_slug` | Short name (e.g., "uwm", "tls") — used for deterministic deal matching. ⚠️ Does **NOT** drive `main_partner__ai_` — see `partner_nickname`. |
| `partner_nickname` | ⚠️ **This is what the calculated deal property `main_partner__ai_` rolls up — not `partner_slug`.** Kebab-case short name (`homeamerican`, `tls`, `clm-mortgage`, `apm`, `nfm-lending`). A partnership created without `partner_nickname` attributes to an **empty string** — the Main Partner association looks correct in the UI but every report reads blank. Verified 2026-08-26. |
| `partner_domain` | ⚠️ **Calculated / read-only** (verified 2026-08-26 — write returns `READ_ONLY_VALUE` 400). Populates automatically from the partnership→company **Main Company** association. To set it, associate the partnership to its company with USER_DEFINED types `120` (base) + `122` ("Main Company"); the domain then resolves from the company record. |
| `partnership_hierarchy` | "Main Partner" or "Sub-Partner (Broker, JV, etc.)" |
| `partner_name` | Full partner name |
| `broker_retail` | ⚠️ **Six options, not two** (verified against the live schema 2026-08-28): `Retail`, `Broker`, `Wholesale`, `Builder`, `Bank/Credit Union`, `Unknown`. It is a **checkbox** (multi-select) — `Retail;Builder` occurs on 4 records. Distribution at 186 records: Retail 89, Broker 70, blank 17, Retail;Builder 4, Wholesale 2. |
| `paid_partner___t_f` | Whether partner receives payments |
| `lo_count` | Number of loan officers |
| `annual_loan_volume` | Yearly loan volume |
| `enterprise_partner` | Boolean — enterprise level |
| `partnership_tier` | Tier 1, Tier 2, Tier 3 |

## Ownership fields — AE vs rep (verified 2026-08-28)

These two are easy to confuse and mean completely different things.

| Property | Type | Meaning |
|---|---|---|
| `lender_sales_account_executive` | owner, single-select | **The enterprise AE who owns the channel.** Portal-wide it only ever holds three values. |
| `lender_sales_manager` | owner, **checkbox (multi)** | **The rep(s) working the partner day to day** — Richie / TJ / Marisa / Tierney / Kara / Brian / Tejas. |
| `referring_lsm_ae__if_applicable_` | owner, single-select | Who referred the partnership in. Informational. |
| `hubspot_owner_id` | owner | Barely used on this object (4 records). Not the ownership signal. |

**The only three enterprise AEs:**

| AE | Owner ID | Channel | Book (2026-08-28) |
|---|---|---|---|
| Kim Tanner | `589265219` | Retail | 78 partnerships |
| John Labrada | `54109913` | Wholesale / Broker | 91 partnerships |
| Nick Plamondon | `331990682` | Builder | small |

⚠️ **When a tracker sheet says "AE" it usually means the rep**, i.e. `lender_sales_manager`.
Kim's Retail Partner Tracker does exactly this. Mapping that column to
`lender_sales_account_executive` would wipe the enterprise-AE split.

## Stage and tier gotchas (verified 2026-08-28)

- **`partnership_stage` is the live status field.** Options: `Prospect`, `Executive Meeting`,
  `Legal Review`, `Ready`, `Launched`, `Nurture`, `Failed`.
- ⚠️ **The Partner pipeline is effectively dead.** `hs_pipeline_stage` is populated on **1 of
  186** records. Writing stage to the pipeline instead of `partnership_stage` is invisible to
  every existing report. Do not "fix" this by starting to write the pipeline.
- ⚠️ **`partnership_tier` has no `Tier 4` option** — only Tier 1/2/3 — despite 20 of Kim's 55
  tracked lenders being Tier 4. Adding an option is a **schema** write, so the HubSpot MCP
  cannot do it; it needs `PATCH /crm/v3/properties/2-48010378/partnership_tier`.
- There are **two** tier fields: `partnership_tier` ("Partnership Tier") and `tier`
  ("Estimated Tier"). Both partly populated, neither authoritative by default. `partnership_tier`
  is the one Kim's sheet maps to.

## Partner Slack channels

Every enterprise partner gets a `#ent-<slug>` channel, auto-created by the **HomeLight Homie**
bot. Two facts make these machine-usable:

1. **The channel topic carries the HubSpot record URL** —
   `HubSpot Partnership: https://app.hubspot.com/contacts/4744876/record/2-48010378/<recordId> ; Domain: ...`
   This is a **better join key than `slack_channel_id`**: authoritative, self-healing, and it
   works for channels whose id was never written to the CRM.
2. **`slack_channel_id` exists on the Partnership object** — 137 of 186 populated as of
   2026-08-28 (was 123; 14 backfilled by matching `#ent-<nickname>`).

⚠️ **The Homie bot's stage-change post is HubSpot's own data echoed into Slack** (it prints
Tier, Stage, AE, LSM(s), Target Launch, Lender Type). Never parse those fields back into
HubSpot — that is a feedback loop that can reinstate a value a human has since corrected.

Partner-launch links are posted by marketing in a fixed shape. **“Marketing Hub” is the
resource center**; the property name differs by object:

| Marketing artifact | Company property | Partnership property | Expected host |
|---|---|---|---|
| `Marketing Hub: <url>` | `resource_center` | `marketing_hub` | `poweredby.homelight.com` |
| `Landing page: <url>` | `landing_page` | `landing_page` | `lenders.homelight.com` |
| `Submission link: <url>` | — | `lead_submission_link` | `equity-app.homelight.com` |

Canonical link registry: [Partners Marketing Hub and Resource Links](https://docs.google.com/spreadsheets/d/1DCzc8HZ_v8R8urrD5QLOsTjPm7V3qlk0xQ1XdIGaRek/edit?gid=0#gid=0). The 2026-09-01 immutable local snapshot is
[`source-snapshot.csv`](../exports/2026-09-01-partner-resource-links-sync/applied/source-snapshot.csv).
Two PDF-only short links were intentionally not written because neither object has a verified
distinct PDF/materials field. See [[projects/2026-09-01-partner-resource-links-hubspot-sync]] for
the executed sync and [[projects/2026-08-28-partner-channel-sync]] for the Slack-channel consumer.

## Major Partners

### UWM (United Wholesale Mortgage) — ID 32130100787
- **Status:** Active, "Pot of Gold" for 2026
- **Type:** Wholesale lender
- **Generating:** ~20 apps/day
- **Distribution:** Each LSM getting 70-80 UWM AEs assigned
- **Key rule:** AE relationship trumps existing LO assignments
- **Commission:** 50/50 split for first 3 conflicted deals, then goes to AE's assigned LSM
- **Weekly Thursday trainings** scheduled
- **Leadership:** Grana and Ken traveling to Detroit week of April 6
- **Large event:** Late April targeting ~400 UWM sales org members
- **Top partner AE teams:** NEXA, Merit/Barret
- **Daily updates** go to Drew Uher and Sumant Sridharan
- **UWM attribution agent** monitors emails/SMS/calls/Slack for referral signals
- **Attribution fix (2026-04-02):** Agent now detects uwm.com links in email bodies

### TLS (The Loan Store)
- **Type:** Wholesale lender
- **Rev share (effective March 2026):**
  - Under 75 closings/mo: $500/closing
  - 75-125 closings/mo: $600/closing
  - 125+ closings/mo: $750/closing
- Comp goes directly to TLS (not individual AEs) — TLS handles internal AE comp
- 2025: 701 total transactions, only 84 funded with TLS
- 2026 YTD: only 10 funded with TLS
- States where HL isn't licensed (VA, MA, MO, VT, HI, AK, NY) → TLS funding
- 90% CLTV option launch expected to 2.5-3x business
- **Sub-TLS DBA auto-waive (stated, DM D0448K7BX1D Joel Shurtleff↔Tulli, 2026-03-04, not
  re-verified against code this pass):** Sales App's agreement-generation flow auto-waives the
  HLCS closing fee for LOs under confirmed sub-TLS DBAs — Joel: *"we are only doing so for a few
  at the moment,"* i.e. an allowlist, not every sub-TLS DBA. There is no query/table Joel or
  Tulli could point to at the time for "which DBAs has Sarah Jaka added under the TLS umbrella" —
  this mapping lived only in Sarah's head/manual tracking as of March 2026; unconfirmed whether a
  `partner_nickname`/DBA table now covers it (see the Partnership-object field work logged
  elsewhere in this file, 2026-08-26 onward).

### Orchard
- **Type:** Partner company
- **Rev share:** $400/closing for 2026 (simplified from tiered 2025 model)
- **YTD closed through March:** $10,400
- **Payment cadence:** Monthly, sent on Thursday following first Monday of month
- **Special rule:** Nick Santulli only owns contacts at Orchard companies

### NVR (National homebuilder)
- **Status:** Contract nearing final turn (1-2 weeks for legal edits)
- **No cancellation fee** (would delay, not needed)
- Will subsidize program selectively (rate credits)
- Pricing framing: "NVR special price - 1% off"

### DR Horton
- Expanding to **32 communities in mid-Atlantic**
- Part of explosive builder channel growth (2-3x more leads in one month than since Nov 2023)

### Other Notable Partners
- **Pulte** — Deal under contract
- **APM (James)** — Wants to join HELOC pilot, pending internal sign-off
- **Edge Home Finance, C2 Financial, Barrett Financial** — Top UWM brokers by conventional purchase volume

## Partner Matching Priority (as of 2026-04-09)

The deal's `partner_slug` is the **authoritative signal** for partner matching. It runs as step 1 in the deterministic waterfall, ahead of enterprise fields, domain maps, and all other signals. This is critical for broker LOs (e.g., NEXA, E Mortgage Capital) who route loans through multiple wholesale lenders — domain/company-based matching is ambiguous for brokers, but the slug from intake is explicit.

If the AI agent fallback picks a partner whose slug doesn't match the deal's slug, the handler overrides with the slug-matched partner.

**Known issue:** HubSpot sometimes returns a cardinality error when swapping Main Partner associations (removing old + adding new too quickly). The agent slug override catches this as a safety net.

## Enterprise Keywords (from Data Bridge codebase)
Domain keywords mapped to partnership IDs for deterministic matching (checked at step 3, after slug and email comms):
- `uwm` → 32130100787 (UWM)
- `united wholesale` → 32130100787
- `tls` / `tlstpo` / `the loan store` → TLS partner ID
- Additional wholesale email domains: `uwm.com`, `tlstpo.com`

## Association Labels
| Label | Type ID | Direction |
|-------|---------|-----------|
| Lender Partner (Main) | 129 | Contact → Partnership |
| Lender Partner (Sub) | 131 | Contact → Partnership |
| Main Partner (Deal) | 132 | Deal → Partnership |
| Secondary Partner (Deal) | 140 | Deal → Partnership |
| Main (Partner→Partner) | 136 | Sub → Parent |

## Top Companies by BBYS Volume (All-Time)

| Rank | Company | Apps | IRUCs | DRXs | Pod |
|------|---------|------|-------|------|-----|
| 1 | Orchard | 4,175 | 520 | 410 | Pod 3 |
| 2 | Fairway Independent Mortgage | 1,593 | 431 | 297 | Pod 1 |
| 3 | CrossCountry Mortgage (CCM) | 1,143 | 305 | 219 | Pod 1 |
| 4 | Lennar | 1,029 | 244 | 204 | Builder |
| 5 | NEXA Mortgage | 509 | 125 | 96 | Pod 3 |
| 6 | Envoy Mortgage | 456 | 100 | 69 | Pod 2 |
| 7 | Edge Home Finance | 350 | 87 | 64 | Pod 2 |
| 8 | Lower | 300 | 67 | 67 | Pod 1 |
| 9 | West Capital Lending | 259 | 40 | 33 | Pod 2 |
| 10 | Barrett Financial Group | 223 | 61 | 50 | Pod 3 |

Top 4 companies account for >60% of all BBYS volume.

## Wholesale Lender Distribution (Closed Deals)

- **TLS (The Loan Store):** ~60% of closed wholesale volume. Many "Main Partner" companies (NEXA, Edge, Barrett) route loans through TLS.
- **UWM:** ~15% and growing fast. ~20 apps/day. 700+ AEs being segmented.
- TLS and UWM together cover ~75% of closed deal wholesale lending.

## Enterprise Partner Tiers

### Tier 1 (QBR-worthy, 10 partners)
CCM, NEXA, Mutual of Omaha, NFM Lending, CMG Financial, APM, Envoy, Edge Home Finance, Arbor Financial, PRMI

### Tier 2 (6 partners)
Lower.com, Loan Factory, Luminate, Atlantic Coast Mortgage, Go Rascal, Xpert Home Lending

### Builders (Separate Track)
Lennar (1,029 apps), K. Hovnanian (64 apps, explosive YTD growth), DR Horton (32 mid-Atlantic communities), Pulte (under contract), NVR (contract near final)

> **Builder channel = homebuilders and their in-house lending arms only** (the five above, plus Home American). A lender is NOT builder channel just because it is a large enterprise partner.
>
> **NFM Lending is RETAIL, not builder** (confirmed by Tulli, 2026-07-29). It is a Tier 1 enterprise partner / Main Partner (`nfm-lending`, HubSpot `31723549092`) and appears on the retail-lender denylist in the broker IR-closed report. Do not add `nfm` to `KNOWN_BUILDER_COMPANIES`, `BUILDER_COMPANY_TERMS`, `BUILDER_EMAIL_DOMAINS`, or `BUILDER_PARTNER_SLUGS` in Data-Bridge.

### 🔑 Khovnanian — 2bps comp carve-out for Tejas + TJ (added 2026-08-29, per decisions/2026-05-12-khovnanian-carveout-tejas-tj.md)

Tejas Narkhede and TJ Sims **each individually** earn **2bps of comp-able value** on every
deal where the partner is Khovnanian (matched on `main_partner__ai_` or `partner_slug`,
case-insensitive, tolerant of "K. Hovnanian" stylization). Applies to deals with
`irClosedDate` in calendar **2026** that are in a paid stage (IR Closed or DR Closed).
Comp-app role `LSM-Khovnanian` — a separate line item, **not split** between the two reps,
paid regardless of who the deal's actual LSM/AE is, and structurally isolated from quota math,
pod-share, Jake's Head-LR pool, and Tiffany's Builder-team comp. Same pattern as Brandi's
`LRM-Override` 1.5bps.

⚠️ **2026-only, code-enforced by year guard** (`KHOVNANIAN_CARVEOUT_YEAR`). Sunsets implicitly
when 2027 close-dates start landing unless extended — decision review flagged for
**2026-12-31**. Relevant heading into the Sep-1 reorg: confirm whether this carve-out survives
the Wholesale/Retail split or needs to be re-pointed once Tejas's book (T1, still undecided
per [[risk-register]] §1 row 3) is resolved.

### Sekisui House U.S. — builder-lender family (added 2026-08-26)

Sekisui House U.S. (SHUS) is a **holding company**, not a lender. It owns Richmond
American Homes / M.D.C. (captive lender **HomeAmerican Mortgage Corporation**), Chesmar
Homes (captive lender **CLM Mortgage, Inc.**), Woodside Homes, and Hubble Homes. Since
integration, LOs across all of these email from the shared `@shus.com` parent domain.

| Entity | Company ID | Partnership | Nickname |
|---|---|---|---|
| Sekisui House U.S. (corporate) | `54191787071` (sekisuihouse.com) | — | — |
| Sekisui House U.S. (shus.com) — **triage bucket** | `48484605290` | — | — |
| HomeAmerican Mortgage Corporation | `36903707683` | `31760751665` | `homeamerican` |
| Richmond American Homes | `36990835045` (mdch.com) | — | — |
| CLM Mortgage, Inc. | `21585703704` (clmmortgage.com) | `60892009516` (created 2026-08-26) | `clm-mortgage` |

**Native parent/child hierarchy is live (built 2026-08-26).** Sekisui House U.S.
`54191787071` is the HubSpot **parent company** of all seven records below, using the
native `Child Company` association (type `13`) so it renders in the standard parent/child
UI rather than a custom field:

| Child | ID | Domain |
|---|---|---|
| HomeAmerican Mortgage Corporation | `36903707683` | homeamericanmortgage.com |
| CLM Mortgage, Inc. | `21585703704` | clmmortgage.com |
| Richmond American Homes | `36990835045` | mdch.com |
| Chesmar Homes | `41739014036` | chesmar.com |
| Woodside Homes | `2738048914` | woodsidehomes.com |
| Hubble Homes | `16983165977` | hubblehomes.com |
| Sekisui House U.S. (shus.com) | `48484605290` | shus.com |

⚠️ **Do not merge the shus.com record into Sekisui House U.S.** Because `shus.com` is a
parent domain, HubSpot's domain-keyed auto-association keeps routing every subsidiary's
LOs to one record. Merging collapses HomeAmerican vs. CLM reporting permanently. Keep
`48484605290` as the shus.com catch-all and re-point LOs to the correct subsidiary as
their employer becomes known.

⚠️ **The hierarchy is structural only — it does not cascade contact classification.** See
[[hubspot]] § v4 association mechanics. Each subsidiary needs its own `lender = "Builder"`.

> **Duplicate:** Woodside Homes exists twice — `2738048914` (woodsidehomes.com) and
> `20405798495` (woodside-homes.com), both 0 apps. Only the first is linked to the parent.
> Not merged (irreversible) — needs a decision.

> The CLM Mortgage partnership `60892009516` still needs `partnership_stage` and `tier`
> set by the partner team — created with name/slug/nickname/hierarchy only.

## Pod Distribution

| Pod | Companies | All-Time Apps | YTD Apps |
|-----|-----------|---------------|----------|
| Pod 1 (Matt, Tierney, Richie) | 28 | 3,718 | 495 |
| Pod 2 (Brian, TJ, Tejas) | 99 | 4,370 | 533 |
| Pod 3 (Marisa, Kara) | 59 | 6,625 | 906 |
| Builder (Tiffany) | 3 | 1,108 | 153 |

Pod 3 highest volume (driven by Orchard) despite only 2 LSMs. Pod 2 manages the most companies (99).

## QBR Program

Covers top 30 enterprise partners with branded decks. Led by John Labrada and Kim Tanner. In transition after Annie's departure. Deck structure: units and volume focus, ideal borrower profile, BBYS overview (removed market data per Labrada's feedback).

## Concentration Risk

- Orchard alone: ~30% of all BBYS applications
- Top 4 companies: >60% of volume
- TLS as wholesale lender: ~60% of closed deals
- 96.5% of closed deals are lender-sourced (vs 3.5% agent-sourced)

## Partner Slugs (canonical list, 2026-04-22)

> Canonical `partner_slug` values for deterministic deal matching. Source: `projects/2026-04-22-partner-slugs.csv`. Preserves established slugs (uwm, tls, lennar, america-first); rest are slugified from partner name.

| Partner | Slug |
|---------|------|
| Atlantic Bay Mortgage | atlantic-bay-mortgage |
| Key Mortgage Services Inc | key-mortgage-services |
| Absolute Home Mortgage Corp | absolute-home-mortgage |
| Acre Mortgage | acre-mortgage |
| American Financial Network | american-financial-network |
| American Pacific Mortgage | american-pacific-mortgage |
| Amres | amres |
| Arbor Financial Group | arbor-financial-group |
| Artisan Home Loans | artisan-home-loans |
| Aspire Mortgage Group | aspire-mortgage-group |
| Bay Equity Home Loans | bay-equity-home-loans | ⚠️ Actual HubSpot slug is `bay-equity` (partnership ID `31759447574`). Canonical table says `bay-equity-home-loans` but the live record uses the shorter slug. See [[2026-05-18-rocket-mortgage-bay-equity-secondary-partner]]. |
| Benchmark | benchmark |
| C2 Financial | c2-financial |
| Canopy Mortgage | canopy-mortgage |
| Cardinal Financial | cardinal-financial |
| Change Home Mortgage | change-home-mortgage |
| Churchill Mortgage | churchill-mortgage |
| City First Mortgage Services | city-first-mortgage-services |
| Coast2Coast Mortgage | coast2coast-mortgage |
| Cornerstone First Mortgage | cornerstone-first-mortgage |
| CrossCountry Mortgage | crosscountry-mortgage |
| DR Horton | dr-horton |
| Direct Mortgage Loans | direct-mortgage-loans |
| E5 Mortgage | e5-mortgage |
| Ease Mortgage | ease-mortgage |
| Envoy Mortgage | envoy-mortgage |
| Fairway Mortgage | fairway-mortgage |
| Fidelity Direct Mortgage | fidelity-direct-mortgage |
| First Coast Mortgage Funding | first-coast-mortgage-funding |
| First Continental Mortgage | first-continental-mortgage |
| First Option Mortgage | first-option-mortgage |
| GO Mortgage | go-mortgage |
| Gershman Mortgage | gershman-mortgage |
| GoRascal | gorascal |
| Gold Star Mortgage | gold-star-mortgage |
| Home American | home-american |
| Home Financing Direct | home-financing-direct |
| HomeSimply | homesimply |
| Homeowners Financial Group | homeowners-financial-group |
| K Hovnanian Homes | k-hovnanian-homes |
| Keller Home Loans | keller-home-loans |
| Lakeview | lakeview |
| Lions Capital Mortgage | lions-capital-mortgage |
| Loan Factory | loan-factory |
| LoanHaus | loanhaus |
| Lower.com | lower |
| Luminate Home Loans | luminate-home-loans |
| Mortgage Equity Partners | mortgage-equity-partners |
| Mortgage Investors Group | mortgage-investors-group |
| Mutual of Omaha Mortgage | mutual-of-omaha-mortgage |
| My City Home Loans | my-city-home-loans |
| NFM Lending | nfm-lending |
| NVR | nvr |
| Neo Home Loans | neo-home-loans |
| New Venture Escrow | new-venture-escrow |
| No Partner | no-partner |
| Northwest Funding Group | northwest-funding-group |
| Orchard | orchard |
| PACOR | pacor |
| People's Mortgage Company | peoples-mortgage-company |
| Primary Residential Mortgage Inc (PRMI) | prmi |
| Pulte Mortgage | pulte-mortgage |
| RWM Home Loans | rwm-home-loans |
| Resource Financial Services | resource-financial-services |
| Ross Mortgage | ross-mortgage |
| Society Mortgage | society-mortgage |
| Success Lending | success-lending |
| Sunward Federal Credit Union | sunward-federal-credit-union |
| Texana | texana |
| The Mortgage Calculator | the-mortgage-calculator |
| The Mortgage Link | the-mortgage-link |
| Thrive Lending | thrive-lending |
| Thrive Mortgage | thrive-mortgage |
| US Mortgage | us-mortgage |
| United Wholesale Mortgage | uwm |
| VIP Mortgage | vip-mortgage |
| America First | america-first |
| Lennar | lennar |
| The Loan Store | tls |

## 2026-08-29 — Slack six-month sweep (Mar 1 – Aug 29, 2026)

> Source: Slack, `#ent-*` partner channels + targeted `slack_search_public_and_private`,
> 2026-03-01 to 2026-08-29, read-only. Full per-partner detail, quotes, and open threads in
> [[projects/2026-08-29-slack-partners-6mo]]. This section is the durable summary; that doc is
> the time-bound working notes.

### 🔴 Bay Equity — not a partner anymore, resolve any lingering references

**Stated** (Joel Shurtleff, `#ent-bayequity`, 2025-07-08; Tejas Narkhede, `#lab-rats-lp-sales`,
2026-04-22): Rocket Mortgage acquired Redfin, which owned Bay Equity Home Loans. Rocket and
UWM are treated as conflicting relationships internally, so **HomeLight will not re-partner
with (now-Rocket) Bay Equity LOs** even though several were prolific BBYS users (Dean Hayes
cited repeatedly as "10x+ his individual production through referrals"). Operational cleanup —
migrating LO records from `@bayequityhomeloans.com` to `@rocket.com` emails in HubSpot/Sales
App, one-off exception requests to keep working top producers — ran continuously from mid-2025
through at least 2026-04. The `#ent-bayequity` and `bay-equity-*` deal channels are silent for
the entire Mar–Aug 2026 window; all activity predates it. **The Top Companies by BBYS Volume
table above still lists no Bay Equity entry, which is consistent** — but any live automation,
LSM tracker row, or `partnership_stage` still marked non-`Failed` for Bay Equity is stale and
should be corrected to reflect the acquisition.

### TLS (The Loan Store) → rebranding to Averra Financial, Inc.

**Stated** (Brenda Philips, Director of Marketing/Comms at TLS, relayed by John Labrada,
`#ent-the-loan-store`, 2026-08-18): legal name change effective **2026-09-30**, public go-live
**2026-10-01**. New logo attached (Averra Financial Inc, RGB). TLS explicitly asked that this
not take effect on HomeLight's side before go-live to avoid early exposure.

- A **new dedicated channel `#ent-the-loan-store` (`C0BR0CF0R4N`) was created 2026-08-18**
  specifically to run this transition — TLS previously had no aggregate `#ent-*` channel (only
  per-deal `#tls-*` channels), consistent with [[bbys-pricing-engine]]'s note that TLS isn't in
  the formal partner config lists either.
- Decision (Labrada + Soleil Ocampo, same thread): **retain the existing submission URL**
  (`equity-app.homelight.com/application-flow?partner=tls`) — it's baked into other marketing
  materials and sales comms — and **only swap the logo**, not the URL slug.
- ⚠️ TLS's marketing team had *previously* asked BBYS materials **not** carry TLS branding at
  all (no logo, no brand colors) — the rebrand request initially read as a reversal of that
  policy but turned out to just be "use our new logo where a logo already exists." Confirm
  before assuming full Averra branding rolls out broadly.
- This is a **before Oct 1, 2026** item — the vault's existing TLS rev-share terms (tiered
  $500/$600/$750 per closing) are not stated to change, only the entity name/logo.

> 🔑 **Technical migration scope confirmed narrow (2026-08-29 check).** Nothing found in
> `#ent-the-loan-store` or elsewhere touches `partner_slug` (`tls` stays `tls`), pricing
> templates, intake-URL keyword configs, or deal-channel naming (`tls-*` channels are unaffected).
> The only planned change is swapping the logo on marketing materials that already carry one —
> the submission URL, partner slug, and backend keyword matching are explicitly staying as-is per
> Labrada's decision. **(inferred)** if a rename-driven backend change is needed later (e.g. a
> `partner_nickname` or company-name update in HubSpot), it hasn't been discussed in Slack as of
> this pass — don't assume engineering has a ticket for it.

### 🔴 UWM — compliance-driven de-branding of the co-marketed landing page (2026-08-25)

**Stated** (John Labrada, `#ent-uwm`, 2026-08-25): *"We got into a little bit of trouble with
UWM recently. Can we please remove a few things from their landing page asap. We would need
the UWM Logo removed, the NMLS number at bottom removed and UWMs name removed from URL."*

Same-day remediation (11-message thread, Soleil Ocampo executing):
- `lenders.homelight.com/uwm` → `lenders.homelight.com/ent-partner` (logo, NMLS number, and
  "uwm" all stripped from the page and the URL).
- The **submission link was explicitly left alone** — `equity.homelight.com/bbys/new/uwm` —
  Labrada: *"Let's keep that for now please."*
- Also stripped: "book a demo" form's email requirement, and any UWM mention in the page's
  link-preview text/meta description.

> 🔑 **Root cause found 2026-08-29** (DM, Labrada→Tulli, `D04PPJA8GBF`, 2026-08-25 10:45 MDT):
> *"Their marketing/compliance never actually told us we could use their name, image, or nmls in
> anything we made lol."* Not a formal RESPA/MSA complaint from a regulator — **HomeLight never
> had UWM's written consent** to use their logo/NMLS/name on the co-branded page; UWM's side
> flagged it and asked for removal. **Current state: fully remediated same day.** By 17:50 MDT
> 2026-08-25: logo, NMLS number, "uwm" in URL (→ `lenders.homelight.com/ent-partner`), the "book a
> demo" form's email requirement, and the link-preview/meta-description text all stripped
> (11-message thread, Soleil Ocampo executing). The **submission link was deliberately left
> alone** (`equity.homelight.com/bbys/new/uwm`) — Labrada: *"Let's keep that for now please."* No
> further UWM-relationship friction found in Slack after 2026-08-25 through the sweep window's
> end (2026-08-29); the relationship otherwise reads as business-as-usual (Bullseye 90 incentive
> push, wholesale/retail split logistics). Treat this as closed, not an open compliance risk —
> but note HomeLight should get UWM's co-marketing use explicitly in writing before rebuilding any
> branded page, since the underlying gap (no signed consent) hasn't been fixed, just the symptom.

### SWBC — bank-channel architecture is live in code, named branding is not

SWBC hit **Executive Meeting** 2026-03-03 (Tier 2, target launch 3/30/26, AE Labrada, "Services
& Partnership Agreement" with a **$1,000 payment** noted in the agreement doc, no marketing hub
or landing page). By 2026-08-18, engineering status (Dave Spivey, `#homes-ops-and-efficiency-pod`)
was:

| Live now | Still to do |
|---|---|
| Bare client intake (`/bbys-intake`, `/bbys-for-builders`) routes to default partner attribution | **Named partner rollout — SWBC and others via sales-app + branded intake URLs** — not live |
| `homelightdirect@homelight.com` replaces `builders@` as the client-facing contact | End-to-end QA on real named-partner and bank flows |
| Slack alert fires if a client-channel lead lands with no source partner | New comms templates (client inspection reminder, builder DTI-drop handoff) |

🔑 **SWBC is architecturally a "Bank" partner, not a Lender partner** — it routes through the
`/bbys-intake` pre-lead flow to the **client-facing team (Nick Plamondon)**, bypassing the
normal LSM/LRM pod structure entirely (Tejas Narkhede, `#centralized-homes-epd-support`,
2026-08-06). Karly Trota's current partner taxonomy (`#we-be-building`, 2026-08-11):

| Category | Partners |
|---|---|
| Builder | Lennar, Pulte, KHov, DHI Mortgage, TriPointe, Sekisui House |
| Bank | SWBC |
| Non-Builder/Bank (client-facing) | Orchard, TOMO, Lakeview |

⚠️ **TOMO and Lakeview are client-facing partners with no prior write-up in this vault** —
only `Lakeview` appears, and only in the bare partner-slug canonical list. Worth a follow-up
pass.

A May 2026 SWBC builder-flow pilot note (Nick Plamondon, `#ent-swbc`): **no LO comp on the
builder flow** — "we do all the work for the program so that's the trade-off." This is a
different comp model from the standard wholesale/retail LO payment structures elsewhere in
this file and should not be assumed to generalize to other builder partners.

### Cardinal Financial — Tier 1 broker label, but still low traction

Channel topic (`#ent-cardinalfinancial`, set 2025-07-17): `LSM: Marisa, Launched: TBD, Tier 1
Broker`. As of 2026-06-18 (Marisa Drake): *"Trying to get some traction with Cardinal."*
2026-08-12 comms-journey planning explicitly **de-scoped Cardinal Financial** from the
bank/builder email-suppression fix ("de-scope cardinal financial for now"). Deal-level activity
in the window is real but thin (single-digit named LOs, no volume/escalation chatter) —
**(inferred)** Tier 1 here reflects the wholesale-broker classification, not actual production
volume; treat the QBR-tier label with skepticism for this partner specifically.

### Fairway, CrossCountry, NFM Lending — steady, no escalations found

- **Fairway**: heavy per-deal volume (10+ concurrent deal channels sampled), positive LO
  relationships (HLCS/Fairway team, Troy Gamble praised by name). ⚠️ Joel Shurtleff **left**
  `#ent-fairway` on 2026-08-26 — worth confirming who owns the aggregate relationship now if
  Joel was the primary contact.
  🔑 **`@fairwaymc.com` → `@home.com` domain migration: resolved via a manual merge tool, not
  an automatic routing fix (checked 2026-08-29).** The underlying incident — Fairway LOs
  switching from `@fairwaymc.com`/`@fairway.com` to `@home.com` broke LO-portal logins and lead
  routing (2026-05-21, since lead routing keys off LO email domain) — was addressed by Jose
  Herrera shipping a **"Link Loan Officers" admin tool** (`sales-app.homelight.com/admin/loan-
  officers`) on 2026-07-02, announced in `#lab-rats-lp-sales` and `#centralized-homes-epd-
  support`. It doesn't auto-migrate on domain change; ops/eng runs it per-LO (or batches it) to
  merge the old and new accounts and preserve transaction history. First batch merged 7
  named Fairway LOs (e.g. `aaron.vang@fairwaymc.com` → `aaron.vang@home.com`). ⚠️ **The tool is
  general-purpose, not Fairway-specific** — it's since been used for at least one other
  carrier-switch case (a Cross Country LO who'd previously worked at Fairway, 2026-07-30). No
  further reports of the original login/routing symptom found after 2026-07-02, but this is a
  standing manual process (ping `#lab-rats-lp-sales`), not a permanent fix to the underlying
  email-domain-keyed routing design — a future LO company switch will need the same manual
  merge again unless routing is re-keyed off something more stable than email domain.
- **CrossCountry (CCM)**: steady cadence of quarterly in-person/Zoom trainings (Kim Tanner +
  Tierney Izar + Matt Leddy), positive feedback loop. A **CCM-specific adoption campaign** is
  live in HubSpot lists: "CCM BBYS 101" targets LOs with **<3 applications in the last 6
  months**, distinct from the all-partner LO panel list (Elly Lee, `#hubspot-help`,
  2026-02-05) — i.e., CrossCountry gets bespoke re-engagement treatment other partners don't.
- **NFM Lending**: highest-signal healthy relationship in the sweep. June 2026: NFM's CEO
  planned to interview a top HomeLight-using LO in a company meeting; Kim Tanner solicited
  nominees (Nick Workman — 11 apps/4 IRUC; Jonathan Walk — 10 apps/2 IRUC; Dana Gounaris — 5
  apps/3 IRUC; Bradley Zerbe — 3 apps/2 IRUC). Heavy, continuous DTI Drop + Equity Boost/HELOC
  usage through August. No escalations found in the window.

### Lower — channel dormant for the entire sweep window

`#ent-lower` has **zero messages between 2025-08-28 and the present** (last real content:
an Apr 2025 payment-logic clarification — $1,000/DR-close for the first 25 closes/month,
$1,500 for the 26th+, paid directly to LOs and 1099'd; MortgagePass/Thrive consolidation into
Lower branding completed around Feb–Apr 2025). **(inferred)** either the relationship has gone
quiet at the ops level, or Lower activity is happening entirely in per-deal channels that
weren't individually swept — the aggregate channel gives no signal either way. Flagging as an
open question rather than a stated fact.

### 🔴 `#homes-partner-mismatch` — the mismatch bot has not posted since 2026-02-05

The "HomeLight Homie" bot posts `*Potential partner mismatch*` alerts (lead ID + AI-determined
partner vs. stored `partner_slug`) to this channel. The last message in the channel's entire
history is dated **2026-02-05** — three-plus weeks before this sweep's window even starts, and
nothing has posted since across the full 6 months. Either the underlying mismatch-detection job
was disabled/broken sometime in Feb 2026, or partner-slug mismatches genuinely stopped
occurring — the latter seems unlikely given the deterministic-matching caveats already
documented elsewhere in this file.

> 🔑 **Builder and origin found (2026-08-29):** Gaurav Hardikar built it as a personal side
> project, not an owned EPD initiative — Group DM `C08BK3QUPDY`, 2025-09-08, Tulli: *"By the way
> #homes-partner-mismatch is running where there's a likely mismatch/missing partner in
> SalesApp. I don't have the capacity to build out tagging logic lol."* Same day, Gaurav (DM
> `C09EY2QDM9N`): *"because of the partnership slug issue, I built a workflow this weekend that
> pulls from a variety of sources to determine the 'main' and if applicable 'secondary'
> partnerships — I just started piping mismatches into #homes-partner-mismatch."* **A weekend
> build with no formal ownership, monitoring cadence, or alerting-on-alerting.** Its death was
> **not proactively noticed** — the only trace of anyone checking it after launch is Jake Vogel
> five weeks later (DM `D051H6JJTK3`, 2025-10-21): *"I haven't been keeping up with it the last
> few days"* — i.e. it was already going unwatched well before its Feb 2026 stop. **(inferred)**
> nobody was reading the channel regularly enough to notice when it went silent; this reads as
> an unowned tool quietly dying, not a known-and-accepted deprecation. Still needs an actual
> engineering check of whatever cron/job produced these posts to confirm disabled vs. broken.

### Variable-pricing partner list: contradicts [[bbys-pricing-engine]]'s seed-catalog claim

[[bbys-pricing-engine]] states the seeded `pricing_templates` catalog already includes 5 VP
partners (`apm, envoy, arbor-financial, nfm-lending, 21st-century`), with the docstring
lagging at 4. **Live Slack evidence says otherwise as of this sweep:**

- 2026-08-04 (Taylor Wong, `#homes-ops-and-efficiency-pod`), pasting **prod** Flagsmith
  `variable_pricing`: `["apm", "envoy", "arbor-financial", "nfm-lending"]` — **4 partners, no
  21st-century.**
- 2026-08-13 (Karly Trota, `#centralized-homes-epd-support` and the `21st-century-*` deal
  channel): explicitly **requests** Century 21 be added to the VP list; Carter Marks confirms
  it needs an engineering change to add "the team to the list."

🔴 **This means 21st-century was not actually a live VP partner in prod until mid-August 2026
at the earliest** — contradicting the framing that the seed catalog already covers 5 partners.
Either the seed catalog and the Flagsmith flag drifted out of sync (consistent with
[[bbys-pricing-engine]]'s own warning that "changing the Flagsmith flag does not change pricing
eligibility... someone must re-seed"), or 21st-century's addition to the seed happened after
2026-08-13 and before this file's `c14079c76c` baseline. Needs reconciling against the current
Flagsmith flag and `pricing_templates.eligible_partner_slugs` directly.

🔴 **Separately, a production data-integrity bug hit VP leads across multiple partners**
(Bruno Gonzalez + Taylor Wong, `#envoy-irx-barber-102-angus-ct-va-express` and
`#homes-ops-and-efficiency-pod`, 2026-07-31): a backfill (`hapi#19718` / `sc-190081`, merged
2026-07-29, `LeadDataService::Bbys::BackfillPricingEngine#stamp_fee!`) re-resolved
`pricing_model` for existing leads against a template catalog that **only had a VP template for
`lennar`** — not the Flagsmith-configured VP partners — so it wrote `flat_fee` onto ~50
VP-eligible leads (tls: 15, apm: 13, nfm-lending: 5, envoy: 4, uwm: 3). It also passed
`force: true`, bypassing the post-signature `agreement_signed_lock?` guard, re-resolving
pricing on at least one already-signed lead. Some leads self-corrected on the next save; **3
leads were confirmed still carrying a wrongly-regenerated flat-fee agreement** as of 2026-07-31.
Manual fixes (Cheryl Funk flipping `pricing_model` back by hand) don't hold — the engine
re-resolves to `standard_bbys` on the next run.

> 🔑 **RESOLVED same day (2026-07-31), confirmed 2026-08-29.** Taylor Wong ran a corrective
> backfill restoring `pricing_model: variable` on all ~50 affected leads within hours of Bruno's
> report — his own words in-thread: *"backfill is complete. i ran this silently so we didn't
> generate a bunch of slack notifications"* (~15:47 MDT, same day). Two follow-ups were flagged
> but not confirmed shipped as of this pass: (1) stop the backfill code path from touching
> `pricing_model` until the pricing-engine feature flag is fully on, (2) add pricing templates for
> the partners still missing one. Do not assume those two are done without checking
> `LeadDataService::Bbys::BackfillPricingEngine` and `pricing_templates` directly.

### Channel-ID reference (major partners swept)

| Partner | Channel | ID | Type | Created |
|---|---|---|---|---|
| UWM | `#ent-uwm` | `C09AK1DQU6T` | private | 2025-08-18 |
| TLS / Averra | `#ent-the-loan-store` | `C0BR0CF0R4N` | private | 2026-08-18 |
| Fairway | `#ent-fairway` | `C06JJ5DFBNH` | private | 2024-02-13 |
| NFM Lending | `#enterprise-nfm` | `C08FPE3DNP2` | private | 2025-02-25 |
| CrossCountry | `#ent-crosscountry` | `C060G8TLWP9` | public | 2023-10-12 |
| Lower | `#ent-lower` | `C07TFG6PJDS` | private | 2024-10-24 |
| Bay Equity | `#ent-bayequity` | `C07UXQ0L679` | private | 2024-11-07 |
| SWBC | `#ent-swbc` | `C0AHWB42D0F` | public | 2026-03-03 |
| Cardinal Financial | `#ent-cardinalfinancial` | `C07GN162DQU` | private | 2024-08-12 |
| Blend | *(none found)* | — | — | — |

⚠️ **Blend has no `#ent-*` channel** — searches for "blend" and "blend integration" turned up
only unrelated `#development-*`/`#billboard-blend` (an internal comms tool announcement
channel, nothing to do with a lender partner) results. **(inferred)** Blend is either not an
active enterprise lender partnership, or its channel uses a naming convention not yet
discovered. Do not assume Blend is a live partner without confirming in HubSpot.

## 2026-08-29 — Nous archive tail (meetings_archive/threads/topics collections, read-only)

From a loop-2 mining pass into two Nous collections a prior pass found but did not open
([[projects/2026-08-29-nous-gamma-mining]]). **Access limit:** `meetings_archive`, `threads`,
and `topics` are search-indexed in Nous but return `permission_denied` on every direct fetch —
everything below is reconstructed from search-result snippets only, not full records. The
`topics` collection's snapshot marker reads `status: 05-14` (2026-05-14) — treat as a **mid-May
2026 snapshot**, not verified current. Full methodology: [[projects/2026-08-29-nous-archive-tail]].

### Zavvie — a proposed warehouse-funding relationship, not previously in this vault

**Not found anywhere else in the vault before this pass.** Zavvie is a separate buy-before-you-sell
fintech (cash-offer model: 5% customer down payment / 95% Zavvie-funded), stated scale "600+
reserve deals (1% default) + 100+ signature loans (0% default)." Per the Nous `topics` narrative
(`zavvie-partnership`, snapshot 2026-05-14): **as of 2026-05-12, Zavvie hit a warehouse-line
crisis requiring emergency resolution by May 15, and HomeLight was the proposed solution** —
i.e. HomeLight considered becoming Zavvie's warehouse-funding backstop. Stated economics: Zavvie
carries warehouse/compliance/loss risk; HomeLight would receive **0.9% of a 1.9%** fee, plus
exclusive funding and a title-business push. Ashwin summarized the economics; Marc Kaplan +
Ashwin were scoping; an ops-coordination meeting was set for 2026-05-13. `(stated, single
source, not code-verified)` **No later vault source confirms whether this closed, and in what
form.** Multiple `threads` items reference "Zavvie deal tape," "Zavvie advance rate
confirmation," "Zavvie first deal review," and "MERS registration and documentation structure"
all still `state: open` in the same snapshot.

### Luminate — TPO HELOC warehouse-line partner (mechanics, not previously documented)

The vault's existing Luminate mentions ([[risk-register]] rows 11–12) cover it only as (a) a
retail comp-plan counterparty and (b) a name floated in an Aug-28 meeting as a possible
Chase-line offset. Nous's `topics` narrative (`luminate-partnership`, snapshot 2026-05-14) adds
the underlying mechanics: **strategic partnership with Luminate (contact: Jerry Kaplan) for TPO
(third-party-originator) HELOC distribution through Luminate's credit-union consortium** —
potential 50-state coverage. **Master warehouse-line model: a $5M line with a $1M minimum
restricted deposit.** ⚠️ Nous's own `people` record for Jerry Kaplan is marked `stale: true`
inside Nous itself — even the source system no longer treats this contact as current. This
predates and may explain why the Aug-28 "Weekly Sync" transcript ([[risk-register]] row 12)
treats "getting the HELOC line with Luminate set up" as a known, discussable option rather than
a cold introduction — the relationship groundwork was laid as early as mid-May.

### Chase/JPM — searched, not found

Targeted search for the pre-pause history behind [[risk-register]] row 12 (the Aug-28-stated
Chase/JPM $150M warehouse-line pause) turned up **nothing** in `meetings_archive`, `threads`,
`topics`, or the pre-May-12 tail of "Meetings - Tulli" — no meeting, thread, or topic mentions
Chase or JPMorgan by name anywhere in what this pass could search. Either the relationship
predates this Nous instance's coverage (earliest record found anywhere: 2026-04-20), was tracked
outside Nous entirely, or uses a name/alias not searched. Flagged as a negative result, not
evidence the relationship didn't exist earlier — do not read this as "the line has no history,"
only as "this instrument doesn't have it."

## Related Notes
- [[hubspot]]
- [[bbys-overview]]
- [[data-bridge]]
- [[top-producers]]
- [[bbys-pricing-engine]]
- [[projects/2026-08-29-slack-partners-6mo]]
- [[projects/2026-08-29-nous-archive-tail]]
