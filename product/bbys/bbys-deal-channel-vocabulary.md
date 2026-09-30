---
last_updated: 2026-08-28
type: context
source: HomeLight-Vault/context/bbys-deal-channel-vocabulary.md
imported: 2026-09-29
---

> **Critical update 2026-05-13 (PM):** the "Bot / automation accounts" section below frames HomeLight Sales App + other bots as deprioritization candidates. That's correct AT THE RELEVANCE-SCORING LAYER but DANGEROUSLY WRONG at the FETCH layer. Bot messages are the PRIMARY content carrier on fresh BBYS deal channels — they post Submission confirmation, Required-from-LO action items, TitleBot pulls, Snapdocs status, etc. Filtering by `m.subtype` at fetch eats 100% of this content and makes the AI Draft see a thin timeline on channels that are anything but thin. See dedicated section "Bot messages are CONTENT, not noise" below + [[2026-05-13-slack-bot-messages-are-content-not-noise]].

# BBYS Deal-Channel Vocabulary

> **This document is the shared protocol for how BBYS deals flow through Slack.** Originally scaffolded as AI-Draft prompt-engineering reference, but it has grown into the canonical machine-readable description of BBYS process. Any future feature that reads, analyzes, summarizes, or routes off deal channels should treat this doc as its source of truth — read FROM the patterns below rather than re-deriving them from scratch.
>
> **Consumers (current + planned):**
> - **AI Draft (live)** — OUTSTANDING ACTION ITEMS detection uses the action-pair table verbatim. See [[2026-05-12-deal-context-ai-draft]].
> - **Priority Queue (live)** — Wave 1 signal extraction reads from the open/resolved phrase lists. Future Phase 3b will read action-pairs for stale-thread detection.
> - **Deal Summary generation (planned)** — needs the action-pair table to decide what's still outstanding vs done.
> - **Anomaly detection (planned)** — a channel that's "stuck" is one where action-requested signals have no matching completion signal within the expected window.
> - **Stage-flip prediction (planned)** — combinations of completion signals predict stage transitions before HubSpot fires them.
>
> **Format conventions for adding to this doc:**
> - Phrase lists (open / resolved signals): literal substrings the channel produces. Test against word boundaries; document specific false-positive risks.
> - Action-pair table rows: action signal + completion signal + (where applicable) the HubSpot milestone that mirrors the completion. Rows include source: who validated the pattern (n=30 May 12, n=NN May 14, etc.).
> - Bot identities: name + Slack user ID + a one-line content summary.
>
> **Sampling history:**
> - **n=30, 2026-05-12** — original sample, 5 channels per stage × 6 active stages.
> - **n=35, 2026-05-13** — outbound rep email voice study (separate sample).
> - **n=28, 2026-05-14** — round-1 extension across all 6 stages (~2,100 messages reviewed); cross-referenced against HubSpot milestone properties (`dealstage`, `bbys_approved_date`, `bbys_agreement_signed_date`, `bbys_ir_contract_date`, `bbys_ir_close_date`, `bbysa_signed_date`). Added 20 new action-pair rows + Cheryl Funk's `*UC Notes*` block + new "Slack-as-source-of-truth" section + 8 new acronyms.
> - **n=25, 2026-05-14 PM** — round-2 outlier-focused sample: 9 terminal channels, 4 stuck/Approved, 5 builder, 5 fresh-batch, 2 trust (+ cross-channel search validation, 12 hits). **Combined n=53.** All 8 round-1 high-frequency patterns validated at the new sample size (no contradictions). Added 10 more pair rows (terminal-from-any-stage, death-cancel, archive-revive, Section flags block, first-time-LO, 180-day pilot, duplicate-lead, EU-negative analyst voice, `-irx-` IR Closed post-close ops, Solar lease/owned variant — corrects round-1's "boolean only" claim). NEW per-stage entries added: IR Closed (`-irx-`) and Failed (`-term-`). NEW LSM→LRM handoff section.
> - **n=55 outbound emails, 2026-05-14 PM** — stage-stratified voice study (separate from the channel sample). Bucketed real-rep outbound emails by lifecycle state (New 9 / Approved-pre-AS 8 / "Manson state" 9 / IRUC 12 / CTC 9 / `-irx-` 8). 12 distinct senders represented. Forced corrections to PR #195: the "are you shopping?" template (0/9 corpus hits) was replaced with corpus-validated Deborah Shutt + Brandi Cirell phrasings. Added: CLIENT-CALL-NARRATION 5th opener (IRUC), HANDOFF/DECLINE 6th opener with per-state recap exception, APPROVAL BLOCK structured artifact (State B), CTC one-line allowance, solar-gating opener, "move the file along" stock verb, 70% generic-open-door closer distribution. **Drip-campaign + OOO corpus contamination** noted as a methodology constraint for any future voice study (~40% of raw outbound emails are HubSpot Sales-Engage templates).
> - **n=10,357 LRM emails, 2026-05-14 PM** — sales-positioning voice study (180-day window, 6 LRM senders, drip + OOO + `hubs.ly` + `hmlt.co` filtered, quoted-reply tails stripped). **NEGATIVE RESULT:** 10 of 11 textbook positioning phrases had 0 hits. The ONE survivor: `non-contingent` (65 hits, 71% in State B agreement-signing-nudges). Shipped to AI Draft prompt (PR #200) as forbid-by-default + State-B carve-out + bounded USER GUIDANCE override. Established the meta-pattern: corpus-validate before adding ANY new prompt rule; negative results are first-class outcomes.
>
> Related: [[hubspot]] · [[bbys-overview]] · [[bbys-edge-cases]] · [[bbys-lender-qualification-funnel]] · [[2026-05-12-deal-context-ai-draft]] · [[2026-05-13-slack-bot-messages-are-content-not-noise]]

## Acronym glossary

These come up constantly in channels; treat them as their own tokens, not generic English.

| Term | Meaning | Where it shows up |
|------|---------|-------------------|
| **IR** | Incoming Residence — the home being purchased | "IR COE", "IR contract", "IR title" |
| **DR** | Departing Residence — the home being sold via BBYS | "DR Back up RPA", "DR Closings" |
| **IRUC** | IR Under Contract — stage flip after purchase contract executed | Channel suffix `-iruc-`, celebratory ":raised_hands: IRUC" |
| **UC on IR** | Same concept at intake (boolean field) | "Already UC on IR" alerts |
| **CTF / CTC** | Clear to Fund (HomeLight stage) / Clear to Close (LO's lender system) — often confused | "Receive CTC" in checklists |
| **COE** | Close of Escrow date | "IR COE 5/21" |
| **EU** | Equity Unlock — core product | "Max EU", "EU amount", "target unlock" |
| **EB** | Equity Boost (with asset docs requirement) | "EB approved", "request EB" |
| **EUID** | Equity Unlock loan signing docs | "EUID has been signed", "trigger EU loan signing email" |
| **BBYSA** | BBYS Agreement (client-signed program contract) | "Signed final BBYSA" |
| **BUO** | Backup Offer (HomeLight fallback if DR doesn't sell) | "BUO has been signed", "Update and Send Backup Offer" |
| **CA / CA Notes** | Client Advocate handoff template at IRUC. Variants from Ashlee Kim, Kyle Bradish, Tiffany Traxler, Cheryl Funk, Ian Pardo. Note: the `Solar` field accepts `No` / `Yes - Lease already provided` / `Yes - Owned` (not a boolean); the `Trust` field carries trust type + executed status when applicable. | 10-line block: IR Address, COE, Financing, EU, EB, Contingency Removal Only, Second Lien, Solar, Trust, HOA |
| **CMF** | Closing Market Fee | "cmf_waived", "cmf_applicable" in SMS templates |
| **DTI Drop** | Debt-to-Income Drop product variant | "EB: false, DTI Drop: true" |
| **LPV** | Lender Property Valuation | |
| **CLTV** | Combined Loan-to-Value | |
| **HOI** | Home Owners Insurance | "HOI Update Received", "HOI Cleared Date" |
| **LO / LRM / Lender Ops / LOS** | Loan Officer / Loan Relationship Manager / Lender Operations / Loan Ops Specialist | Channel topic always lists all three roles |
| **LA / DRA / NHC / BSR** | Listing Agent / DR Agent / New Home Consultant (builder) / Builder Sales Rep | |
| **CIF** | Client Information Form | Appears in Pending checklists |
| **RR** | Required Repairs (property conditions) | "pre hl purchase - required repair - Roof" |
| **PSA / RPA** | Purchase & Sale Agreement / Residential Purchase Agreement | The four signable docs in flight: PSA / RPA / BBYSA / BUO |
| **HELOC** | Home Equity Line of Credit | Bell notification at intake when state is eligible |
| **CRO** | Contingency Removal Only — IRUC variant where EU=$0 (often DTI Drop), skips EU loan signing step | `*Contingency Removal Only Next Steps Email sent!*` block at IRUC |
| **MOA** | Memorandum of Agreement / MOA Closing — recording-related, post-funding step | "schedule MOA Closing" in Cheryl's UC Notes; "MOA recording" in CTC channels |
| **IMAD** | Wire confirmation code returned with each disbursement | `IMAD to IR Title: 20260511F7B74M2C000505` in Closing Signing Request Form post |
| **BSR** | Builder Sales Rep (Lennar / K-Hov / Khovnanian) — sibling to NHC | Appears in builder-channel handoffs alongside NHC |
| **EUA** | Equity Unlock Approval — sent to client after approval, kicks off Winback FUP automations | "EUA received follow-up" in HomeLight Homie automation subject lines |
| **Soft credit colors** | `Green` = clean (auto-pass), `Yellow` = manual review trigger, `Failed` = manual pull required. Bureau code follows (`TU` / `EQ` / `EX`) | `Intelligence: Yellow EX Result: Passed` in `:credit_card: *Auto Soft Credit Check Results*` post |
| **2-for-1 condo merge** | Two clients moving in together, each selling their own condo, often need DTI Drop on one of the two units | "his clients are moving in together and selling each of their condos … they may just need DTI drop for one condo" |

## Channel conventions

### Patricia Pinckard's `:extreme-teamwork:` sigil
One of two canonical IRUC+ checklist artifacts. Pinckard (and Gessa) posts a `:extreme-teamwork: *Pending:*` block per deal with a bulleted checklist.
- **Strikethrough markdown (`~text~`) = done.**
- **Plain bullets = still open.**
- Future v2 of any relevance scorer could parse strikethrough state for granular open/done tracking.

### LSM → LRM handoff at IRUC
Added 2026-05-14 round 2. The LSM-to-LRM handoff at IRUC is NOT a single canonical message. It's a two-part artifact:

1. **`Email sent: BBYS_BR_IRUC_CA_Handoff`** — automated bot event from HomeLight Sales App, fires at stage flip to IRUC. This is the system-side handoff record.
2. **CA Notes block** — the LSM (Ashlee Kim, Kyle Bradish, Tiffany Traxler, Cheryl Funk, Ian Pardo each have their own variant) posts a structured 10-line block per the [[bbys-deal-channel-vocabulary#Channel conventions]] CA Notes template (IR Address / COE / Financing / EU / EB / Contingency Removal Only / Second Lien / Solar / Trust / HOA).

The CA Notes block IS the LSM's curated handoff to the LRM. There's no separate "I'm handing this off" message — the CA Notes block content is the handoff. Future features needing to detect handoff completeness should check for both the `BBYS_BR_IRUC_CA_Handoff` bot event AND the CA Notes block from the LSM.

### Cheryl Funk's `*UC Notes*` block
The second canonical IRUC checklist artifact, observed in 2/5 sampled IRUC channels (Carranza, Luna) on the 2026-05-14 sample. Different markdown shape from Pinckard's sigil but same purpose.
Format:
```
*UC Notes - LO DR Val: $X*
...
5/14: Pending:
-Inspection - *date*
-Final BBYSA + RPA - not sent
-schedule MOA Closing est:
-CTC?
```
- **Items get crossed off in subsequent posts** — often Cheryl edits-in-place or reposts the updated block.
- **Maps to multiple HubSpot milestones simultaneously** (inspection, final BBYSA, MOA, CTF) — one block summarizes 3-5 pending properties.
- A relevance scorer or summary feature should treat both Pinckard's `:extreme-teamwork:` block AND Cheryl's `*UC Notes*` block as canonical IRUC source-of-truth.

### Channel rename pattern is the canonical stage signal — with one caveat
`-new-` → `-rev-` → `-appr-` → `-as-` → `-iruc-` → `-ctc-` → `-term-`. The `HomeLight Sales App` posts "has renamed the channel from … to …" — that's the most reliable stage-transition event in the system. More reliable than reading message bodies.
- **Caveat from the 2026-05-14 sample:** channels occasionally bounce-rename (observed: `fairway-appr-sweeney → fairway-rev-sweeney → fairway-appr-sweeney` then `fairway-as-sweeney` within 2 hours). If a feature keys on channel name alone, it'll race against the actual `dealstage` flip. **Treat the most recent `HomeLight Sales App: has renamed the channel from X to Y` message as authoritative, not the current channel-name string.**
- **Stronger alternative:** `HomeLight Support: Stage UPDATED to <name>` posts. Stage-flip messages from this user are more reliable than the channel rename and don't suffer from bounce-renames. Treat as the primary stage-flip signal when both are present.

### DocuSign event ordering
1. `:eyes:` — viewed (noise, no real signal)
2. `:lower_left_ballpoint_pen:` — signed
3. `:white_check_mark:` — all clients done → stage flips

Only events 2 and 3 carry real signal.

### "fully executed: no"
Embedded in the AI-IR-contract reviewer's auto-template. If this appears in an otherwise-resolved-looking block, the contract is NOT actually executed — that's an open signal hidden inside a template.

### Two-LO-deal pattern
"2 for 1 with [other-client]" — when two BBYS files combine into one IR purchase. Channels cross-reference each other in CA Notes ("This is in combination with 731 Easter Lily Pl (Natasha Parker)"). Treat these references as high-signal — they often imply blocking dependencies.

## Open-thread signal phrases

What "this thread still needs action" looks like in real channels:

**Strong (unambiguous open signals):**
- `:extreme-teamwork:` + `pending:` (Patricia's sigil)
- `fully executed: no` (AI-IR-contract review template)
- `please send` / `please upload` / `please correct`
- `still waiting` / `haven't received` / `haven't gotten`
- `lvm for` / `left voicemail` / `called and texted` / `no answer`
- `any update` / `where are we on`
- `have not reviewed` / `has not been scheduled`
- `additional document request` / `task can not be processed`
- `failed to send` / `INVALID_RECIPIENT_ID` (real tech blockers, look like noise)
- `needs to apply`
- `can we rush` / `rush this` / `to close on time`

**Weaker (often open, sometimes ambiguous):**
- `we will need` / `still need` / `waiting on`
- `pending email` / `pending photos` / `pending trust` / `pending lien`
- `can we update` / `can we change` / `can we make` / `can we trigger`
- `any chance we can push` / `any room for` / `fee exception`
- `please follow up` (imperative — descriptive "this was the follow up email" doesn't count)
- `blocker` / `blocked` / `stuck`
- `please request` / `asap`

**Phrases to NOT trust as open signals (substring false positives):**
- `"we need"` alone — matches "we need to celebrate when this closes"
- `"follow up"` alone — matches "this was the follow up email"
- `"thank you"` — frequently means OPEN ("thank you for sending the IRUC")

## Resolved signal phrases

**Reliable:**
- `iruc!` / `flipped iruc` (BBYS stage-flip celebration)
- `loan funded` (terminal state at CTF)
- `this has been approved`
- `trust approved`
- `backup offer has been signed`
- `prelim funding requested`
- `all set ` / ` all set.` / `all set!` (word-boundary variants — bare "all set" matches "not all set")
- `good to go` / `green light`
- `answered questions`

**Phrases to NOT trust as resolved:**
- `:raised_hands:` alone — fires on any celebratory emoji unrelated to deal status
- `"thank you"` — see above; often introduces an open ask

## Per-stage tone shift

- **New (998755441):** open signals cluster around intake gaps — "needs to apply for Equity Boost", "IR state unknown, ask LO to confirm eligibility", LVMs, "pending email for [client]", missing photos / trust docs / lien balance. Lots of "first-time LO — resources included in email."
- **In Review (998755442):** data-gathering + rush requests — "tasking for a rush with 2020 listing photos", "can we update the target EU request", "LO is confirming home condition", missing-photo detections (kitchen / living_room / bedroom / exterior most common).
- **Approved (998755443):** "2 business days and the Loan Officer has not reviewed the agreement" + "please send out the docusign" dominate. Negotiation enters: "any room for EB?", "can we make a fee exception?", "max EU we can do is $X".
- **Agreement Signed (998755444):** handoff vocabulary — "BUO has been signed", "trust review", "CA Notes:" template, "soft pulls uploaded". Open: "Inspection has not been scheduled yet please follow up".
- **IR In Escrow / IRUC (998755445):** Patricia's `:extreme-teamwork: *Pending:*` checklist is the canonical artifact. Heavy doc-chase — HOI, mortgage statement, payoffs, wire instructions. Notary scheduling. "We have to get this going ASAP to close on time" appears repeatedly.
- **Clear to Fund (998755446):** funding-release requests — "please release wire to IR title for [date]", "request to release funds today", "Additional Document Request — Incoming Residence Proof of Clear to Close". Snapdocs webhooks (in_progress / closed) dominate volume, almost all skippable. Terminal: "Loan Funded: [loan #]".
- **IR Closed / `-irx-` (post-funding ops loop):** added 2026-05-14 round 2 (this stage was MISSED in round 1's 30-channel sample). Channel suffix `-ctc-` → `-irx-` on `Stage UPDATED to Ir Closed`. Three post-close sub-loops dominate:
  - **Maintenance Reserve (MR) disbursement** — human `Maintenance Reserve Request <@<analyst>> client requesting $X in MR / This is for: <list>` → analyst response `MR disbursement of $X approved. Taxes are escrowed in 1st mortgage. There will be $Y left in MR after this disbursement.` Disbursements can fire multiple times across the term-life of the file.
  - **Extension papers** — when DR is taking longer than expected: `Please prepare extension papers N days at X% expiring <date>` → signed amendments posted. Tied to BUO timing.
  - **Recording-rejection loop** — `Recording Status for <loan#> (lead_id: ...): REJECTED` → human investigation → `Recording Status: FINALIZING → RECORDED` resolution. Some state/county combos (CA, parts of AL/MD) trigger `Specialized Manual recording State/County. Please follow manual process.` instead of the automated rec flow.
  - HubSpot mirrors `Loan Funded` date + `IR Closed Date` but NONE of the MR / extension / recording-rejection signals are captured server-side. These are pure-Slack ops state.
- **Failed / `-term-` (terminal):** added 2026-05-14 round 2. Channel suffix `-<any-stage>-` → `-term-`. Terminal flip can arrive from ANY stage including same-day `new→term` (duplicate-lead, intake-fail). Stage text in `HomeLight Support` bot is `Stage UPDATED to Failed` (literal "Failed", not "term"). Cancel reasons cluster: client-route-change, borrower-canceling-bridge, death/hardship, duplicate-lead, DR-extension-granted-elsewhere, fee resistance, structural ineligibility (reverse-1031, multi-parcel, investment property). Channels occasionally `archive → unarchive` weeks later when a client re-engages.

## Bot / automation accounts in deal channels

**High-volume / low-relevance (good candidates for deprioritization):**
- `HomeLight Sales App` (U054856DP6W) — ~70% of all channel messages in IRUC+ stages. Stage changes, email-sent receipts, channel renames, TitleBot pulls, solar detection, missing-photo alerts, DocuSign view/sign events, Snapdocs webhooks, Inspectify status, agreement generations.
- `HomeLight Homie` — LO Client Welcome Onboarding receipts. **Also runs the Winback FUP automation series:** `[HL Lenders] FUP #1 / #2 / #3 | BBYS Winback | EUA received follow-up` — fires at increasing intervals after EUA (Equity Unlock Approval) is sent to client without engagement. Each message explicitly says `"Hold off on manual follow-up for ~24h to avoid double-tapping."` Treat the FUP # as the urgency signal.
- `Easy Automations Service Account` (U04QDD4EVL7) — auto-joins channels only, never posts content.
- `Zapier - Post-Contract Email Bot` (U014H9S8E7J) — Builder doc-collection emails, Notary Signing requests, closing-detail submissions.
- `Closing Signing Request Form` — Notary scheduling notifications, prelim funding details. Also posts the wire-confirmation block with `Amount to IR Title`, `IMAD to IR Title`, `Disbursed Date` after CTC wire release.
- `Contingency Removal Bot` — fires `*Contingency Removal Only Next Steps Email sent!*` blocks at IRUC for CRO files. CRO files often have `EU = $0` and skip the EU loan signing step (DTI Drop variant especially).
- `Slackbot` — OOO auto-replies.

**Mixed (bot identity but real signal — DO NOT blanket-deprioritize):**
- `:bell: HELOC Lead Notification`
- `:rotating_light: Deal is already Under Contract on Incoming Residence`
- `:alert: NEEDS TO APPLY FOR EQUITY BOOST`
- `INVALID_RECIPIENT_ID` failures
- `Inspection has not been scheduled yet please follow up` reminders
- `Additional Document Request` sent (especially the `Incoming Residence Proof of Clear to Close` variant — recurrence is the urgency signal)
- AI IR Contract review verdicts — `Fully Executed?: Yes` AND `Fully Executed?: No` are both high-signal; the verdict pair is the canonical IR-contract gate (added 2026-05-14, 11/28 channels)
- `*Auto Soft Credit Check Results*` (`Yellow` / `Failed` results trigger manual review; `Green` is self-completing but still meaningful as a passed-gate signal)
- `Trust Decision: *Approved* / *Declined*` posts from the trust review sub-team — pure-Slack record, no HubSpot mirror

**Channel-creation noise (always skippable):**
- "X has joined the channel" — `Hugh Rodman`, `Jarrod`, `JL Evangelista`, `Karly Trota`, `Nick Plamondon` all appear as default joiners on every new channel.

## Surprises worth knowing

- **"jump" / "you can hop!" / "Mahalo!"** — Lender Ops jargon for "this is yours to take, swap me out of the channel." Internal handoff, not client-facing action.
- **"Growth Lord's Royal Decree"** — actual rule name for the UC-on-IR-at-intake alert. Real, not dev humor. **The channel will already have IR COE in the lead-submission block when this fires** — it's the canonical "we're already racing" intake signal.
- **Bounce-renames** — channels occasionally rename stage→prev-stage→stage within 2 hours (observed: Sweeney `appr → rev → appr → as`). Don't trust the current channel-name string; trust the most recent `HomeLight Sales App: has renamed the channel from X to Y` message or (better) `HomeLight Support: Stage UPDATED to <name>`.
- **`:fire:` and `:cowboy-eyes:`** — weak signal emojis. `:fire:` ≈ deal moving fast; `:cowboy-eyes:` ≈ Texas file. Both anecdotal; don't build alerts off them.
- **Color codes from soft credit** — `Intelligence: Yellow EX` = manual review trigger; `Green TU/EQ/EX` = auto-pass. The bureau letters following the color carry no signal on their own (just indicates which bureau was pulled).
- **"PC received, just need bbysa signed to start next steps"** — "PC" here = "phone call", NOT "purchase contract."
- **`:rocket:` emoji** — sometimes used by Lender Ops to mark a deal moving through stages aggressively. Weak signal at best.
- **Bot account `HomeLight Support`** — bot identity reused by ops to bulk-update; appears as the "Dispositioned By" actor in BBYS Lead Notifications. Treat as bot for volume reasons.

## Action-requested → completion pairs

> Added 2026-05-13 from the AI Draft v1.5.1 work (see [[2026-05-12-deal-context-ai-draft]]). These pairs are what the AI Draft now uses to extract OUTSTANDING ACTION ITEMS for the bullet-list opener variant.

The core BBYS pattern: someone (LSM, LRM, ops, client advocate) asks a counterparty (LO, agent, client) for a specific artifact. The thread is "open" until a completing message lands. The AI Draft pulls the still-open asks and converts them to bullet points the rep can paste into a check-in email.

Pairs we've validated in real channels:

| Action requested (open) | Completion signal (closes the pair) |
|--------------------------|-------------------------------------|
| `please send the BBYSA` / `awaiting BBYSA` | `signed final BBYSA` / `BBYSA executed` / DocuSign `:white_check_mark:` on BBYSA envelope |
| `please send the docusign` / `send out the docusign` | `:lower_left_ballpoint_pen:` followed by `:white_check_mark:` on same envelope |
| `please upload soft pulls` / `pending soft pulls` | `soft pulls uploaded` |
| `pending trust docs` / `need trust docs` | `trust approved` / `trust review complete` |
| `pending lien balance` / `need payoff` | `payoff received` / `payoff uploaded` |
| `pending HOI` / `HOI update needed` | `HOI Update Received` / `HOI Cleared Date` set |
| `Inspection has not been scheduled yet please follow up` | inspection appointment confirmation / Inspectify status update |
| `needs to apply for Equity Boost` / `please request EB` | `EB approved` / `EB confirmed` |
| `request to release funds` / `please release wire` | `Loan Funded:` terminal message |
| `LVM` / `left voicemail` / `called and texted` | client / LO reply in thread |
| `please correct [X]` / `please fix [X]` | a re-send of the corrected artifact (NOT a "ty" — that's an ack, not a completion) |
| **HomeLight Sales App escalating-alert:** `Its been N business days and the Loan Officer has not [sent the agreement \| reviewed the agreement \| sent the AS \| sent the program agreement \| sent the BBYSA]` | a later message confirming the artifact was sent, OR a stage flip to `Agreement Signed` / `IRUC` / later. **Cadence:** 2-day → 7-day → 14-day (validated round 2 across Cornejo, DeRosa, Keys, Tecson). **Recurrence is a strong signal** — multiple alerts at increasing intervals means the team has been waiting on that exact artifact for that many days. Added 2026-05-14 from the Stephenson channel debrief; validated round 2. |
| **`System: Emailed loan officer (Buy Before You Sell lead): <ask text>`** | a later message from the recipient confirming action on the ask. The ask text inside (e.g., `Need AS asap/send DS?`) IS the action requested, parsed by artifact mentioned. Added 2026-05-14 — these are how the Sales App forwards manual outreach to the team, and the ask text inside is high-signal. |
| **`System: Called loan officer: <call notes>`** | call notes that already describe action items the LO committed to (`going to work on getting photos for this`) — these are partial completions. Treat as "in progress" with the named artifact (photos) still outstanding until uploaded. |
| **Channel-stage-name mismatch:** channel name starts with `appr-` (Approved stage) but no `agreement signed` / `BBYSA executed` / DocuSign `:white_check_mark:` message AND escalating-alert pings about agreement not sent | the agreement (specifically) is outstanding. Reference it by name, never settle for generic "any update?". |
| **AI IR Contract review verdict** (CRITICAL — canonical IR-contract gate; appears in 20/53 channels (38%) combined n=53): literal `<@LRM> <@LenderOps> AI has reviewed the IR Contract:` (always 2 tags) followed by structured fields including `Fully Executed?: Yes/No / Counter Offer Present: Yes/No / Close of Escrow: <date or blank>` | a subsequent AI review on the same channel with `Fully Executed?: Yes`, OR `System: BBYS Request IR Contract Upload` → re-review. **The AI re-runs on contract corrections (often within 30min — sometimes 2-3 verdicts on the same channel for the same upload).** Count distinct channels, not occurrences. **HubSpot mirror:** `bbys_ir_contract_date` populates when fully executed — but the AI verdict is the LEADING signal, often days ahead of the HubSpot flip. |
| **Property Condition repair loop** (Inspectify, 4/28): `Property Condition statuses have been updated: ... Open: * pre funding - required repair - [Roof / HVAC / Well / Mold / Foundation / Pest / Gas Leak]` | same bot reposts with the condition under `Satisfied:` + `New Property Condition document uploaded for X`. **No HubSpot mirror** — lives in Sales App per-condition flags. |
| **Inspectify reschedule chain** (3/28): `*Inspectify order update!* Home Inspection: Reschedule Requested (<@Patricia Pinckard>)` | a later `Home Inspection: Scheduled, <new datetime>`. **No HubSpot mirror** — Sales App field only; HubSpot shows the final scheduled-event without the reschedule chain. |
| **CTC Additional Document Request loop** (4/5 CTC channels; validated 6/8 combined n=53): `BBYS Lender - Additional Document Request sent to <LO email>` with `**Documents Requested**: - <one of: Incoming Residence Proof of Clear to Close / HOI Declaration / Primary Client 1003 / Primary Client Credit Report / Miscellaneous>` — fires repeatedly on the same channel. **Requested-doc set is BROADER than just "Proof of CTC"** (round 2 confirmation): the ADR template is reused for any pre-CTF document gap. | `Stage UPDATED to Clear To Fund` flip OR `new documents have been uploaded to the google drive`. **Recurrence is the urgency signal** — 1 request = normal; 3+ requests + no flip = stuck. HubSpot mirror: `dealstage` → CTC (998755446). |
| **EU loan signing trigger** (4/5 CTC + 1 IRUC; validated 7/8 combined n=53): human `please trigger SA task` / `Please trigger EU` / `Can you please trigger the EU loan signing email` directed at the file processor | bot completion sequence: `Trigger equity unlock document signing Task completed by <name>.` → `EU doc signing task completed. Notary Scheduling link: ...` → `BBYS LO - Schedule Closings Signing email sent` → optional `Send EUIDs Task completed` on the same chain. **No HubSpot mirror** — intermediate gate before CTF. Candidate for a new tracked timestamp. |
| **Backup Offer (BUO) 3-step review/sign chain** (4/5 CTC + 1 IRUC; validated 8/9 combined n=53): `Task Alert: Review Backup Offer for <Client>` (List Ops) → `Send Approved Backup Offer` task → `Final BBYSA and backup contract out for clients signature` | `Client signed the final BBYSA and DR Back up RPA. Files saved in SA and Gdrive` (human) + `Departing Residence Backup Contract Signed Date updated to: <date>` + `Final Agreement Signed Date updated to: <date>`. **HubSpot mirror:** BUO-signed + final-agreement-signed timestamps. **Denied branch (round 2 confirmation):** `BUO Reviewed and approved` / `BUO Reviewed and denied` is the explicit two-variant outcome; denied → `Update and Send Backup Offer` re-do loop. |
| **Wire release request / wire confirmation** (4/5 CTC; validated 6/7 combined n=53): human `please release wire to IR title for <date>` / `Funding to IR title requested for <date>` / `Prelim funding requested for <date>` | `BBYS Lender - CTC Wire Confirmation email sent` + `Closing Signing Request Form` post with literal fields: `Amount to IR Title: <amount>` / `Second Lien Payoff Amount: <amount>` / `IMAD to IR Title: <code>` / `Disbursed Date: <date>`. **HubSpot mirror:** disburse-to-IR-title timestamp. The IMAD code is the canonical wire-confirmation receipt. **Wire-amount correction loop also seen** (round 2): "loan docs are incorrect - the wire amount should be an even 90k" → re-disbursement with new IMAD. |
| **Trust review decision pair** (1/5 AS but extremely high-signal — supersedes generic "trust approved" entry): `<!subteam^...> Stage updated to agreement signed please perform HOA and title review` triggers a Trust reviewer's structured post | `Trust Decision: *Approved* / Trust Type: *Revocable* / Trust Executed: *Yes* / Trustees: ... / Trust Vesting: ...`. **No HubSpot mirror** — pure-Slack signal. For trust-eligible deals the Slack record is the only structured trust record. |
| **Equity Boost submission/approval lifecycle** (4/28, separate from older "EB approved" entry): `the application for equity boost has been submitted!` (Sales App, structured block with target EB amount + asset documents) | `Equity Boost has been approved. EU: $X LPV: $Y HC: $Z Credit Score: ... Max Equity Boost: $...`. Sometimes carries the disclaimer `EB approved prior to approval team review. EB approval solely based on assets submitted (does not account for CLTV).` — don't treat EB-approved alone as terminal; file may get re-evaluated. **HubSpot mirror:** `approved_equity_boost_amount` (per `reference_hubspot_property_gotchas` — note the `Approved Equity Unlock` typo; this is a memory tag, not a vault doc — see the dangling-link note in `[[hubspot]]`). |
| **Reverse-1031 / Special-structure DOA pattern** (2/5 New): pre-approval call/email surfaces a structural ineligibility (reverse 1031, multi-parcel, lot+home, investment property) | `System: Emailed loan officer ... E-mailed loan officer that this one is truly DOA` from HomeLight Sales App, OR stage flip to terminal. **HubSpot mirror:** `dealstage` → closed-lost (998755...term). |
| **IR Contract terminated / shopping-again** (2/5 Approved): LO email update `Unfortunately they decided not to pursue this purchase. Lets keep the application active for a bit.` OR `IR contract is terminated` | new contract upload (`Requested IR Contract Upload from LoanOfficer X` → AI contract review re-fires with new property address) OR terminal stage. **HubSpot signal:** `bbys_ir_contract_date` reset to null OR stale; channel re-enters escalating-alert loop. |
| **Pricing exception pair** (variable / flat-fee / pilot, 3/28): human `<@Jake Vogel> Can we get approval for variable pricing on this one?` / `<@Jake> any chance to get flat fee of 2%` | `<@LRM> <@LenderOps> Variable Pricing Approved` / `2% Fee Exception Approved.` from Jake Vogel. **No HubSpot mirror** — pure Slack. Candidate for a `pricing_exception_type` field. |
| **Coming-soon / not-yet-listed escalation** (3/28 New→Review transitions): human `Listing coming soon <date> - no photos as of yet (just N photos of exterior)` + `please review and task for approval` when listed | Sales App `This property is now listed, please review and task for approval.` → `Stage UPDATED to In Review`. **HubSpot mirror:** `dr_active_on_mls` flips true; `bbys_approved_date` set. |
| **Vested-owner-omitted-from-submission gap** (3/28): bot `<NAME> is/are vested on the departing residence but at least one was omitted from lead submission. Confirm contact information for this/these people!` | human ack `Email for <SecondaryClient>` / `added 2nd title holder to SA, asked LO for contact info` followed by channel topic update reflecting both names OR a `Name 2` populated in resubmission. **Mostly NOT mirrored in HubSpot** — pure-Slack signal for the secondary client. |
| **Soft credit check pair** (all 5 AS+IRUC channels): self-firing `<@LenderOps> :credit_card: *Auto Soft Credit Check Results* Primary Client: <Name> Intelligence: <Green/Yellow> <TU/EQ/EX> Result: <Passed/Failed>` | self-completing for `Green` results. `Yellow` or `Failed` triggers a follow-up `Uploaded soft pull` from Lender Ops manually pulling. **No HubSpot mirror.** Yellow = manual review trigger. |
| **Builder doc-collection / NHC handoff** (builder-specific, 2/5 IRUC for Lennar/K-Hov): `Zapier - Post-Contract Email Bot: *Builder Doc Collection and IR Closing Detail Confirmation request emails sent separately to client and builder. Emails Titled: "BBYS: Under Contract! Next Steps - <Client> - <Address>"` | `Zapier - Post-Contract Email Bot: *BBYS Loan Documents have been uploaded by the client.* Submission Notes: Please begin file set up if you haven't already.` (with gdrive link). **HubSpot mirror:** `bbys_ir_contract_date` + IR-closing-detail field. |
| **Contingency Removal Only (CRO) handoff** (2/5 IRUC): `*Contingency Removal Only Next Steps Email sent!* LO/LOA Email: <emails> Email Title: "BBYS: Under Contract Action Needed - <Client>"` | subsequent IR-contract upload + `Stage UPDATED to Ir Contract`. CRO files often have `EU = $0` and skip the EU loan signing step (DTI Drop variant especially). **HubSpot mirror:** likely a `contingency_removal_only` boolean; appears in CA Notes. |
| **Cheryl Funk's `*UC Notes*` checklist** (2/5 IRUC, sibling to Patricia's `:extreme-teamwork:`): manual block posted at IRUC by Cheryl: `*UC Notes - LO DR Val: $X* ... <date>: Pending: -Inspection - *date* -Final BBYSA + RPA - not sent -schedule MOA Closing est: -CTC?` | items get crossed off in subsequent posts (often by Cheryl editing/reposting the block). **Maps to multiple HubSpot milestones simultaneously** (inspection, final BBYSA, MOA, CTF). |
| **Snapdocs e-sign / RON ceremony failure** (2/5 CTC): Snapdocs `Closing comment created` event posting human-readable failures like `Was not able to log into the meeting on May 6th at 6PM.` / `To Join the meeting, it was greyed out and was not able to get anything notarized.` | human request `<@Derek><@LenderOps> Hey can someone help me extend closing through today for RON and resend an invitation please?` → `Snapdocs webhook ... signing_complete`. **No HubSpot mirror** — Sales App + Snapdocs only. |
| **Wire-split / overage-limit** (1/5 CTC, rare but high-stakes): human `LO asked if we can split the wire? I guess title only wants us to send $X and the rest $Y wired to the client directly. The overage limit on this one is $Z so we should be good.` | `Loan Funded: <loan #>` + `Closing Signing Request Form ... Amount to IR Title: <split-amount>` reflecting the smaller wire. **HubSpot mirror:** Loan-funded date + amount-disbursed-to-IR-title; doesn't capture the split. |
| **Terminal flip from any prior stage** (9/9 terminal channels in round 2): human `Please fail this file <@LenderOps>` OR client message `Please cancel this application` OR LO message `Borrower is canceling the bridge loan` | `Stage UPDATED to Failed` + channel rename `*-<prev>-*` → `*-term-*`. **Terminal can arrive from ANY stage** — observed `iruc→term`, `as→term`, `appr→term`, AND same-day `new→term` (duplicate-lead). HubSpot mirror: `dealstage` → closed-lost. Added 2026-05-14 round 2 (n=25). |
| **Death / hardship cancel** (1/9 terminal, edge case): client/family message `Due to the untimely passing of <client>, we are unable to fulfill the contract. Therefore, we must cancel it.` | `Stage UPDATED to Failed`; `DocuSign envelope voided`; channel archived. **No HubSpot field for cancel-reason** — pure-Slack record. Distinct from generic cancel because it carries finality and may need legal-team awareness. |
| **Archive → unarchive revival** (1/25 round 2 — Tecson): Slack `<user> archived the channel` followed days later by `<user> unarchived the channel` + new client message reviving the file | channel re-enters active flow; new agreement may be regenerated. **Slack-only signal.** Worth treating archive-then-unarchive as a re-engagement opportunity, not a dead deal. |
| **Section flags block** (NEW lead-intake structured ask matrix, 3/5 fresh New): bot block `BBYS LO Lead Submission Confirmation Next Steps email accepted/queued by CommunicationsService / LO recipient: <email> / Communication request ID: <id> / Section flags: <12 booleans incl. upload_departing_home_photos / request_listing_agent_contact_information / request_titleholder_vested_owner_email / request_lien_balance_confirmation / request_solar_documentation / request_trust_documentation / first_time_lo_resources>` | each individual flag's underlying ask gets completed (photos uploaded, agent info confirmed, etc.). **Slack-only — high-signal structured machine-readable.** Parse the flag list to know what's open at intake. |
| **First-time-LO signal** (4/25 round 2, recurring across stages): within Section flags block: `first_time_lo_resources: true` OR human CA Notes line `First time LO ...` / `Situation: First submittal and to go IRUC` | LO completes first deal (channel reaches `-iruc-` or later). **Slack-only** at first-deal lifecycle; could mirror to lender record. Indicates "extra handholding" mode. |
| **180-day program-period pilot** (1/25 round 2 — Austin, validates Sweeney from round 1): richie-sf message `LO needs 180 days to make this one happen for the borrower - the LO understands there will probably be a 2 month holdback on the mortgage payment` | (none — pilot terms baked into final agreement, no completion event). **Slack-only.** Any feature reading HubSpot alone will report standard terms and be wrong. |
| **Duplicate lead detection** (1/25 round 2 — Kelly): in `*New BBYS lead!*` block: `Possible Duplicate Lead. This DR address has been previously submitted: <link to other lead>` | human ack `this is a dupe that <X> and I are working` → `Go ahead and fail this one out!` → `Stage UPDATED to Failed` on the duplicate. Resolves intake-side; the live deal continues on the other channel. |
| **EU-negative analyst voice** (Brian Karch pattern, 2/25 round 2): `<@LRM> <@LRM2> <@LenderOps> the EU result for this <house/property> is negative, at -$Xk. There is room in the HC CLTV for EB to help out if they seek EU. Otherwise, if they want DTI drop instead we likely would be able to push the EU to $0 instead.` | EB application submitted → EB approved, OR client switches to DTI Drop, OR file fails (CLTV too high). **Slack-only structural analysis** — value drops to HubSpot only after approval/decision. |
| **`-irx-` IR Closed post-close ops loop** (2/25 round 2; MISSED in round 1): `Stage UPDATED to Ir Closed` (channel rename `-ctc-` → `-irx-`) triggers post-close events: `Maintenance Reserve Request <@<analyst>> client requesting $X in MR / This is for: <list>`; `Please prepare extension papers N days at X% expiring <date>`; `Recording Status for <loan#>: REJECTED` | MR: `MR disbursement of $X approved`; extension: signed amendments posted; recording: `Recording Status: FINALIZING → RECORDED`. **Partially mirrored** (Loan-funded date + IR Closed Date exist); MR / extension / recording-rejection are Slack-only. See per-stage tone-shift section below for `-irx-`. |
| **Solar CA-Notes variant** (Round 1 wrong; correction): CA Notes line is NOT just a boolean. Accepts `Solar: No` OR `Solar: Yes - Lease already provided` OR `Solar: Yes - Owned`. Sales App `Solar Panel Detection / Result: Yes / Confidence: %` reconciles with CA Notes declaration. | Solar docs in deal folder. **Slack carries the lease/owned distinction in CA Notes** — HubSpot probably has only the boolean. Correction to round-1's "Slack carries only boolean" claim. Confirmed 1/25 explicit (Dejesus-Patricio) + 4 implicit. |

**Pair-detection rules used by the AI Draft:**
- If an open ask has a later message in the same thread or within 48h that matches the completion signal, drop it from outstanding items.
- A bot ack (`:eyes:` on DocuSign, "viewed") does NOT close a pair. Only `:lower_left_ballpoint_pen:` + `:white_check_mark:` together close DocuSign asks.
- Patricia's `:extreme-teamwork: *Pending:*` checklist AND Cheryl Funk's `*UC Notes*` block both take precedence over scanning loose messages — when either exists, prefer parsing it.
- Asks older than 14 days that are still open get a `(stale)` tag — usually they were quietly resolved in DM or never followed up.
- **Recurrence is a first-class signal.** The same alert / ADR / escalation firing multiple times (7-day → 14-day, single ADR → 3rd ADR, etc.) means the team has been waiting that long for that exact artifact. Treat repetition as urgency, not as duplicate noise.

## Slack-as-source-of-truth — patterns NOT mirrored in HubSpot

> Added 2026-05-14 from the n=28 extension sample. These are channel-only signals — if a BBYS AI feature reads exclusively from HubSpot, it will silently miss these. For each, the channel IS the canonical record.

The full BBYS deal state is **not** captured in HubSpot. Several high-signal patterns live exclusively in Slack channel content. Any future feature (deal summarization, stale-thread detection, anomaly detection) that wants to know "what's actually going on with this deal" needs to read both sources.

| Slack-only pattern | What you miss reading HubSpot alone | Suggested mitigation |
|---|---|---|
| **AI IR Contract verdicts** (`Fully Executed?: Yes/No / Counter Offer Present: Yes/No`) | The leading signal that the IR contract is partly executed. `bbys_ir_contract_date` populates only when fully executed — sometimes days after the AI verdict surfaces partial execution. | Read AI verdict messages from channel as the IR-contract gate. Re-verdicts within 30min after a re-upload are normal. |
| **Pricing exceptions** (variable pricing approved, 2% flat-fee exception, 180-day pilot, mortgage holdback) | The actual program terms on the deal. None of these leave a HubSpot trace. Sweeney had a 180-day program-period pilot with 2-month mortgage holdback — all Slack-only. | If summarizing the deal, scan channel for `<@Jake Vogel>` / `Approval Granted` / `Fee Exception Approved` and surface as a "non-standard terms" flag. |
| **Trust review decisions** (trust type, trustees, vesting language) | The trust eligibility outcome and the structured trust attributes. HubSpot deal properties don't capture trust type / executed-status / trustees / vesting. | For trust-eligible deals, the Slack record IS the only structured trust record. A `pure_slack_trust_signals` mirror table would help. |
| **EB approved-pre-team-review disclaimer** | The qualifying note that EB approval was assets-only and doesn't account for CLTV. HubSpot's `approved_equity_boost_amount` reflects the approval but not the conditional disclaimer. | Don't treat EB-approved alone as terminal. Read channel for the disclaimer text; flag for re-evaluation risk. |
| **Inspectify reschedules** | The chain of reschedules behind a current inspection date. HubSpot shows the most recent scheduled-event without the history. | If detecting "inspection-fatigue" deals (3+ reschedules), read Inspectify update messages from channel, not HubSpot. |
| **CTC Additional Document Request recurrence** | The fact that the same `Incoming Residence Proof of Clear to Close` request has fired 3+ times. HubSpot shows the most recent `dealstage` but not the request count. | Count occurrences in channel. Single ADR = normal; 3+ ADRs + no CTF flip = stuck. |
| **EU loan signing trigger** | The intermediate gate between IRUC and CTF — "trigger EU" → notary scheduling → signing complete. No HubSpot property tracks this transition. | Channel-only timestamp. Worth a `eu_loan_signing_triggered_at` HubSpot property if anomaly detection needs the gate. |
| **Snapdocs e-sign / RON failures** | The chain of failed signing attempts. HubSpot shows only the successful `signing_complete` event. | Read Snapdocs `Closing comment created` events from channel for failure history. |
| **Vested-owner gaps** (secondary client omitted from submission) | The "missing co-client" alert and the manual ack/resubmit. HubSpot may have a single primary client only. | Channel topic update after ack reflects both names — use as the canonical co-client record. |
| **2-for-1 condo merge / multi-LO files** | The cross-reference between two related deals being merged into one IR purchase. HubSpot deals are independent records. | Channel CA Notes "This is in combination with <other-address> (<other-client>)" — the only reliable cross-deal link. |
| **Solar lease vs owned distinction** | HubSpot probably has only a `Solar: Yes/No` boolean. CA Notes captures the structural distinction (`Yes - Lease already provided` vs `Yes - Owned`) which materially affects underwriting. | Parse the CA Notes `Solar:` field — read the trailing qualifier when present. Round-1 was wrong here; round-2 confirmed the distinction is recorded. |
| **`-irx-` post-close ops state** (MR / extension / recording-rejection) | The chain of post-funding activity. HubSpot mirrors `Loan Funded` date + `IR Closed Date` but NONE of the maintenance-reserve disbursements, extension requests, recording-rejection loops, or specialized-manual-recording flags are captured server-side. | Read channel for `Maintenance Reserve Request` blocks, `extension papers` asks, and `Recording Status: REJECTED / FINALIZING / RECORDED` events. |
| **Archive → unarchive revival** | A "dead" deal that reopens later when the client re-engages. HubSpot may not surface the unarchive moment cleanly — the deal might just appear active again. | Watch for `<user> unarchived the channel` Slack events on previously-archived `-term-` deals; treat as a re-engagement opportunity. |
| **Section flags block at intake** | The 12-boolean machine-readable matrix of what each new lead needs (photos, agent contact, titleholder email, lien balance, solar docs, trust docs, first-time-LO resources, etc.). HubSpot intake fields don't carry this structured shape. | Parse the Sales App `Section flags: ...` block on `-new-` channels to know what's open from day one. |
| **Cancel-reason taxonomy** | The free-text cancel reason on `-term-` channels (client-route-change, borrower-canceling-bridge, death/hardship, duplicate-lead, DR-extension-granted, fee-resistance, structural-ineligibility). HubSpot just shows `closed-lost`. | Parse the human/client message preceding the `Stage UPDATED to Failed` event; cluster cancel reasons into the taxonomy. |
| **Pilot / exception terms** (180-day pilot, mortgage holdback, variable pricing, 2% flat-fee, 30-day-list-exception) | Any non-standard program terms negotiated for a specific deal. None of these leave a HubSpot trace. | Search channel for `<@Jake Vogel>` / `Approval Granted` / `Fee Exception Approved` / `LO needs N days to make this one happen`. |

**Implication for any future BBYS AI feature:** the canonical "what's happening with this deal" answer requires reading both HubSpot AND the deal's Slack channel. Reading HubSpot alone gives you the milestones; reading the channel alone gives you the in-flight texture and exception state. Combining both gives the full picture.

## What real reps DON'T write — voice study (n=35 outbound emails, 2026-05-13)

> Added 2026-05-13 from the AI Draft v1.5 voice study. Sampled 35 real outbound HomeLight rep emails (LSM + LRM + Lender Ops, mixed stages). The pattern is striking enough to call out explicitly so future AI features don't regress.

**0 of 35 real reps did any of these:**
- Recap deal facts the recipient already knows ("As you know, this is a BBYS file for [client] at [address]…"). Reps assume context.
- Use passive standby filler ("Let me know if you need anything", "happy to help", "looking forward to hearing from you", "as always"). Real reps stop the email at the ask.
- Lead with the client's address or property details. Real reps lead with the ask, an answer, or what they just did.
- Use "I hope this email finds you well" or any variant. Zero instances.
- Sign with title blocks. Just the first name, or nothing.

**What real reps DO write:**
- **DIRECT ANSWER opener** ("Yes — funds will hit Tuesday." / "No, EB isn't approved on this one.") — most common when responding to a specific LO question.
- **REP'S ACTION opener** ("Just sent the updated BBYSA over." / "Pulled the latest payoff — uploading now.") — when reporting what they did.
- **BARE ASK opener** ("Need the signed BBYSA before we can move to IRUC." / "Can you confirm the HOI carrier?") — when the email IS the ask.
- **ACKNOWLEDGEMENT opener** ("Got it — thanks." / "Confirmed.") — short receipts.

**Length pattern:** median real-rep email is 2 sentences. Longest in the sample was 5 sentences. Anything 6+ sentences reads as AI-generated and was a major iteration target throughout v1.2-v1.6.

**Implication for any future BBYS AI feature:** if you're generating outbound copy for a HomeLight rep, the bar is "could the recipient tell this was AI?" — and the failure modes are always the same five things above. Prompt explicitly against them; don't expect the model to figure it out from examples alone.

## Stage-stratified voice study (n=55 outbound emails, 2026-05-14)

> Added 2026-05-14 from the n=55 stage-stratified voice study (vs the earlier generic n=35). Sampled real-rep outbound emails from the `hub_emails` mirror in Supabase, bucketed by lifecycle state. **This study replaced the generic "what reps DON'T write" view with state-specific findings.** Some patterns the original n=35 study identified as universally forbidden turn out to have legitimate per-state exceptions (decline / handoff emails DO recap deal facts).

### Per-state findings

| State | n | Median length | Dominant opener | Notable signature phrasings |
|---|---|---|---|---|
| **A — New** | 9 | 2 sentences | REP'S ACTION 33% / DIRECT ANSWER 22% | "we will need photos", "task this for review", "I am covering for [colleague]", "expedite once they hit our system" |
| **B — Approved/In Review (pre-AS)** | 8 | 5 sentences (skewed by APPROVAL BLOCK) | REP'S ACTION 50% (approval announcements) | APPROVAL BLOCK: 6-line structured artifact (Available Funds / Outstanding Balances / BBYS Loan Amount / 3% Maintenance Reserve) + Next-Steps 4-bullet list |
| **C — Approved + AS-signed (pre-IR-contract)** | 9 | 3 sentences | Even split across 4 patterns | Deborah Shutt bullet chase: "These files await signature only to move forward: A / B / C. Is there any chance they can sign today?" / Brandi math-led: "We are currently sitting at negative $X. If they can pay down their mortgage to $Y, we can still make the file work." |
| **D — IRUC** | 12 | 4 sentences | REP'S ACTION 42% | NEW 5th opener: CLIENT-CALL-NARRATION ("I just got off the phone with [Client]. They are [emotional state] because [situation]"). Stock verb: "move the file along" (Tiffany Traxler). Solar gating opener. Urgency framing: "coming up on the time where they will have to make two mortgage payments." |
| **E — CTC** | 9 | 2 sentences (4/9 single-sentence) | REP'S ACTION 56% | ONE-LINE confirmations: "Our wire was sent earlier today!" / "Consider it done!" / "Thank you!". Most laconic of all states — pad NOTHING. |
| **F — IR Closed `-irx-`** | 8 | 3 sentences (bimodal) | ACKNOWLEDGEMENT 38% | Short acks ("Thank you again for the referral!") OR long technical explainers (BUO mechanic, MR refund, day-121 EB repayment, extension fee). Reps narrow to status / explanation / acknowledgement — almost never chases. |

### Per-state exceptions to "0/35 reps recap deal facts"

The original n=35 voice study found a **blanket** "no deal-fact recap" rule. The n=55 study found **one legitimate per-state exception**: 3/12 IRUC emails DO recap when the email IS a HANDOFF / DECLINE / FIRST-TOUCH-AFTER-REFERRAL. Example: *"Thanks for referring [Client] to us! Unfortunately, we are not able to work with them because their home is manufactured."* Detection criterion: no prior outbound from sender to recipient in the timeline.

### Real-rep closer distribution (corpus-validated)

| Closer shape | Frequency |
|---|---|
| Generic open-door ("Please let me know if you have any questions" / "If you need something else, please let me know!") | **~70%** |
| Concrete next action ("I'll let you know once we hear back" / "Let's chat tomorrow") | ~15% |
| Absent (just signature) | ~15% |

**Implication:** generic open-doors are the dominant real-rep closer. Don't try to avoid them. The "Generic Open-doors are FINE" clause in the AI Draft prompt's ASPIRATIONAL-GUIDANCE CHECK is corpus-validated.

### Methodology notes — drip-campaign and OOO contamination

Of all "outbound rep emails" in the raw sample, ~40% in stages B–F are **HubSpot Sales-Engage drip-campaign templates** sent under the LSM's name, NOT real-rep voice. Subject lines like *"Need Assistance Explaining BBYS?"*, *"Unlock up to 90%…"*, *"Thought you'd find this useful — webinar tomorrow"*, *"Congrats on YOUR first BBYS deal!"*, *"How's your calendar look?"* — all carry `hs-sales-engage.com` or `meetings.hubspot.com` link tokens.

**Recommendation for any voice-study, model-training, or sample-extraction work on outbound rep emails:**
- **Exclude** messages containing `hs-sales-engage.com` or `meetings.hubspot.com` link tokens.
- **Exclude** subject prefixes `OOO`, `SLOW TO RESPOND`, `Out of Office` (auto-replies fire dozens of times per rep PTO).
- The raw outbound-email corpus has these two contaminants dominating volume; voice profile must be downstream of the filter.

### Validated falsification — the "are you shopping?" template that wasn't

PR #195 (2026-05-13 PM) added "we refreshed the approval [N weeks] ago. Is the client actively shopping or under contract yet?" as a canonical State C ask. The n=55 voice study found **0/9 corpus hits** for this phrasing in the State C bucket. Real reps in that state either chase the signature directly (Deborah Shutt's bullet template) or solution around the math (Brandi Cirell's "negative $X" template). PR #196 replaced the unvalidated phrasing with the corpus-validated shapes and explicitly forbade the original.

**Meta-lesson for future state-aware template work:** templates that describe what reps "would say" must be **corpus-validated (n≥5 hits) before merging**. Confidence-driven template-writing produces output that looks plausible to engineers but doesn't match real-rep voice. The user caught this one in a live Manson-channel draft within hours of shipping — fix the authoring workflow, not just the resulting bad templates.

## Sales-positioning voice study — when the corpus says "no" (n=10,357 LRM emails, 2026-05-14 PM)

> Added 2026-05-14 PM. The user requested BBYS sales-positioning language in AI Draft. Per the prior meta-lesson, dispatched a corpus voice study BEFORE adding any positioning rules. The result was a clear negative: **real LRMs don't pitch.**

### Methodology

Sampled 10,357 outbound LRM emails (180-day window) from `hub_emails` in Supabase. Filtered:
- Drip-campaign contamination: `hs-sales-engage.com`, `meetings.hubspot.com`, **and now also `hubs.ly` and `hmlt.co` short-domain link tokens** (a few Kara Kleingarn LSM-prospecting templates slipped past the standard drip filter; they use `hubs.ly` instead — add to any future contamination filter).
- OOO subjects (`OOO`, `SLOW TO RESPOND`, `Out of Office`).
- **Senders filtered to the 6 LRMs** (Ashlee Kim, Deborah Shutt, Kyle Bradish, Brandi Cirell, Ian Pardo, Angelica Espinosa). Including LSMs contaminates the sample with prospecting copy. Senders matter more than subjects.
- `SPLIT_PART(email_body, 'wrote:', 1)` to strip Gmail quoted-reply tails (without this, "non-contingent" hit count was 4,039 from product-name + quoted-conditional-approval template false-positives; with it, 65 real corpus hits).

### Headline finding: 10 of 11 textbook positioning phrases have ZERO hits

| Phrase | Hits in 10,357 LRM emails |
|---|---|
| compete with cash | 0 |
| stronger offer / strongest offer | 0 |
| move once / buy first sell second | 0 |
| peace of mind / dream home | 0 |
| level the playing field / win in this market | 0 |
| guaranteed sale | 1 (likely quoted context) |
| we'll buy the home / we will buy the home | 0 |
| unlock your equity (as positioning verb — product name "Equity Unlock" is fine) | 0 |

**Real LRMs don't pitch. They operate.** Marketing positioning vocabulary lives in the LSM-prospecting / Sales-Engage drip channel — same contamination class the n=55 study filtered out.

### The ONE positioning concept that survives: `non-contingent`

65 corpus hits across all 6 LRMs. Distribution:

| State | Hits | Notes |
|---|---|---|
| State B (Approved-pre-AS) | 46 (71%) | Agreement-signing nudges — "sign so your client can write non-contingent offers" |
| State A (intake) | 8 | Often paired with DTI Drop / EB explainers |
| State A/B DTI-Drop pitch | 11 | Technical mechanic: "non-contingent offer + remove DR liabilities" |
| State D (IRUC) | 1 | Single ad-hoc instance |
| States C / E / F | 0 | Zero hits in AS-signed-pre-contract, CTC, IR Closed |

Three corpus-validated shapes that real LRMs use (the only `non-contingent` phrasings that should ever appear in AI Draft output):

1. **Deborah Shutt template** (13 hits): "...allows your client to write offers non contingent on the sale of their home"
2. **Ashlee Kim / Kyle Bradish template** (24 hits): "...move forward as a non-contingent buyer" (often paired with: "Signing does not obligate the client. It simply keeps them shopping-ready with a non-contingent offer.")
3. **Brandi Cirell conversational** (ad-hoc): "Signing just allows them to shop, and make non-contingent offers."
4. **DTI Drop mechanic** (Brandi, recurring): "...write a non-contingent offer and lets you remove the departing property liabilities."

### Rule shipped to AI Draft prompt (PR #200)

- **Default:** forbid all 10 zero-hit marketing phrases across all states.
- **State B carve-out:** `non-contingent` allowed only in State B agreement-signing-nudge context + DTI-Drop / zero-EU explainers.
- **USER GUIDANCE override:** bounded — if rep explicitly asks for positioning, allow `non-contingent` but still forbid the 10 marketing phrases.
- **FINAL CHECKLIST self-audit:** POSITIONING-VOCABULARY CHECK with State-B verification gate.

### Methodology learning for future voice studies

- **Add `hubs.ly` and `hmlt.co` to the drip-contamination filter.** Standard `hs-sales-engage.com` / `meetings.hubspot.com` filter caught most templated marketing but not Kara Kleingarn's LSM-prospecting templates which use HubSpot short domains.
- **Sender-filter to LRMs (transactional voice) vs LSMs (prospecting voice).** Including LSMs corrupts the LRM voice profile.
- **Strip quoted-reply tails with `SPLIT_PART(body, 'wrote:', 1)`** before phrase-counting. Without this, ~98% of "non-contingent" hits were quoted-reply false-positives (4,039 vs 65 real).
- **Subject-prefix-based state inference** ("Conditional approval for…" = State B, "Upload Incoming Residence Contract…" = State D) converges on the same answer as direct `dealstage` association; useful when the deal join is expensive.

### Meta-meta-lesson catalogued

The v1.12 corpus-validation gate worked. The user's "add positioning" instinct could have produced confidently-wrong rules in the prompt. The sub-agent voice study produced **a clear, evidence-backed NO with a narrow YES carve-out**. Future "let's add X to the prompt" instincts follow this exact gate: sub-agent voice study → corpus result → ship corpus-validated rule (which may be "don't add it") → test for the rule's presence + the explicit reasoning anchor.

## Bot messages are CONTENT, not noise — fetch-layer rule

> Added 2026-05-13 PM after the Pomeroy bug. See decision log [[2026-05-13-slack-bot-messages-are-content-not-noise]].

The "Bot / automation accounts" section above is the relevance-scoring layer view. At the FETCH layer, bot messages are higher-signal than 90% of human messages on a fresh BBYS deal channel. **Never blanket-filter Slack messages by `subtype` at fetch.**

### What bot messages actually carry (verified on real BBYS channels)

| Bot | High-signal content it posts (verbatim or near-verbatim) |
|-----|---------------------------------------------------------|
| **HomeLight Sales App** | `Submission confirmation email sent to LO — <email>` followed by `:clipboard: Required from LO:` and a bullet list (departing home photos, titleholder/vested owner email, lien balance, etc.). `System: Called loan officer (Buy Before You Sell lead):` followed by call notes. `Stage X → Stage Y` flips. DocuSign view/sign events. Inspectify status. |
| **TitleBot** | Title-pull dumps: `Owner Names`, `Vesting Owner`, `Vesting Right`, `Legal Description`, `Year Built`. Flags like `IR state unknown, ask LO to confirm eligibility`. |
| **Snapdocs / Notary bots** | Notary scheduling, prelim funding details, signing complete events. |
| **Zapier - Post-Contract Email Bot** | Builder doc-collection emails, closing-detail submissions. |
| **BBYS Lender email automations** | "Incoming Residence Questionnaire email sent to <email>", "Order Inspection email sent", etc. — these are the canonical ACTION-REQUESTED events the AI Draft's bullet-list opener detects pairs for. |

### Where the content actually lives in the Slack message payload

Bot messages frequently put the title in `text` and the substantive payload in `attachments[].text` or modern `blocks[].text.text` / `blocks[].elements[].text`. Walking `text` alone misses the real content.

Required walk order: `text → blocks (section + rich_text elements + fields) → attachments (pretext / title / text / nested blocks / fallback)`. Dedupe consecutive identical chunks (some bots repeat the title verbatim in both `text` and the first block).

### Sender display for bot messages

`users.info` won't have an entry for bot user IDs. Fall back to `bot_profile.name` so messages render as "HomeLight Sales App" / "TitleBot" instead of a bare `bot_id`. Critical for the AI Draft to apply the INTERNAL-NAME SUPPRESSION rule correctly — without a readable bot name, the model can't recognize internal-shorthand names in user-provided guidance.

### Noise-subtypes allowlist (drop these at fetch — these have no deal content)

`channel_join`, `channel_leave`, `channel_topic`, `channel_purpose`, `channel_name`, `channel_archive`, `channel_unarchive`, `pinned_item`, `unpinned_item`, `group_join`, `group_leave`, `tombstone`. Everything else (including all `bot_message` variants) is content — preserve at fetch, dampen at scoring if needed.

## Update protocol

When BBYS vocabulary shifts (new ops conventions, new bot accounts, new stage names), update this file. Last full sample: 2026-05-12 (30 channels, 5 per stage across the 6 active stages). Voice study sample: 2026-05-13 (n=35 outbound rep emails). Bot-content rule added: 2026-05-13 PM (Pomeroy debrief).

---

## 2026-08-28 — naming pattern update

Observed across live channels during the Slack sweep. Wider than previously recorded:

```
#[<partner>-]<stage-code>-<client-lastname>-<address>-<state>-<tier>
```

**The partner prefix is optional** — `#appr-cooley-11-cuerda-way-ar-express`,
`#iruc-harrison-2508-red-draw-rd-tx-express`, `#new-strauss-26-bentley-rd-ca-express` all
omit it.

**Partner prefixes seen:** `uwm` · `tls` · `fairway` · `nfm-lending` · `merit-lending` ·
`cross-country` · `ahmc` · `go-rascal` · `bay-equity` · `equitysmart`

**Stage codes:** `new` · `appr` (approval) · `iruc` (IR under contract) · `drx` · `irx` ·
`as` · `term`

**Tiers:** `express` · `light` · `rapid`

> 🔑 **The stage code is not static** — HAPI's `update_slack_channel_name_worker` **renames
> the channel as the deal progresses**. A channel that was `#…-appr-…` becomes `#…-iruc-…`.
> Any archived reference to a channel name may no longer resolve, and **channel-name-based
> stage inference reflects the current stage, not the stage at the time of the message.**

See [[bbys-integration-map]] for the full Slack↔HAPI write path.
