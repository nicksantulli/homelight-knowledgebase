---
last_updated: 2026-08-29
status: current
source (updated 2026-08-29 — BIPs/IRAX corrected from Google Drive comp plans; AE collision, comp-able value, TE added): vault synthesis — full sweep of context/*.md + projects/2026-08-28/29-*.md, cross-referenced against grep for ALL-CAPS tokens
scope: Master acronym and vocabulary index for the vault. Resolves collisions, defines orphan acronyms, fixes one wrong entry.
source: HomeLight-Vault/context/glossary.md
imported: 2026-09-29
---

# Glossary — acronyms, roles, and collisions

**Read this before quoting any acronym in the vault.** Six live terms mean two-or-more
different things depending on which system, doc, or era you're reading. Getting one wrong in
front of a partner or in an exec deck is the failure mode this file exists to prevent.

Seeded from the [[bbys-overview]] §Terminology table (now superseded — that table stays in
place but this file is canonical for collisions). Extended by grepping every `context/*.md`
file for ALL-CAPS tokens and tracing each to its first defining use.

**Epistemic key:** 🔴 = corrected error · ⚠️ = live collision, disambiguate before using ·
`(inferred)` = deduced from code/usage, not stated outright · `UNRESOLVED` = flagged, not solved.

---

## 🔴 Correction: SRM (fixing [[bbys-overview]])

**[[bbys-overview]] §Terminology has said, since its creation, that SRM means "Suppression
list — contacts not to be reassigned." This is wrong and is corrected here 2026-08-29.**

- **Correct:** SRM = **Strategic Relationship Manager**, a real role. SRMs own AGENT
  relationships and carry a small handful of LO contacts. They appear in the **`srm`** owner
  field on deals and occasionally in **`lsm_collaborator`** when subbing as LSM on partner
  deals. Onboarded into Data Bridge 2026-04-25 with `team_role='SRM'`. Source: [[team]]
  §"Strategic Relationship Managers (SRMs)".
- **Where the wrong reading came from:** there is a real HubSpot list named **"SRM Retained
  LOs - Suppression Sheet"** — contacts an SRM has claimed that should not be auto-reassigned
  by the LSM/LRM waterfall. A nightly cron (`srm-suppression-audit`, PR #620) audits it. The
  list is *named after the role*; "SRM Suppression" is not what the acronym SRM means.
  Source: [[tools-and-stack]] (2026-07-28 changelog entry, `srm-suppression-audit`).
- **Blast radius:** every `srm` field, every "SRM" pivot on the LSM/LRM Reports page, and
  every mention of "SRM Suppression" in Slack read correctly once you know these are the same
  three letters describing a role and a list named for it — not two meanings of the acronym.
- **Action taken:** [[bbys-overview]] §Terminology's SRM row has been rewritten in place with
  a pointer to this correction (see that file, 2026-08-29). The original wrong claim is not
  silently deleted — this section is the permanent record of what was wrong and why.

---

## ⚠️ Collision section — the six that will burn you

| Term | Meaning 1 | Meaning 2 | Meaning 3 | Disambiguate by |
|---|---|---|---|---|
| **SRM** | Strategic Relationship Manager (role) — **correct** | "SRM Suppression" list, named after the role — not a second meaning of the acronym, but reads like one | — | See correction above. If in doubt, it's the role. |
| **LSM** | **Lender Success Manager** — the canonical roster/`job_title` value ([[team]] §"Email Signature Directory", 9-person roster) | *Lender Sales Manager* — used in [[hubspot-workflows]] §"TAM <> LSM Notifications", [[bbys-routing-and-ownership]] `TransactionTeam::EMPLOYEE_ROLES` label, [[hubspot.md]] association-type 198 label | *Lending Sales Manager* — [[bbys-overview]] §Terminology (superseded by this file) | Use "Lender Success Manager" going forward. All three are observed in the vault; none is a typo, they're just inconsistent across docs written at different times. **Irrelevant after Sep 1 anyway** — the Sep-1 reorg renames the *display* title LSM → **AE** (`team_role` stays `'LSM'` in the DB — display-layer rename only, per [[team]] banner, decision D2). |
| **pod** | **Kubernetes replica** — e.g. `homes-sales-hub-api` replica count. Source: [[2026-08-28-hub-cutover-master-sequence]] §"Pod is now ambiguous" | **Sales team** — Pod 1/Pod 2/Pod 3/Builder, the LSM roster grouping in [[team]], becoming Retail/Wholesale/Builder-Client at Sep-1 cutover | — | Say "k8s pod" or "sales pod" explicitly in any shared channel. The hub-cutover doc already flags this — use it as the model for how to write a collision note. |
| **CA** | **Client Advisor** — `client_advisor` role, `TransactionTeam::EMPLOYEE_ROLES` slug, dashboard `BbysCaDashboardPage`. Source: [[bbys-routing-and-ownership]] §"CA is ambiguous" | **Contract Advisor** — `contract_advisor` role, a *separate, real* HubSpot/Sales-App role. Faaz's launch post called the same dashboard the "Contract Advisor Dashboard" while the code says Client Advisor — both terms are correct for **different roles**, not two names for one role. Reorg renames Contract Advisor → **Closing Manager** ([[meeting-cadences]] §43). | **Client Advocate** — informal Slack usage for the "CA Notes" handoff-block author (usually the LSM), e.g. "CA / CA Notes" in [[bbys-deal-channel-vocabulary]] | Ask which role before answering. The round-robin enum (`lrm/ca/tp/brm`) uses a single `ca` value that **does not disambiguate** — this is a real system gap, not just a documentation one. |
| **AE** | **Enterprise AE** — the senior/oversight title, only 3 people: Kim Tanner (`589265219`), John Labrada (`54109913`), Nick Plamondon (`331990682`). Field: `lender_sales_account_executive`. | **Post-reorg LSM rename** — Sep-1 display-layer rename of every working LSM's title to "Account Executive" (`lender_sales_manager` field, unchanged). | **Partner tracker "AE" column** — in partner-side Google Sheets, the "AE" column means the *working rep* (the LSM), not the enterprise AE. | Source: [[hubspot.md]] 2026-08-28 changelog, "Field semantics corrected." Field name, not the word "AE" in a sentence, tells you which one. |
| **CLTV** | **Combined Loan-to-Value**, one metric name, **three separate columns on `bbys_leads`**: `combined_loan_to_value_ratio_house_canary_value` · `combined_loan_to_value_ratio_equity_boost_exception_value` · `combined_loan_to_value_ratio_heloc_agent_lender_value` (plus `_intake` and `approved_combined_loan_value_ratio`) | Product **ceiling**, a separate concept: `base-unlock-70` 70% / `equity-boost-85` 85% / `heloc-90` 90%, defaulting to 90% for unknown product IDs | — | "What's our average CLTV" is unanswerable without naming the column *and* the product. Source: [[bi-metrics-definitions]] §"CLTV — three columns, plus product ceilings". |
| **CTC** | **Clear To Close** — the LO's own lender system's term | **Clear To Fund** — the HomeLight/Sales-App stage name (`dealstage` 998755446), reused as the Slack channel-name token `-ctc-` | — | The two are **often confused in practice** per [[bbys-deal-channel-vocabulary]] §44 ("CTF / CTC ... often confused"). When a Slack channel says `-ctc-`, that's HomeLight's Clear To Fund stage, not the LO's Clear To Close. |
| **US News** | **Organic co-brand lead-gen site** — Jekyll → S3 static-site generator, documented in [[repo-homelight-growth]] | **Stripe-backed paid agent sponsorship product** — built in HAPI May–Aug 2026 by Emmanuel Diaz; Production Index ≥1.0 qualification, <10-transaction pre-filter | — | Ask which before answering any "US News" question. Source: [[growth-marketing-d2c]] §"'US News' is two systems (2026-08-29)". |
| **ML** | Common misread: a machine-learning / matching system | **Actual meaning of `ml-uploader`: MeridianLink**, a loan-origination-system integration tool internally called "Jootie." Zero relationship to any ML/AI matching system — confirmed by a full grep of the repo for machine-learning terms returning zero matches. | — | Source: [[repo-bi-queries-and-ml]] §"ml-uploader ('Jootie') — MeridianLink, not Machine Learning" 🔴 flagged there as a correction to a prior working assumption. |
| **stage** (deal stage / lifecycle stage) | At least **four separate numeric stage scales** exist across HAPI, dbt, Periscope, and HubSpot's `dealstage` — none share a numbering scheme | — | — | **Do not try to memorize these here.** Full reconciliation lives in [[bi-metrics-definitions]] §"The core problem: four numeric stage scales" — go there, this file only flags that the collision exists. |
| **TAM** | **"Top Agent Market"** — a metric/data cut in the Elite/Agent MBR dataset (`elite_and_tam_mbr`, `tam_agents`, `tam_rachel`, `tam_slu`). Source: [[bi-agent-and-finance-analytics]] §"Elite / TAM performance" | Reads as a **person or role** in HubSpot workflow copy — "TAM books meeting → notify LSM" ([[hubspot-workflows]] §"TAM <> LSM Notifications"); [[lo-lifecycle]] uses "TAM ↔ LSM notifications" the same way | — | **UNRESOLVED.** No vault doc states what role or team "TAM-the-person" refers to (Territory Account Manager? Top Agent Manager?) — it may be a legacy/external system's term picked up verbatim into these workflow names. Flagged, not solved — see Open questions. |
| **CA (state)** | California, the US state — appears constantly in buy-box/state lists ([[bbys-buy-box-and-eligibility]]) | Client Advisor / Contract Advisor (see CA role collision above) | — | Context always disambiguates (a state list vs. a role sentence), but grep for bare "CA" will surface both — don't trust an automated search without reading the line. |

---

## Roles

Canonical role vocabulary is `TransactionTeam::EMPLOYEE_ROLES` in HAPI — source:
[[bbys-routing-and-ownership]] §"The role vocabulary". Post-reorg display renames noted where
they exist; the underlying `team_role` DB values are **unchanged** by the rename (decision D2).

| Term | Expansion | Meaning | Source | Notes |
|---|---|---|---|---|
| **LSM** | Lender Success Manager (canonical) | Works the LO relationship; the primary sales rep role for the lending channel | [[team]] | See collision section. Renamed **AE** (display only) at Sep-1 cutover. |
| **LRM** | Lender Relationship Manager | Post-approval handoff owner; works deals after LSM origination | [[team]] | Renamed **AM** (Application Manager, display only) at Sep-1 cutover. |
| **SRM** | Strategic Relationship Manager | Owns AGENT relationships, carries a handful of LO contacts | [[team]] | 🔴 See correction above — do not use the "Suppression list" definition. |
| **CA** | Client Advisor *or* Contract Advisor | Two distinct real roles sharing one abbreviation | [[bbys-routing-and-ownership]] | See collision section. Contract Advisor renamed **Closing Manager** at reorg. |
| **AE** | Account Executive | Enterprise AE (3 people) *or* post-reorg LSM rename *or* partner-tracker "rep" column | [[hubspot.md]] | See collision section. |
| **AM** | Application Manager | Post-reorg display name for LRM | [[team]] banner | Display-layer only; `team_role` stays `'LRM'`. |
| **LOS** | Lender Operations Specialist | `lender_operations_specialist` role slug | [[bbys-routing-and-ownership]] | Not to be confused with "loan origination system" — that meaning does not appear as an acronym in this vault. |
| **LOA** | Loan Officer Assistant | `loan_officer_assistant` role slug | [[bbys-routing-and-ownership]] | ⚠️ Deprecated role per the same source table. |
| **AAM** | Agent Account Manager | `agent_account_manager` role slug, agent-marketplace side (not BBYS/lending) | [[bbys-routing-and-ownership]] | |
| **ASM** | Agent Success Manager (inferred label) | `agent_success_manager` role slug, agent-marketplace side | [[bbys-routing-and-ownership]] | Also collides with `asm-portal`, a deprecated internal repo name ("AS App" / Agent Services) — see [[repo-sales-react]]. Different thing entirely; the repo name is not the role. |
| **BRM** | Builder Relationship Manager | `builder_relationship_manager` role slug | [[bbys-routing-and-ownership]] | Round-robin enum value `brm`. |
| **TP** | UNRESOLVED — round-robin enum value `tp` (alongside `lrm`/`ca`/`brm`) | Possibly "Transaction Processor" | [[bbys-routing-and-ownership]] §Open questions | Flagged as unresolved in that file too; carried here unsolved. |
| **CRO** | (role of file, not a person) Contingency Removal Only | IRUC variant where EU = $0, often paired with DTI Drop; skips the EU loan-signing step | [[bbys-deal-channel-vocabulary]] | Listed under roles only because it's frequently mistaken for a title in Slack shorthand — it describes a *file type*, not a person. |
| **LRM Assistant** | — | Saint Lucia outsourced team handling aged BBYS deals post the 2-week automation window | [[team]] | `team_role='LRM_ASSISTANT'`, one active person (Janice Clement) as of 2026-06-19. |
| **Listing Ops** | Listing Operations Specialist | Owns the listing side of BBYS deals; `listing_operations_specialist` field is a NAME string, not an owner ID | [[team]] | Three of four are not HubSpot owners. |

## Products, programs, and tiers

| Term | Expansion | Meaning | Source | Notes |
|---|---|---|---|---|
| **BBYS** | Buy Before You Sell | The core product | [[bbys-overview]] | |
| **EU** | Equity Unlock | Equity from DR advanced to fund IR purchase | [[bbys-overview]] | |
| **EB** | Equity Boost | Additional lending beyond standard EU | [[bbys-overview]] | |
| **HELOC** | Home Equity Line of Credit | New product, launched 2026-03-04 | [[bbys-overview]] | Originated/serviced by HLHL. |
| **UGA** | Upside Guarantee Agreement | Post-program-period mechanism: HL buys at LPV, relists, remits net profit to client | [[bbys-overview]] | |
| **BYOC** | "Bring Your Own Cash" | Legacy name for [[dti-drop]]; still the flag name in Sales App, agreements, email templates | [[bbys-overview]] | Same product as DTI Drop. |
| **DTIDA** | DTI Drop Agreement `(inferred)` | The paperwork/agreement product for the DTI Drop / Contingency-Removal-Only path, separate pricing from the three CLTV-tier products | [[bbys-overview]] §"Pricing constants" | Not spelled out anywhere in the vault; expansion inferred from context ("`contingency_removal_only`... `DTIDA` agreement"). |
| **BBYSA** | BBYS Agreement | The client-signed program contract | [[bbys-deal-channel-vocabulary]] | One of four signable docs in flight alongside PSA/RPA/BUO. |
| **PSA / RPA** | Purchase & Sale Agreement / Residential Purchase Agreement | The buy-side contract docs | [[bbys-deal-channel-vocabulary]] | |
| **BUO** | Backup Offer | DR backup-contract document, 3-step review/sign chain | [[bbys-deal-channel-vocabulary]] | |
| **MOA** | Memorandum of Agreement | Recording-related, post-funding step | [[bbys-deal-channel-vocabulary]] | "MOA Closing." |
| **TI+** | "Trade-In+" (legacy) | Pre-BBYS product, retired early 2024. Separate `trade_in_leads` table, `user_type = cc_trade_in` | [[bbys-overview]] | Recurring source of report count mismatches vs BBYS. |
| **Cash Offer** | — | ⚠️ In 2026 Slack, almost never means HomeLight's legacy Cash Offer product — usually a generic industry term, a competitor's investor cash offer, or a third-party platform ("Cash Offer Pro") | [[homelight-company-overview]] §1 | HomeLight's own "Cash Close / Trade-In / Cash Offer 2.0" is **sunset** — zero 2026 Slack traffic. |
| **HLM** | HomeLight Listing Management | Was `disclosures.io`, cut over to `hlm.homelight.com` 2026-08-19 | [[homelight-company-overview]] | Also informally called "HomeLight Marketplace-adjacent" in older Slack usage ([[growth-marketing-d2c]]) — same product, HLM is the operative abbreviation now. |
| **HLCS** | HomeLight Closing Services | Title/escrow arm, Qualia-based | [[bbys-overview]], [[hlcs-title-escrow]] | |
| **HLHL** | HomeLight Home Loans | Originates + services the HELOC leg | [[bbys-overview]] | |
| **EVA** | (name, not an acronym expanded in-vault) AI escrow agent | Launched 2026-04-27, cost-out inside HLCS, publicly positioned as a platform | [[homelight-company-overview]] | Repos `eva-ai`, `eva-api`, `eva-ui`. |
| **PPL** | Pay-Per-Lead | Flat price per intro, billed via Stripe metered usage; the investor-network revenue model that replaced rev-share | [[homelight-agent-marketplace]], [[homelight-company-overview]] | "PPL takes priority over rev share" per the 2026-07-17 note. |
| **GVG (capital)** | `gvg_capital` — a named investor/lead-buyer partner, not a defined acronym | One of ~15 partners tracked weekly in the investor network's `PARTNER_MARKETING_SOURCES` | [[growth-marketing-d2c]], [[homelight-agent-marketplace]] | Referenced in the 2026-07-28 expected-revenue haircut (GVG early-stage leads cut ~30%). Company name, not an acronym to expand. |
| **ELP** | Elite Lender Program | The LO-channel retention/tiering program | [[lo-lifecycle]] ("ELP Dashboard") | Tiers: Elite (3+ YTD closes) / Diamond (6+) / Obsidian (11+) — see below. |
| **CMF** | Closing Management Fee | $1,450, waived if the seller uses HLCS | [[bbys-overview]] | |
| **VP** | Variable Pricing | Fee scales with days-to-DR-sale; now a Sales App "Pricing Model" field | [[bbys-overview]] | |
| **DUS** | Document Upload Service | Live, in-flight rewrite of BBYS document upload — replaces the old presign→S3→pdf-merge flow with explicit, stateful sessions | [[bbys-document-pipeline]] | Gated by flags `document-upload-service-enabled` + the (confusingly shared-named) `meridianlink-loan-creation-integration-phase2`. |
| **DDE** | `document_data_extraction_service` | Powers AI document-extraction review bots downstream of DUS | [[bbys-document-pipeline]] §5 | DUS is the plumbing, DDE + Extend AI is the intelligence layer. |
| **BESI** | (system integration identifier, not spelled out in-vault) | `encompass_loan_application_id` is actually a **BESI id** — Encompass itself never shipped. The column name is a lie left over from an integration that changed direction. | [[bbys-integration-map]] | See "BESI, resolved" in [[repo-eave-hlhl]] for the fuller trace — two LOSes (loan origination systems) now serve two rails of the same lender, synced by BESI. |

### Elite Lender Program tiers

| Tier | Threshold (YTD IR closes, current calendar year) | Count as of 2026-07-28 dashboard | Source |
|---|---|---|---|
| Elite | 3+ | ~152 LOs ([[lo-lifecycle]]) / bucket "3+" 55–58 depending on dedup pass | [[tools-and-stack]] Elite LOs Dashboard changelog |
| Diamond | 6+ | 24 LOs | [[lo-lifecycle]] |
| Obsidian | 11+ | 6 LOs | [[lo-lifecycle]] |

⚠️ Cohort math has moved multiple times on documented dates — 954 (post-Orchard exclusion) →
924 (post-dedupe) → 856 (post-builder exclusion). **Attach a date to any Elite/Diamond/
Obsidian count you quote.** See [[tools-and-stack]] for the full changelog.

### Approval-speed tiers (a different "tier" — do not conflate with Elite/Diamond/Obsidian)

`enum approval_type: { express: "express", instant: "instant", light: "light", rapid:
"rapid" }`, default `express`. These are the `express` / `light` / `rapid` (and rarely
`instant`) suffixes in Slack deal-channel names (e.g. `#uwm-appr-hartog-...-ca-express`).
Gated by flags `bbys-light-approvals`, `bbys-rapid-approvals` (+ `-create-tasks`). Source:
[[bbys-lifecycle-operations]] §"Four tiers, express is the default."

⚠️ **"Tier" is itself an overloaded word** — approval-speed tier, Elite/Diamond/Obsidian
tier, and `partnership_tier` (a HubSpot field on Partnerships, unrelated to either) are three
different concepts that all get called "tier" in conversation.

## Deal / stage codes

Slack deal-channel naming: `[lender-prefix]-[stage-token]-[address]-[state]-[approval-tier]`.
The `HomeLight Sales App` bot renames the channel on every stage transition — the rename event
is the most reliable stage-transition signal in the system. Source:
[[bbys-stage-progression]], [[bbys-integration-map]].

| Token | Stage | Notes |
|---|---|---|
| `new` | New lead | |
| `rev` | In Review | |
| `appr` | Approved | |
| `as` | Agreement Signed | |
| `iruc` | IR Under Contract | Also called "IR In Escrow" / "IR Contract" — HubSpot's stage label is `IR_IN_ESCROW` (998755445) but the bot's channel-rename text says "Ir Contract." |
| `ctc` | Clear To Fund | ⚠️ See CTC collision above — this is HomeLight's stage, not the LO's "Clear to Close." |
| `irx` | IR Closed | Primary close milestone; stage 998755447. |
| `drx` | DR Closed | Final close; stage 998815822. |
| `term` | Terminated / failed | Channel renames to `-term-` on file failure, from any prior stage. |

**Do not re-derive the four numeric stage scales here** (HAPI's, dbt's, Periscope's, and
HubSpot's `dealstage` — none share a numbering scheme). That reconciliation is
[[bi-metrics-definitions]] §"The core problem: four numeric stage scales" — go there.

Other stage-adjacent terms:

| Term | Meaning | Source |
|---|---|---|
| **IRUC** | IR Under Contract | [[bbys-overview]] |
| **IRX** | IR Closed | [[bbys-overview]] |
| **DRX** | DR Closed | [[bbys-overview]] |
| **CTF** | Clear To Fund (spelled out) | [[bbys-deal-channel-vocabulary]] — same stage as the `ctc` channel token; "CTF/CTC often confused" per that doc. |
| **COE** | Close of Escrow (date) | [[bbys-deal-channel-vocabulary]] |
| **SAL** | Sales Accepted Lead | The middle stage of the MQL → SAL → SQL lender-qualification funnel; stamped the first time a non-blank sales-responsiveness label is written | [[bbys-lender-qualification-funnel]] |
| **SQL** | Sales Qualified Lead | LO who passes qualification scoring | [[bbys-overview]] |
| **MQL** | Marketing Qualified Lead | Engagement score ≥ 50 | [[bbys-overview]] |
| **RON** | Remote Online Notarization | State-gated closing-signing method; the gating list lives in a PDF, not queryable config | [[sales-ops-operating-model]] §"Closing ops" |
| **NBS** | Non-Borrower Spouse | Fields added 2026-05-28 for community-property-state signings; plumbing not fully reliable as of 2026-06-23 | [[sales-ops-operating-model]] |
| **DR** | Departure Residence | The home being sold | [[bbys-overview]] |
| **IR** | Incoming Residence | The home being purchased | [[bbys-overview]] |
| **EUC** | Early Use of Cash | Clients who drop out early | [[bbys-overview]] |
| **PO** | Payoff (demand) | Itemized payoff issued at DR close | [[bbys-overview]] |
| **DOM** | Days on Market | Drives the interest-expense day count in CPAI | [[bbys-overview]] |

## Systems and metrics

| Term | Expansion | Meaning | Source | Notes |
|---|---|---|---|---|
| **RAV** | Risk-Adjusted Value `(inferred from field name risk_adjusted_value)` | The valuation figure after risk adjustment; drives the guaranteed price | [[bbys-buy-box-and-eligibility]], [[sales-ops-operating-model]] | Market-level RAV adjustments are directive-driven (Jason Smith issues standing Slack guidance), not systematized. |
| **RAP** | Risk-Adjusted Percentage | The multiplier applied to get from estimated value to RAV, keyed off a value-ratio lookup table | [[bbys-buy-box-and-eligibility]] §"Risk-adjusted percentage (the RAP tables)" | **Three tables exist** — base, `RISK_ON_RAP` (v1.3, more generous), `RISK_OFF_RAP` (more conservative). Establish which is live before modeling anything; the base table isn't even versioned. Peaks at 0.85 when estimated value ≈ AVM. |
| **LPVAL** | Task tag, expansion not stated in-vault | Filters valuation tasks in Sales App; on completion, triggers a manual follow-on "Conditional Approval" / "Refresh Approval" task | [[sales-ops-operating-model]] §5 | `UNRESOLVED` — likely "Loan Payoff Valuation" or similar but not spelled out anywhere sourced. |
| **LPV** | Loan Payoff Value | The figure HL purchases at under the UGA; appears in the econ model | [[bbys-overview]] | Do not confuse with LPVAL (task tag, above) — different terms, similar letters. |
| **CPAI** | Contribution Profit After Interest | **Not** "cost per application" — Finance/BI owned metric | [[bbys-overview]] | |
| **TPD** | Transactions Per Day | Cohort performance metric | [[bbys-overview]] | |
| **BIPs** | Basis points ("bps"), of DR volume | 🔴 **SUPERSEDED 2026-04-01** — see [[comp-plans]] §0 | [[comp-plans]], [[bbys-overview]], [[q2-priorities]] | The old model ("BIPs of DR Volume + IRAX bonus, threshold 200") is dead. Current LSM comp is bps per the **LO's Nth close** (6/10/13/2) on a derived **comp-able value**, plus a quota accelerator. LRMs: 3 bps flat. |
| **IRAX** | IR Close bonus threshold | 🔴 **SUPERSEDED 2026-04-01** — replaced by the quota-attainment accelerator | [[comp-plans]] §0, §3 | The `IRX` root survives in the deal fields `lo_bbys_irx_number` / `lo_s_irx_count` (the LO's Nth close, tier selector) — but "IRAX" as a bonus threshold no longer exists. **2026-08-29:** the Data Bridge KB's `bot_facing` "BBYS Terminology & Acronyms" article (dated April 2026, feeds the HubSpot-Helper Slack bot) still defines both BIPs and IRAX with no supersession note — the bot may be teaching the dead comp model. See [[data-bridge-kb-index]]. |
| **Comp-able value** | The comp basis (2026 plans) | `hl_deals_lp_bbys_lo_est_dr_value` if ≤$1M, else `homelight_value × 1.05`; `homelight_value` if the LO estimate is null | [[comp-plans]] §2 | ⚠️ A **third** revenue-adjacent basis alongside `bbys_revenue` and `actual_program_fee_amount` — it will not reconcile to either. |
| **AE** | Account Executive | ⚠️ **Collision.** (a) The Sep-1-2026 rename of **LSM**; (b) a *different, older* population in john.labrada@'s Feb-2025 "AE Comp Plan"; (c) in the Aug-2026 wholesale allocation docs, "current AE"/"proposed AE" refer to the post-reorg wholesale owners. | [[comp-plans]] §8, [[sales-ops-operating-model]] §8, [[team]] | Always date-qualify "AE." |
| **TE** | Target Equity | The approval threshold that splits the funnel: approvals **at TE** convert to IR Contract at **60%**, approvals **below TE** at **30%** | [[bbys-unit-economics]] §11, [[bbys-equity-boost]], [[heloc-product]] | 🔑 The 2:1 conversion gap is the entire economic case for Equity Boost / HELOC Boost. |
| **DPL** | (funding metric, "DPL funding") | ⚠️ **Three separate, substantially-different implementations**: dbt `bbys_dpl_funding.sql`, Periscope `views/bbys_dpl_funding`, Periscope `views/bi_bbys_dpl_funding` | [[bi-metrics-definitions]] §"DPL funding — three definitions", [[repo-bi-periscope]] | Name a source every time you cite it. |
| **CLTV** | Combined Loan-to-Value | See collision section — three separate DB columns plus product ceilings | [[bi-metrics-definitions]] | |
| **MBR** | Monthly Business Review | The recurring exec/finance reporting cadence — D2C MBR, HLCS MBR, Elite/TAM MBR are each separate decks | [[homelight-company-overview]], [[bi-agent-and-finance-analytics]] | `elite_and_tam_mbr` is the literal data source table backing the Elite/TAM MBR deck. |
| **TAM** | "Top Agent Market" (metric cut) *or* a person/role in workflow copy | See collision section — unresolved | [[bi-agent-and-finance-analytics]], [[hubspot-workflows]] | |
| **CRO** | Contingency Removal Only | IRUC variant, EU=$0 | [[bbys-deal-channel-vocabulary]] | Also listed under Roles above (common miscategorization in Slack shorthand). |
| **PCQ** | (term abbreviated in a Slack quote, not spelled out) | Appears in "missing House Canary AVM can make PCQ n/a" | [[bbys-lifecycle-operations]] §48 | `UNRESOLVED` — single occurrence in the vault, no defining context found. Likely a pre-qualification/pricing check of some kind. |
| **EUR** | (term abbreviated in the same Slack quote as PCQ) | "EUR = $0 blocks max-light [approval]" | [[bbys-lifecycle-operations]] §48 | `UNRESOLVED` — likely "Equity Unlock Result" or similar `(inferred)`, not spelled out. Do not confuse with the currency. |
| **AIOT** | "AI Over Text" | The text-claim referral flow (vs. digital-call claim) in the agent dashboard | [[repo-agent-dashboard]] §2.2 | `claimType: 'aiot' \| 'digital_call'`. |
| **TAD** | Technical Approach Document | Output artifact of the internal voice-scoping tool `homelight-product` | [[repo-small-econ-batch]] | Not a BBYS/RevOps term — internal eng-tooling artifact, included here because it's an orphan acronym that would otherwise look undefined. |
| **smlac** | `same_month_lead_after_close` (field-name fragment, not a spoken acronym) | Splits leads created after a same-month close from those created before, in the `_bbys_retention_v2` retention definition | [[bi-metrics-definitions]], [[repo-bi-periscope]] | `smlac_lender_leads` / `smlac_active_lo_flag` column names. |

> Orchestrator note (2026-08-29): PCQ and EUR were initially marked unresolved but are
> defined in [[bbys-lifecycle-operations]] (PropertyConditionQuestionnaire snapshot type) and
> [[2026-08-28-vault-refresh-slack-findings]] §2/§15 respectively — rows fixed above.

## Open questions

- **TP** (round-robin role enum, alongside `lrm`/`ca`/`brm`) — carried over unresolved from
  [[bbys-routing-and-ownership]]. Possibly "Transaction Processor."
- **TAM-as-person** — what role or team does "TAM books meeting → notify LSM" actually refer
  to? Not the same TAM as "Top Agent Market." No vault doc names it.
- **LPVAL** expansion — task tag used constantly by the valuations team but never spelled out
  in any source read for this pass.
- **PCQ** and **EUR** — both single-occurrence terms from one Slack quote in
  [[bbys-lifecycle-operations]] §48. Worth a targeted Slack search around that quote's date if
  someone needs them precisely.
- **GVG capital** — confirmed as a partner/investor name, not an acronym, but no doc states
  what (if anything) the letters stand for. Low priority; treat as a proper noun.
- Whether any other `context/*.md` file still carries a **stale acronym expansion**
  contradicting this file — this pass covered every file grepped for ALL-CAPS tokens, but a
  full byte-for-byte diff against every doc's own terminology callouts was not done. A future
  pass should grep each new context doc against this file's collision list before merging.
