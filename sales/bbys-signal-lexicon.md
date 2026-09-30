---
type: project
last_updated: 2026-05-15
related: [[bbys-deal-channel-vocabulary]] [[bbys-stage-progression]] [[bbys-overview]] [[bbys-edge-cases]] [[2026-05-12-deal-context-ai-draft]]
source: HomeLight-Vault/projects/2026-05-15-bbys-signal-lexicon.md
imported: 2026-09-29
---

# BBYS Signal Lexicon — Slack-Grounded, Direction-Resolved

> Purpose: ground-truth reference for the AI-draft prompt's BBYS ARTIFACT DIRECTION TABLE + WINBACK FUP SIGNAL rules, and feed the deterministic `signalLexicon.ts` parser sketched at `src/hubspot-email/signalLexicon.ts`. Bug that triggered this work: LLM kept inverting EUA direction ("can you confirm the Equity Unlock approval?" asked TO the LO, who is the recipient, not the approver).
>
> This doc **builds on** [[bbys-deal-channel-vocabulary]] (n=53 channels, the existing lexicon) and [[bbys-stage-progression]] (per-stage corpus). Do not duplicate either — read those first; this doc adds the direction layer and the deterministic-parsing layer.

## Methodology

- **Date range:** all corpus evidence cited below is from messages dated 2026-04-26 through 2026-05-15.
- **Sample size:** existing vault corpus n=53 channels (full-content sampling), plus new targeted sampling for this work: 20 Winback FUP messages (verbatim Slack search), 6 full-channel reads spanning all active stages (`-new-`, `-rev-`, `-appr-`, `-as-`, `-iruc-`, `-ctc-`, `-irx-`, `-term-`), plus prefix verification searches. **No re-sampling of 210 channels** — the existing n=53 already validates every high-frequency pattern; the marginal value of a new 30-per-stage sample was lower than building the parser sketch.
- **Channel prefix conventions** (verified by direct search on 2026-05-15):
  - `-new-` (NEW / 998755441) — lead just submitted.
  - `-rev-` (IN_REVIEW / 998755442).
  - `-appr-` (APPROVED / 998755443) — also where Winback FUP fires.
  - `-as-` (AGREEMENT_SIGNED / 998755444).
  - `-iruc-` (IR_IN_ESCROW / 998755445) — bot stage label says "Ir Contract".
  - `-ctc-` (CLEAR_TO_FUND / 998755446).
  - `-irx-` (IR_CLOSED / 998755447) — post-funding ops loop.
  - `-drx-` (DR_CLOSED / 998815822) — terminal-success.
  - `-term-` (closed-lost). Can arrive from ANY prior stage; reached even from same-day `-new-` (duplicate-lead intake fail).
- **Lender-prefix conventions:** `tls-*` (TLS / The Loan Store dominant in 2026 H1), `orchard-*`, `fairway-*`, `cross-country-*`, `uwm-*`, `mig-*`, `rwm-*`, `apm-*`, `dr-horton-*`, `first-continental-*`, `luminate-*`, plus a no-prefix variant (`appr-*` / `irx-*`) for older channels. **Lender prefix is decorative for stage parsing** — only the stage token between hyphens matters.

---

## 1. Artifact Direction Master Table

> **The canonical rule.** Every BBYS artifact has a fixed direction. Asking the wrong party to confirm/approve/grant an artifact reads as confused or as if the email was written by the wrong side. This table is the source of truth for the prompt's ARTIFACT DIRECTION TABLE block. Updated 2026-05-15 from corpus.

Direction notation: **FROM → TO**. "Who can confirm/approve" is the authoritative party — asking anyone else about that artifact reads as inverted.

### 1a. HomeLight-issued artifacts (LO is recipient, NOT approver)

| Artifact | FROM | TO | Who can confirm / approve / grant | Corpus signal that issuance occurred |
|---|---|---|---|---|
| **EUA / Equity Unlock Approval / "the approval"** | HomeLight underwriting | client + LO (CC) | HomeLight underwriting | `Stage UPDATED to Approved` + `BBYS conditional agreement generated` + `Email sent: BBYS_initial_approval_notification_to_loan_officer` |
| **Approval amount / Available Funds / 3% Maintenance Reserve / BBYS Loan Amount** | HomeLight underwriting | LO | HomeLight underwriting | APPROVAL BLOCK in LRM email to LO; `Equity Unlock Range: $X - $Y` in lead-submission block |
| **APPROVAL BLOCK** (6-line: Available Funds / Outstanding Balances / BBYS Loan Amount / 3% MR / Next Steps) | HomeLight LRM | LO | HomeLight LRM | LRM outbound email at approval (corpus: State B template); not a bot artifact |
| **BBYSA / program agreement / "the agreement" / "the DocuSign"** | HomeLight (DocuSign envelope) | client (signs); LO (forwards) | client signs; LO forwards only | `Agreement was sent to client(s) for signature (DocuSign).` → `:eyes:` → `:lower_left_ballpoint_pen:` → `:white_check_mark: All clients completed signing the program agreement` |
| **EUID / Equity Unlock Initial Disclosure / EU loan signing docs** | HomeLight | client (via notary) | HomeLight ops | `EU doc signing task completed.` → `BBYS LO - Schedule Closings Signing email sent` |
| **Notary signing / closing signing** | HomeLight (scheduling link) | client | HomeLight scheduling | `Buy Before You Sell Notary Scheduling link: https://equity.homelight.com/bbys/document-signing/...` |
| **Inspection / Inspectify order** | HomeLight ops (post-IR-contract) | inspector | HomeLight + Inspectify | `*Inspectify order update!* Home Inspection: Requested` → `Home Inspection: Scheduled, <datetime>` |
| **Conditional agreement (Google Doc)** | HomeLight | LO (portal) | HomeLight | `BBYS conditional agreement generated loan officer was emailed, and can review and send agreement in portal <docs.google.com/...>` |
| **Variable Pricing / fee exception approval** | Jake Vogel (HL) | LRM + Lender Ops | Jake Vogel | `<@LRM> <@LenderOps> Variable Pricing Approved` / `Variable Pricing Approved` / `2% Fee Exception Approved` |
| **EU Loan Funded** | HomeLight + Warehouse (JPM) | client (disbursed to IR title) | HomeLight Funding (Sarah Coyne, Renee Cook, John Baltazar) | `Loan Funded: <loan#>` + `IMAD to IR Title: <wire-code>` |
| **Trust Decision** | HomeLight trust subteam | file | trust subteam | `Trust Decision: *Approved* / Trust Type: *Revocable* / Trust Executed: *Yes*` |
| **Soft credit pull (Yellow / Failed)** | HomeLight ops (iSoftPull) | file | HomeLight ops | `<@LenderOps> :credit_card: *Auto Soft Credit Check Results*` |
| **Backup Offer (BUO)** | HomeLight (drafted) | client + DRA (signed) | HomeLight | `BUO Reviewed and approved` → `Final BBYSA and backup contract out for clients signature` |
| **Maintenance Reserve disbursement** | HomeLight analyst (post-funding) | client | HomeLight analyst | `MR disbursement of $X approved.` |

### 1b. LO/agent/client-issued artifacts (LO IS the owning party)

| Artifact | FROM | TO | Who can confirm | Corpus signal that issuance occurred |
|---|---|---|---|---|
| **IR Contract / Incoming Residence Contract** | LO/buyer agent | HomeLight (upload) | LO | `Requested IR Contract Upload from LoanOfficer <name>` → `AI has reviewed the IR Contract: ... Fully Executed?: Yes` |
| **Departing home photos** | LO/agent/client | HomeLight (upload tool) | LO/agent | `BBYS - Property photos uploaded. Uploaded Property Images can be accessed here: <link>` |
| **Soft pulls / credit report / 1003** | LO | HomeLight | LO | `Uploaded soft pull - <gdrive link>` (manual upload by Lender Ops on the LO's behalf when auto pull yellows out) |
| **Title docs / vested-owner names / lien balance** | LO / title company | HomeLight | LO | `Confirm lien balance` checklist item; vested-owner section in TitleBot pull |
| **Trust docs** | LO / client / trust attorney | HomeLight | LO | `Pending Trust Docs` checklist item; trust review subteam page |
| **HOI / hazard insurance declaration** | LO / client / insurance carrier | HomeLight | LO/client | `HOI Update Received Task completed` + `Home Owners Insurance Cleared Date updated to: <date>` |
| **Mortgage Statement** | LO / client | HomeLight | LO/client | `Pending mortgage statement` checklist item |
| **2nd-lien payoff** | LO / 2nd-lien servicer | HomeLight | LO | `Pending lien info` / `Second Lien Payoff Amount: <amount>` |
| **DR contract / DR Back up RPA** | DR agent | HomeLight | LO/agent | `Client signed the final BBYSA and DR Back up RPA.` |
| **Well tests / pest / foundation reports** | LO / inspector | HomeLight | LO | `pre funding - required repair - <type>` Property Condition flag |
| **T&E rep contact / Closing Title Company info** | LO | HomeLight | LO | filled into Closing Detail form by LO |
| **LO Closing Detail Link submission** | LO | HomeLight | LO | `LO Closing Detail Link` filled |
| **IR Closing Detail Confirmation** | LO | HomeLight | LO | `Zapier - Post-Contract Email Bot: *Builder Doc Collection and IR Closing Detail Confirmation request emails sent separately to client and builder*` |
| **EB application submission** | LO/client (via Equity-app link) | HomeLight underwriting | LO/client | `the application for equity boost has been submitted!` |
| **EB asset documents (bank statements, brokerage)** | client (via Extend.ai workflow) | HomeLight underwriting | client | structured asset list in EB submission block |

### 1c. Joint / bidirectional artifacts

| Artifact | Originator | Filler | Returner | Notes |
|---|---|---|---|---|
| **CIF / DocuSign Client Info Sheet** | HomeLight | LO/client | HomeLight | LO can fill on the client's behalf; client can fill via portal. |
| **IR Contract Questionnaire** | HomeLight (Lender bot, with link) | LO | HomeLight | `BBYS Lender - Incoming Residence Questionnaire email sent to <LO email>` |
| **CA Notes block** | HomeLight LSM (Ashlee/Kyle/Tiffany/Cheryl/Ian) | LSM | LRM + Listing Ops (handoff) | Per-LSM template variant — 10-line block. See [[bbys-deal-channel-vocabulary]] Channel Conventions section. |
| **UC Notes block** | HomeLight LSM (Cheryl Funk variant) | Cheryl | rolling status on channel | Strikethrough items as they complete. |

### 1d. Common LLM direction-inversion failure modes (CRITICAL)

These are the 5 inversion shapes the LLM does most often. All forbidden in the prompt:

1. **"Can you confirm the Equity Unlock approval?"** — asked TO the LO. The LO is the **recipient** of the EUA; HomeLight is the approver. ✗
2. **"Please confirm the approved amount."** — asked TO the LO. The LO is reading the amount FROM HomeLight; they don't validate it FOR HomeLight. ✗
3. **"Any notes we should add to the file re: the approval?"** — asked TO the LO. The LO doesn't write to HomeLight's file; HomeLight writes to its own file. ✗
4. **"Can you sign off on the BBYSA?"** — asked TO the LO. The LO doesn't sign the BBYSA — the **client** does. The LO forwards via portal. ✗
5. **"Please schedule the inspection."** — asked TO the LO. **HomeLight** schedules Inspectify after the IR contract is in; the LO is the **recipient** of the inspection-order email. ✗

Equivalent test: **if your email reads like it could have been sent by the LO TO HomeLight rather than HomeLight TO the LO, the direction is inverted. REWRITE.**

---

## 2. Bot Message Catalog

Organized by bot identity. Each row: verbatim trigger phrase (regex-extractable) → frequency → stages → direction → meaning → implied outstanding action.

Frequency notation: `H` = appears on >50% of channels in scope; `M` = 20-50%; `L` = <20%; `R` = rare (1-3 channels).

### 2a. HomeLight Sales App (U054856DP6W)

| Trigger (verbatim) | Freq | Stages | Direction | Meaning | Implied outstanding action |
|---|---|---|---|---|---|
| `*New Buy Before You Sell (Lender Platform) lead!*` | H | new | HL → channel | Lead just submitted | None — informational anchor for the channel |
| `*Submission confirmation email sent to LO — <email>*` + `:clipboard: *Required from LO*` | H | new | HL → LO (email); HL → channel (notification) | Confirmation of intake email + structured checklist | LO owes the items in the checklist |
| `has renamed the channel from "X" to "Y"` | H | all | HL → channel | Stage transition (authoritative) | None — closes prior stage's outstanding items |
| `HomeLight Support: Stage UPDATED to <stage>` | H | all | HL → channel | Stage flip (more reliable than rename) | Same as rename |
| `BBYS Lender - Lead Submission Confirmation Next Steps email sent` | H | new | HL → LO | Lead-submission email sent | LO owes the Required-from-LO items |
| `BBYS Lender Photo Upload Receipt Unified Comms email sent` | M | new, rev | HL → LO | Photo upload acknowledged | None — closes "pending photos" |
| `*TitleBot* - pulled <date> *Owner Names*: <names> *Vesting Owner*: <vesting>` | H | new | TitleBot → channel | Title pulled | If `Vesting Owner: unknown` → confirm vested owners |
| `Could not pull title for <address>.` | L | new | TitleBot → channel | Title pull failure | LO/title-company must confirm titleholder + vesting manually |
| `IR state unknown, ask LO to confirm eligibility` | L | new | HL → LRM | State not yet identified | LO confirms IR state |
| `:sunny: *Solar Panel Detection* *Property:* <addr> • *Result:* Yes/No • *Confidence:* <pct>%` | H | new | HL → channel | Solar detection result | If Yes → LO confirms lease vs owned in CA Notes |
| `:alert::alert: <@LRM>, NEEDS TO APPLY FOR EQUITY BOOST!! :alert::alert:` | L | new, rev | HL → LRM | High CLTV; needs EB | LRM nudges LO to start EB app |
| `:rotating_light: Deal is already Under Contract on Incoming Residence` | R | new | HL → channel | Already racing at intake | LRM expedites everything |
| `:bell: HELOC Lead Notification` | L | new | HL → channel | State-eligible for HELOC alternative | None — informational |
| `BBYS_apply_for_Equity_Boost Email email sent` | L | new | HL → LO | EB-application email sent | LO/client submits EB |
| `BBYS - Property photos uploaded. Uploaded Property Images can be accessed here: <link>` | H | new, rev | client/LO → HL | Photos uploaded | Closes "pending photos" |
| `Missing photos detected in photo submission: ["<room>"]` | M | rev | HL → LRM | Missing rooms in photo set | LO/client uploads missing rooms |
| `the application for equity boost has been submitted!` | M | rev | LO/client → HL | EB submitted | None — awaiting EB decision |
| `Equity Boost has been approved. EU: $X LPV: $Y HC: $Z Credit Score: ... Max Equity Boost: $...` | M | rev | HL → channel | EB approved | None — closes EB-application loop |
| `BBYS conditional agreement generated loan officer was emailed, and can review and send agreement in portal <gdoc-link>` | H | appr | HL → LO | Conditional agreement ready in portal | LO owes: review + send to client |
| `Email sent: BBYS_initial_approval_notification_to_loan_officer` | H | appr | HL → LO | Approval notification fired | LO acknowledges + advances |
| `Email sent: BBYS_review_agreement_changes_notification_loan_officer` | M | appr | HL → LO | Agreement changes (e.g., variable pricing addendum) ready | LO reviews + resends |
| `<@LRM>, <@LenderOps> Its been 2 business days and the Loan Officer has not reviewed the agreement` | H | appr | HL → LRM | LO hasn't opened conditional agreement | LRM nudges LO |
| `<@LRM>, <@LenderOps> Its been 7 business days and the Loan Officer has not sent the agreement` | H | appr | HL → LRM | LO opened but hasn't sent agreement to client | LRM nudges LO; escalating |
| `<@LRM>, <@LenderOps> Its been 14 business days and the Loan Officer has not sent the agreement` | M | appr | HL → LRM | Stuck — terminal-risk | LRM seriously escalates |
| `Loan Officer sent agreement to clients for signature` | H | appr | LO → HL | LO sent BBYSA to client | None — closes 2/7-day alert chain |
| `Agreement was sent to client(s) for signature (DocuSign).` | H | appr | HL → channel | DocuSign envelope dispatched | Client signs |
| `:eyes: Client <email> viewed or received the agreement (DocuSign).` | H | appr | DocuSign → channel | Client opened envelope | Client signs |
| `:lower_left_ballpoint_pen: Client <email> signed the program agreement (DocuSign).` | H | appr | DocuSign → channel | One client signed | If 2-client deal, awaiting second |
| `:white_check_mark: All clients completed signing the program agreement (DocuSign).` | H | appr → as | DocuSign → channel | All clients signed | None — stage flips |
| `<!subteam^S020NNR6YCE> Stage updated to agreement signed please perform HOA and title review` | H | as | HL → trust/title subteam | AS handoff | Subteam reviews HOA + title |
| `:credit_card: *Auto Soft Credit Check Results* *Primary Client:* <name> • Intelligence: <color> <bureau> • Report: <link> • Result: <Passed/Failed>` | H | as | iSoftPull → channel | Soft credit pulled | Yellow/Failed → manual review |
| `Uploaded soft pull - <gdrive link>` | M | as | Lender Ops manual → HL | Manual pull uploaded after Yellow/Failed | Closes soft-pull pair |
| `Requested IR Contract Upload from LoanOfficer <name>` | H | as | HL → LO | IR contract upload request fired | LO uploads IR contract |
| `System: BBYS Request IR Contract Upload (Buy Before You Sell lead)` | H | as | HL → channel | Upload-request system event | Pair to above |
| `<@LRM> <@LenderOps> AI has reviewed the IR Contract: File uploaded to: <link> Effective Date: <date> Fully Executed?: Yes/No Buyer Name: <names> Purchase Property Address: <addr> Close of Escrow: <date> Purchase Price: <amount> Loan amount: <amount> Down payment Amount: <amount> Financing Type: <type> Loan Contingency End Date: <date> Title and Escrow Company: <co> Sellers in Possession: <text> Counter Offer Present: Yes/No` | H | as → iruc | HL → LRM | AI contract verdict | If Fully Executed: No → LO uploads corrected; if Yes → stage advances |
| `<actor>: Stage UPDATED to Ir Contract (Buy Before You Sell lead)` + `Client IR contract upload (Homes): <client-link>` | H | iruc | HL → channel | IRUC stage flip | None — closes IR-contract upload |
| `BBYS LO - Order Inspection email sent` | H | iruc | HL → LO | Inspection-order email fired | LO awaits Inspectify |
| `*Request Inspection Task is ready* <inspectify-link>` | H | iruc | HL → ops | Inspectify task ready | Ops places order |
| `<!subteam^S020NNR6YCE> System: *Inspectify order update!* • Home Inspection: Requested` | H | iruc | Inspectify → HL | Inspection requested | Inspectify schedules |
| `<!subteam^S020NNR6YCE> System: *Inspectify order update!* • Home Inspection: Scheduled, <date> <time>` | H | iruc | Inspectify → HL | Inspection scheduled | None — closes |
| `<!subteam^S020NNR6YCE> System: *Inspectify order update!* • Home Inspection: Reschedule Requested (<@LSM>)` | M | iruc | Inspectify → HL | Inspection reschedule | Inspectify reschedules |
| `BBYS Lender - Incoming Residence Questionnaire email sent to <LO email> IR Contract Questionnaire link: <link>` | H | iruc | HL → LO | IR Questionnaire dispatched | LO fills it |
| `Buy Before You Sell Notary Scheduling link: https://equity.homelight.com/bbys/document-signing/<id>` | H | iruc | HL → client | Notary scheduling link issued | Client schedules |
| `Property Condition statuses have been updated: ... Open: * pre funding - required repair - <Roof/HVAC/Pest/Well/Mold/Foundation/Gas Leak>` | M | iruc | Inspectify/HL → channel | Repair flag open | Client/agent fixes |
| `BBYS_Equity_Boost_missing_documents Email` | L | rev | HL → LO | EB asset docs missing | LO/client uploads |
| `Trigger equity unlock document signing Task completed by <name>` | M | iruc → ctc | HL ops → channel | EU loan signing triggered | Notary scheduling fires next |
| `EU doc signing task completed. Notary Scheduling link: <link>` | M | iruc → ctc | HL → client | EU notary scheduled | Client signs |
| `BBYS LO - Schedule Closings Signing email sent` | M | iruc → ctc | HL → LO | LO informed of closing-signing schedule | None |
| `Send EUIDs Task completed` | M | iruc → ctc | HL ops → channel | EUIDs sent | None |
| `Task Alert: Review Backup Offer for <client>` | M | iruc → ctc | List Ops → channel | BUO review task fired | List Ops reviews |
| `BUO Reviewed and approved` / `BUO Reviewed and denied` | M | iruc → ctc | List Ops → channel | BUO outcome | If denied → re-do loop |
| `Final BBYSA and backup contract out for clients signature` | M | iruc → ctc | HL → client | Final docs out | Client signs |
| `Client signed the final BBYSA and DR Back up RPA. Files saved in SA and Gdrive` | M | ctc | client → HL | Final docs signed | None |
| `Departing Residence Backup Contract Signed Date updated to: <date>` | M | ctc | HL → channel | BUO signed timestamp | None |
| `Final Agreement Signed Date updated to: <date>` | M | ctc | HL → channel | Final BBYSA signed timestamp | None |
| `Snapdocs webhook received Event: Closing signed document available` | H | ctc | Snapdocs → channel | Doc available | None — fires many times |
| `Snapdocs webhook received Event: Status changed ... closed status updated to: Closing is closed.` | H | ctc | Snapdocs → channel | Closing complete | None |
| `BBYS Lender - Additional Document Request sent to <LO email> ... **Documents Requested**: - <doc-type>` | H | ctc | HL → LO | Doc gap before CTF | LO uploads. Recurrence (3+) = stuck |
| `Portal doc download auth code requested by <LO email> via sms: <code>` | M | ctc | LO → HL | LO auth | None |
| `new documents have been uploaded to the google drive <link>` | H | ctc | HL → channel | Doc uploaded | Closes ADR loop |
| `Uploaded document to Meridian Link for loan file: <loan#>. Please find the documents under EDocs in Meridian Link.` | M | ctc | HL → channel | ML upload | None |
| `Email sent: BBYS_Lender_CTF_Prior_to_Funding_Request` | M | ctc | HL → LO | CTF prior-to-fund request | LO uploads CTC proof |
| `Review Submitted Documents Task completed by <name>` | M | ctc | HL ops → channel | Doc review done | None |
| `Request Funding Task completed by <name>` | H | ctc | HL ops → channel | Funding requested | Wire fires |
| `BBYS Lender - CTC Wire Confirmation email sent` | H | ctc → irx | HL → LO | Wire confirmation | None |
| `Loan Funded: <loan#> (lead_id: <id>)` | H | ctc → irx | HL → channel | Loan funded | Stage flips to IR Closed |
| `Specialized Hybrid recording State/County. Please follow manual process.` | L | irx | HL → channel | Manual recording needed | Ops processes manually |
| `Recording Status for <loan#> (lead_id: <id>): RECORDED Timestamp: <date>` | M | irx | Simplifile → HL | Recording done | None |
| `Recording Status for <loan#> (lead_id: <id>): REJECTED` | L | irx | Simplifile → HL | Recording rejected | Ops investigates |
| `Confirm IR Closed Task completed by <name>. IR COE Date: <date>` | H | irx | HL ops → channel | IR closed task done | Stage flips |
| `Email sent: BBYS_Day_0_all_status` | M | irx | HL → channel | Day-0 post-close email fired | None |
| `Email sent: BBYS_Day_7_post_EU_Funding_DR_not_listed_agent_check_in_email` | M | irx | HL → channel | Day-7 DR check-in fired | LO/agent nudges DR listing |
| `Email sent: BBYS_LO_postIRclose_feedback_request` | M | irx | HL → LO | Post-close feedback request | LO replies (NPS) |
| `Maintenance Reserve Request <@<analyst>> client requesting $X in MR / This is for: <list>` | L | irx | LSM → analyst | MR disbursement requested | Analyst approves |
| `MR disbursement of $X approved. Taxes are escrowed in 1st mortgage. There will be $Y left in MR after this disbursement.` | L | irx | analyst → channel | MR approved | None |
| `Please prepare extension papers N days at X% expiring <date>` | L | irx | LSM → analyst | Extension request | Amendments drafted |
| `<actor>: Stage UPDATED to Failed (Buy Before You Sell lead)` | H | term | HL → channel | Terminal flip | None — channel renames `-term-` |
| `BBYS Lead #<id> has been moved to approved. <sales-app-link>` | M | appr | HL → channel | Approval mirror | None |
| `<@LRM> - Loan Officer has requested assistance with the approval review.` | M | appr | HL → LRM | LO asked for help | LRM calls LO |
| `<@LRM> - Loan Officer has scheduled a meeting on your calendar to review the approval.` | L | appr | HL → LRM | LO booked meeting | None |
| `Possible Duplicate Lead. This DR address has been previously submitted: <link to other lead>` | L | new | HL → channel | Duplicate-lead alert | LSM fails the dupe |

### 2b. HomeLight Sales App `System:` event variants (high-frequency)

These are bot-relayed copies of LO comms — they look like prose but are deterministically prefixed. **The text body inside the System line IS high-signal**: it's the verbatim ask or call notes.

| Trigger (verbatim prefix) | Freq | Stages | Direction | Meaning | Implied outstanding action |
|---|---|---|---|---|---|
| `System: Emailed loan officer (Buy Before You Sell lead): <body>` | H | all | LRM/LSM → LO (relayed by HL) | Outbound email from HL rep to LO | Body usually contains the ask. Parse for action verbs |
| `System: Loan officer emailed (Buy Before You Sell lead): <body>` | H | all | LO → HL (relayed) | Inbound email from LO | Body often contains LO question. Watch for "I have a question", "can we...?", "is it possible..." |
| `System: Called loan officer (Buy Before You Sell lead): <call notes>` | M | all | LRM/LSM → LO (relayed) | Outbound call summary | Body describes call. Watch for committed actions ("LO said they would upload by EOD") |
| `System: Called lo left message (Buy Before You Sell lead): <notes>` | M | all | LRM/LSM → LO (relayed) | VM left for LO | LO owes call-back |
| `System: Texted loan officer (Buy Before You Sell lead): <text>` | M | all | LRM/LSM → LO (relayed) | SMS sent | LO replies via SMS chain |
| `System: BBYS Request IR Contract Upload (Buy Before You Sell lead)` | H | as | HL → LO | Upload-request system event | LO uploads |
| `System: BBYS Lender CTF Prior to Funding Request (Buy Before You Sell lead)` | M | ctc | HL → LO | CTF doc request | LO uploads CTC proof |

### 2c. HomeLight Homie (U08P6K8KNVC)

| Trigger (verbatim) | Freq | Stages | Direction | Meaning | Implied outstanding action |
|---|---|---|---|---|---|
| `:robot_face: Automated email just sent to *<LO name>* - <<email>> - on *<DR addr> - <city> - <zip> - <CLIENT NAME>*: *[LO Client Welcome Onboarding]* Hold off on manual follow-up for ~24h to avoid double-tapping.` | H | new | HL → LO | LO welcome email | None |
| `:robot_face: Automated email just sent to *<LO>* - <<email>> - on *<DR addr> - <city> - <zip> - <CLIENT>*: *[HL Lenders] FUP #1 \| BBYS Winback \| EUA received follow-up* Hold off on manual follow-up for ~24h to avoid double-tapping.` | H | **appr** (post-EUA) | HL → LO | **FUP #1**: client got EUA, no engagement ~5d | **Re-engagement** — NOT approval-confirmation |
| `[HL Lenders] FUP #2 \| BBYS Winback \| EUA received follow-up` | M | appr | HL → LO | **FUP #2**: ~5d after #1, still no engagement | Re-engagement; escalating |
| `[HL Lenders] FUP #3 \| BBYS Winback \| EUA received follow-up` | M | appr | HL → LO | **FUP #3**: ~5d after #2, terminal-risk | Re-engagement; LRM should manually intervene |

**WINBACK FUP CADENCE (corpus-validated, n=20):**
- FUP #1 fires ~5 business days after `Stage UPDATED to Approved` + no agreement reviewed/sent.
- FUP #2 fires ~5 business days after #1.
- FUP #3 fires ~5 business days after #2.
- After FUP #3, manual follow-up window opens (no more automated FUPs).
- **Parallel to FUP series**: HL Sales App escalating-alert chain fires `2-day → 7-day → 14-day` "agreement not reviewed/sent" pings. The FUP series and the alert chain are independent — both can fire on the same channel on the same day.

### 2d. Other bots

| Bot | Trigger (verbatim, near-verbatim) | Freq | Stages | Direction | Meaning |
|---|---|---|---|---|---|
| **Easy Automations Service Account** (U04QDD4EVL7) | `<@U04QDD4EVL7\|Easy Automations Service Account> has joined the channel` | H | all | bot → channel | Auto-join, never posts content |
| **HomeLight Hub** | `has joined the channel` | H | all | bot → channel | Context-population join |
| **Zapier - Post-Contract Email Bot** (U014H9S8E7J) | `*Builder Doc Collection and IR Closing Detail Confirmation request emails sent separately to client and builder. Emails Titled: "BBYS: Under Contract! Next Steps - <client> - <addr>"` | M | iruc | HL → LO + client | Builder doc collection started |
| **Zapier - Post-Contract Email Bot** | `*BBYS Loan Documents have been uploaded by the client.* Submission Notes: Please begin file set up if you haven't already.` | M | iruc | client → HL | Builder docs returned |
| **Closing Signing Request Form** / **Prelim Funding Confirmation** | `<@Sarah> <@Renee> <@Derek> BBYS Loan #: <loan#> Warehouse: JPM Prelim Funding Date: <date> Note Amount: <amt> Total EU Amount: <amt> Disburse to IR Title: <date>` | H | ctc | HL → funding team | Prelim funding scheduled |
| **Closing Signing Request Form** / **Prelim Funding Confirmation** | `<@Sarah> <@Renee> <@Derek> Amount to IR Title: <amt> Second Lien Payoff Amount: <amt> IMAD to IR Title: <imad-code> Disbursed Date: <date>` | H | ctc → irx | HL → funding team | Wire confirmed; IMAD is wire receipt |
| **Contingency Removal Bot** | `*Contingency Removal Only Next Steps Email sent!* LO/LOA Email: <emails> Email Title: "BBYS: Under Contract Action Needed - <client>"` | L | iruc | HL → LO | CRO file handoff (EU=$0, DTI Drop variant) |
| **Slackbot** | OOO auto-replies | M | all | bot → channel | OOO; usually skip |
| **TitleBot** (within HomeLight Sales App) | see Sales App section | H | new | TitleBot → channel | Title pull |
| **Solar Panel Detection** (within HomeLight Sales App) | see Sales App section | H | new | Solar bot → channel | Solar result |

### 2e. Mixed-identity high-signal posts

Don't blanket-deprioritize these — they look botty but carry real signal:

- `:bell: HELOC Lead Notification`
- `:rotating_light: Deal is already Under Contract on Incoming Residence`
- `:alert::alert: NEEDS TO APPLY FOR EQUITY BOOST!! :alert::alert:`
- `INVALID_RECIPIENT_ID` (real tech blocker)
- `Inspection has not been scheduled yet please follow up`
- `Additional Document Request sent` (especially with `Incoming Residence Proof of Clear to Close`)
- AI IR Contract review verdicts (`Fully Executed?: Yes/No` is the canonical gate)
- `*Auto Soft Credit Check Results*` (Yellow/Failed needs human action)
- `Trust Decision: *Approved/Declined*`

---

## 3. Human Message Patterns

### 3a. LSM/LRM-to-LenderOps tagging (NEW + IN_REVIEW)

Compact pending-items list. Shape:
```
<@LRM-or-LenderOps>
Pending photos
Agent info
Pending email for [name]
Confirm lien balance
This in Trust — Need Trust Docs
```
- **Direction**: Lender Ops → LRM (asking LRM to chase LO).
- **Outstanding action**: LRM emails/calls LO to collect each bullet.

### 3b. CA Notes block (AGREEMENT_SIGNED → IRUC handoff)

Authored by an LSM (Ashlee Kim, Kyle Bradish, Tiffany Traxler, Cheryl Funk, Ian Pardo). Shape:
```
• IR Address: <addr>
• COE: <date>
• Financing: <Conventional/FHA/VA/...>
• EU Amount: $<amt>
• EB Amount: $<amt>
• Contingency Removal Only: <Yes/No>
• Second Lien: <Yes/No> $<amt>
• Solar: <No / Yes - Lease already provided / Yes - Owned>
• Trust: <No / type-and-execution>
• HOA: <Yes/No/Unsure>
Situation: <free-text incl. first-time-LO flag, urgency notes, pricing exceptions>
```
- **Direction**: LSM → LRM + Listing Ops (handoff doc).
- **Outstanding action**: LRM hands off; Listing Ops takes over channel ownership.

### 3c. Cheryl Funk's `*UC Notes - LO DR Val: $X*` block (IRUC → CTC)

Sibling artifact to Patricia Pinckard's `:extreme-teamwork:` `*Pending:*` checklist. Shape:
```
5/15 *UC Notes - LO DR Val: 561,856*
• COE = 6/14
• EU = $100,000
• *Vesting -* <names>
• DR Address: <addr>
• DR year Built: <year>
5/15: Pending
-Inspection - *date*
-EUID - not sent
-Final BBYSA + RPA - not sent
-schedule EU Closing est:
-CTC?
```
Items get crossed off in subsequent re-posts. Maps to multiple HubSpot milestones simultaneously.

### 3d. Wire-release request (CTC)

Shape: `<@<funding analyst, usually Johnna Gallego / Renee Cook / John Baltazar>> Please release wire to IR title for <date>`
- **Direction**: LSM → Funding team.
- **Outstanding action**: Funding analyst releases wire; `BBYS Lender - CTC Wire Confirmation email sent` + `Loan Funded:` close the pair.

### 3e. Variable pricing approval ask (APPROVED + IRUC)

Shape: `<@<Jake Vogel>> Any chance for variable pricing for this home? Not traditional over $1m, but clients are very fee sensitive...`
Resolution: `<@<LRM>> <@<LenderOps>> Variable Pricing Approved` (Jake Vogel posts).

### 3f. LO email replies relayed via `System: Loan officer emailed:`

Critical structural pattern: the **content** of these inbound emails is high-signal but the model sometimes misses that they're from the LO. The display name is `HomeLight Sales App` (the bot relaying); the actual author is the LO.

Recurring shapes:
- *"I just wanted to follow up with you on this as the borrowers have still yet to sign these documents..."* — LO escalating a stuck BBYSA-signing.
- *"Wow! Lightening fast! Thank you :blush: I'm looking forward to sharing with my client in the morning."* — LO ack on agreement received.
- *"Unfortunately they decided not to pursue this purchase. Lets keep the application active for a bit."* — LO killing the IR contract but keeping file alive.
- *"Is it a possibility that I schedule a call with you, me and the borrowers together to discuss these concerns and have you answer their specific questions?"* — LO requesting 3-way.

### 3g. Winback FUP human follow-ups (post-FUP-#1)

Real LSM responses observed in the corpus to a `[HL Lenders] FUP #N | BBYS Winback | EUA received follow-up`:

- `Per LO: Client is home shopping` (Deborah Shutt — most common single-line ack)
- `Clients are home shoppping!` (Deborah Shutt, repeated post-FUP-#3)
- *(silence — LSM didn't respond between FUP fires)*

These are the canonical human responses to a Winback signal. The AI Draft should generate emails to the LO that match these LSM-internal acks in tone — short, operational, no recap.

### 3h. Cancel-trigger human messages (terminal flip)

- `Please fail this file <@LenderOps>` — LSM-internal kill switch
- `Borrower is canceling the bridge loan` — LO relayed
- `Please cancel this application` — client direct
- `Go ahead and fail this one out!` — duplicate-lead resolution
- `Hey, ya that's not going to work. Not enough to move to the next house. Thanks` — observed verbatim from LO in Wilson channel; structural-ineligibility / fee resistance

---

## 4. Stage-Transition Signals

| Transition | Authoritative event | Secondary event | Channel rename |
|---|---|---|---|
| `new → rev` | `HomeLight Support: Stage UPDATED to In Review` | photo upload + LRM tasking | `*-new-*` → `*-rev-*` |
| `rev → appr` | `HomeLight Support: Stage UPDATED to Approved` | `BBYS conditional agreement generated` + `Email sent: BBYS_initial_approval_notification_to_loan_officer` | `*-rev-*` → `*-appr-*` |
| `appr → as` | `:white_check_mark: All clients completed signing the program agreement (DocuSign).` + `HomeLight Support: Stage UPDATED to Agreement Signed` | `<!subteam^S020NNR6YCE> Stage updated to agreement signed please perform HOA and title review` + Auto Soft Credit Check Results | `*-appr-*` → `*-as-*` |
| `as → iruc` | `<actor>: Stage UPDATED to Ir Contract` | `Client IR contract upload (Homes): <link>` + `BBYS Lender - Incoming Residence Questionnaire email sent` + `Buy Before You Sell Notary Scheduling link` + `*Request Inspection Task is ready*` | `*-as-*` → `*-iruc-*` |
| `iruc → ctc` | `<actor>: Stage UPDATED to Clear To Fund` | `Request Funding Task completed by <name>` + `Signed Docs Received Task completed` | `*-iruc-*` → `*-ctc-*` |
| `ctc → irx` | `<actor>: Stage UPDATED to Ir Closed` | `Loan Funded: <loan#>` + `Confirm IR Closed Task completed` + `Email sent: BBYS_LO_postIRclose_feedback_request` | `*-ctc-*` → `*-irx-*` |
| `irx → drx` | `<actor>: Stage UPDATED to Dr Closed` | AI-reviewed DR contract + `Email sent: BBYS_DR_in_escrow_payoff_request` | `*-irx-*` → `*-drx-*` |
| `<any> → term` | `<actor>: Stage UPDATED to Failed` | Cancel-trigger human message; sometimes DocuSign void; sometimes channel archive | `*-<prev>-*` → `*-term-*` |
| `term → <revival>` | `<user> unarchived the channel` | New client/LO message reviving file | (rename may not fire) |

**Bounce-rename caveat (from [[bbys-deal-channel-vocabulary]] n=53):** channels occasionally bounce-rename (Sweeney: `appr → rev → appr → as` in 2 hours). The most-recent `HomeLight Support: Stage UPDATED to <stage>` is authoritative; channel name string can lag.

---

## 5. Open Questions / Ambiguities

Things the corpus does NOT fully resolve — flagged for future sampling or operator confirmation:

1. **FUP cadence variation.** All n=20 Winback FUPs sampled this round were `#3` or `#2` — only one `#1` confirmed in the n=20. The ~5d cadence is inferred from a single full-channel timeline (Paynter). Future work: parse all FUP timestamps per-channel and confirm interval distribution.
2. **`-drx-` (DR_CLOSED) corpus is thin** in this sample. Recommend a separate DR-Closed-stage sample of n=10 channels to validate the wrap-up bot catalog (post-close feedback emails, Reconveyance, MERS final, lien-release failures).
3. **Builder-channel patterns** (Lennar / K-Hov / DR Horton) underrepresented in this sample. The CRO + Zapier Post-Contract Email Bot flows differ structurally. Future: dedicated builder-channel sample.
4. **`Solar: Yes - Owned` vs `Yes - Lease already provided`** — confirmed in n=53 corpus but only 1/n=6 in this round. Underwriting impact: lease-owned distinction materially changes loan packaging. Worth a separate solar-deal sample to nail the boundary.
5. **Pricing exception taxonomy.** Variable / 2% flat-fee / 180-day pilot / mortgage holdback all exist in corpus but no clean coverage of which-LO-gets-which-exception. Operator (Jake Vogel) is the authority — could codify in a pricing-decision table.
6. **`Recording Status: REJECTED` recovery loop** is real (n=53) but observed in only n=1 here. Worth a dedicated `-irx-` sample of n=10 to map state/county → manual-vs-automated breakdown.
7. **`Specialized Hybrid recording State/County. Please follow manual process.`** vs `Specialized Manual recording` — observed both phrasings; unclear whether they trigger different downstream flows.
8. **CRO/DTI Drop direction.** The CRO Next Steps Email Bot is its own identity but the underlying artifact (EU = $0 / DTI Drop variant) is structural. Whether to treat as a distinct stage variant or a property of the IRUC stage is open.

---

## 6. Parser Strategy — what's deterministically resolvable

A summary of what `signalLexicon.ts` can handle deterministically vs what the prompt still has to do:

**Resolvable in deterministic parser (probably ~75-80% of the ARTIFACT DIRECTION TABLE):**
- Stage detection — `HomeLight Support: Stage UPDATED to <stage>` (regex-driven; tolerate bounce-rename).
- Winback FUP detection — `[HL Lenders] FUP #<N> | BBYS Winback | EUA received follow-up` from `HomeLight Homie` (verbatim, the FUP # is a literal integer).
- Approval state — `Stage UPDATED to Approved` + presence of escalating-alert messages = pending; `:white_check_mark: All clients completed signing the program agreement` = signed.
- BBYSA signing chain — DocuSign emoji ladder is closed-form.
- AI IR Contract verdict — `Fully Executed?: Yes/No` parseable.
- HOI cleared — `Home Owners Insurance Cleared Date updated to: <date>`.
- Loan funded — `Loan Funded: <loan#>`.
- Soft credit color — `Intelligence: <color> <bureau>`.
- Variable pricing approval — `Variable Pricing Approved` from Jake Vogel.
- IRUC entry — `Stage UPDATED to Ir Contract` + `Client IR contract upload (Homes): <link>`.
- Inspection state — Inspectify webhook chain.
- Wire / IMAD — `IMAD to IR Title: <code>`.
- Section flags — 12-boolean block at intake.
- Trust decision — `Trust Decision: *Approved* / *Declined*`.
- EB submission/approval — explicit signal blocks.
- Recording state — `Recording Status: ...: <RECORDED/REJECTED/FINALIZING>`.

**NOT resolvable deterministically (must stay in prompt):**
- Free-text `System: Emailed loan officer:` ask content — the body is prose and the model must interpret intent. Best the parser can do is extract the prose and tag it as "outbound LO email" without parsing the ask.
- LO-question parsing inside `System: Loan officer emailed:` — same prose problem.
- CA Notes "Situation" free-text field — first-time-LO flag is parseable; the rest is freeform.
- Human ambiguity ("PC received" — phone-call or purchase-contract? Context determines.).
- Tone matching to per-state voice — corpus-validated voice rules belong in prompt.
- Per-state opener selection (REP'S ACTION / DIRECT ANSWER / BARE ASK / ACKNOWLEDGEMENT / CLIENT-CALL-NARRATION).
- Pricing-exception negotiation flow (LO ask → Jake response).

**Rough split:** if `ResolvedDealState` carries 15 closed-form fields (stage, outstandingArtifacts, recentlyCompleted, winbackState, approvalState, eua_sent_at, inspection_state, hoi_cleared_at, loan_funded_at, soft_credit_color, variable_pricing_approved, ir_contract_executed, bbysa_signed, ctc_proof_uploaded, recording_state) — those 15 cover ~75% of "what's blocking this deal" signal. Prompt then handles voice, tone, opener selection, and prose-ask interpretation on top.

---

## 7. Update protocol

When the BBYS bot landscape shifts (new bot identity, new event template, new stage label) update this doc AND `signalLexicon.ts` in lockstep. Tests for the parser should fail loudly when a new bot pattern appears and isn't catalogued.

Related artifacts to keep in sync:
- `[[bbys-deal-channel-vocabulary]]` (the umbrella lexicon — read FROM, write TO if convention shifts)
- `[[bbys-stage-progression]]` (per-stage corpus)
- `src/hubspot-email/draftGenerator.ts` BBYS ARTIFACT DIRECTION TABLE block
- `src/hubspot-email/signalLexicon.ts` (parser)
- The Action-Pair table in `[[bbys-deal-channel-vocabulary]]`
