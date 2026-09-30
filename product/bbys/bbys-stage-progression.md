<!-- source: HomeLight-Vault/context/bbys-stage-progression.md | imported: 2026-09-29 -->

# BBYS Deal Stage Progression (Slack-Grounded)

**Source**: Synthesis of 36 BBYS deal Slack channels sampled 2026-05-13 across all 8 active pipeline stages (5 each NEW / IN_REVIEW / APPROVED / AGREEMENT_SIGNED / IR_IN_ESCROW; 4 CTC; 5 IR_CLOSED; 2 DR_CLOSED).

**Purpose**: Ground-truth reference for what *actually* happens in each BBYS deal stage — what's discussed, what's needed, where it lives, who owns it, how it's presented. Used to keep the Copilot agent from hallucinating generic mortgage-checklist responses ("signed borrower authorization", "property photos", etc.) when answering "what does this deal need to move forward."

**Naming convention**: Every channel is prefixed with the LO's lender (`mig-`, `uwm-`, `orchard-`, `tls-`, `fairway-`, `cross-country-`, `rwm-`, `apm-`, `dr-horton-`, `first-continental-`) and a stage token (`new`, `rev`, `appr`, `as`, `iruc`, `ctc`, `irx`, `drx`, `term`). The Sales App auto-renames the channel on every stage transition — the rename event itself is a strong signal.

---

## NEW (998755441)

**Conversation.** Channel auto-created on lead submission. Almost all messages are bots: `HomeLight Sales App` posts the canonical "New Buy Before You Sell (Lender Platform) lead" block (client, LO, agent, approval type, lender/agent valuation, target unlock amount, equity unlock range, finance type, upload tokens); `TitleBot` posts vested owner names + legal description + year built; `Solar Panel Detection` posts result; `HomeLight Homie` confirms the LO sent the Client Welcome Onboarding email. Human chatter is sparse and dominated by Lender Ops (Michelle Guarino, Jessie Guiang) tagging the LRM with an outstanding-items list: `<@LRM> Pending photos / Pending email for [spouse] / Confirm lien balance / This in Trust — Need Trust Docs`. LRMs (Ashlee Kim, Ian Pardo, Barb Griego, Angelica Espinosa) sometimes paste a "called the LO and left a VM" note.

**Artifacts.** Submission confirmation email lists "Required from LO" items: departing-home photos, listing-agent contact info, loan-assistant contact info, lien balance, titleholder/vested-owner email, trust documentation. Other surfaces: TitleBot output, Equity Boost App Link, Property Photos Upload Link, IR Contract Upload Link, HELOC notification, "Incoming residence under contract at intake" red-light banner, Solar Panel detection, lender stats (`Lender Leads`, `Lender Closes`), first-time-LO flag.

**Where info lives.** Sales App at `sales-app.homelight.com/buy-before-you-sell/leads/{id}` is canonical. Client-facing uploads at `equity-app.homelight.com/client/*`. HomeLight Hub joins the channel within ~minutes of creation.

**Blockers.** "NEEDS TO APPLY FOR EQUITY BOOST" alert fires when CLTV is high; LRM has to nudge LO into the Equity Boost flow. Missing photos, missing vested-owner email, missing spouse/co-titleholder email, unconfirmed lien balance, trust documentation absent, LO unresponsive ("LVM for LO"), unknown IR state. First-time LOs typically need a kickoff call from the LRM.

**Presentation.** Bot blocks use bullets + `*bold*`. Humans use short tags. Emojis: `:alert::alert:` (must-apply-for-EB), `:rotating_light:` (already UC at intake), `:bell:` (rule-matched notification), `:sunny:` (solar detection).

**Stage-exit signal.** `HomeLight Sales App: HomeLight Support: Stage UPDATED to In Review`, channel rename `*-new-*` → `*-rev-*`, plus `Tasked for with 24 hour turnaround`.

---

## IN_REVIEW (998755442)

**Conversation.** Lender Ops is actively underwriting. Photo upload receipts (`BBYS — Property photos uploaded`) repeat as the client uploads. Lender Ops pings the LRM with the remaining outstanding-items checklist. LRM pushes back on Lender Ops to rush ("Please rush review, clients found dream home and looking to make an offer ASAP"). If the deal needs Equity Boost, the LO gets the equity-boost application sent — followed by `BBYS_Equity_Boost_missing_documents Email`, and eventually `the application for equity boost has been submitted!` with primary/additional client names, credit-score buckets, asset documents pointing at an Extend.ai workflow run.

**Artifacts.** Equity Boost app submission (credit score, asset accounts, account types like "Investment or Brokerage", asset docs Extend workflow link), `Target Unlock Amount` adjustments ("LO looking for max EU $300,000"), variable-pricing requests routed to Jake Vogel for approval. AI-reviewed IR Contract block surfaces when LO uploads contract: effective date, fully executed, buyer name(s), IR property address, COE, purchase price, loan amount, down payment, financing type, loan contingency end date, title and escrow company, sellers in possession, counter offer present.

**Where info lives.** Sales App lead detail (system of record for the conditional approval), Extend.ai workflow dashboards for asset and IR-contract OCR, Google Drive "Deals folder" auto-created by the contract review bot.

**Blockers.** Missing photos (`Missing photos detected in photo submission: ["exterior"]` recurs), HELOC eligibility decisions ("HELOC prequalification decision updated... Combined LTV of 93.33% exceeds 89.99% limit"), variable-pricing approval (default 2.4% — Vogel signs off on alternate pricing), Equity Boost not yet applied for, ambiguous title/vesting (e.g. trust with no docs).

**Presentation.** LRM-to-LenderOps tagging is heavy. Pricing exception requests use a short `<@Jake Vogel>` thread. Lender Ops note lists use compact "Pending: A, B, C" bullets.

**Stage-exit signal.** `Stage UPDATED to Approved` + channel rename to `*-appr-*` + `BBYS conditional agreement generated loan officer was emailed` + Google-Doc link to the agreement + `Email sent: BBYS_initial_approval_notification_to_loan_officer`.

---

## APPROVED (998755443)

**Conversation.** Two clocks now run in parallel: (1) get LO to review the conditional agreement and forward to clients, (2) get clients to sign DocuSign. The LRM calls the LO ("LVM for LO about conditional approval", "ran through full approval call, got the green light to send over for signatures") and the bot logs every email/call. When the LO is fee-sensitive, they request variable pricing or a DTI-Drop alternative; LRM tags Jake Vogel for the pricing approval. Bot pings escalate over time: `Its been 2 business days and the Loan Officer has not reviewed the agreement`.

**Artifacts.** "BBYS conditional agreement" Google Doc, DocuSign envelope events (`:eyes: Client viewed`, `Agreement was sent to client(s) for signature`, `:lower_left_ballpoint_pen: signed`, `:white_check_mark: All clients completed signing`), variable-pricing addendum PDF, program-fee explainer (2.4% standard, 1% DTI Drop / $5k min if clients bring own funds, 120-day sale window, 15–20 day processing). LO frequently emails questions about HELOC vs BBYS, fee negotiability, contingency reserves, maintenance holdbacks — these come in via "Loan officer emailed" bot relays.

**Where info lives.** Google Doc agreement, DocuSign envelopes (no direct URL in channel — only event log), Sales App lead detail, the "portal" the LO uses to review/send the agreement.

**Blockers.** LO unresponsive on conditional review; client questions LRM can't answer over email (Lindsay Terry case — borrowers wouldn't sign without a 3-way call); fee resistance; competing HELOC; pricing changes requiring re-issue + new DocuSign; client-data corrections (wrong email on file, wrong client name, removing non-titleholders from the SA). Failed deals exit here as `Stage UPDATED to Failed` + channel renamed to `*-term-*` + archived.

**Presentation.** DocuSign events are bot-driven and emoji-prefixed. Pricing approvals are one-liners from Jake Vogel: `<@LRM> <@LenderOps> Variable Pricing Approved`. LRM frequently quotes the LO's email back into the channel.

**Stage-exit signal.** `All clients completed signing the program agreement (DocuSign)` → `Stage UPDATED to Agreement Signed` → channel rename `*-appr-*` → `*-as-*` → `<!subteam^S020NNR6YCE> Stage updated to agreement signed please perform HOA and title review` + auto soft-credit-check results (Green/Yellow/Orange TU or EX).

---

## AGREEMENT_SIGNED (998755444)

**Conversation.** Handoff from LRM (intake) to the Client Manager / Listing Ops world. Soft credit results post for primary + additional clients (intelligence color + isoftpull report link). LRM (or whoever's on intake — Ian Pardo, Brandi Cirell, Tiffany Traxler, Michael Coffey) is now waiting on the IR purchase contract upload from the LO. The LO gets reminded via SMS/email (`Requested IR Contract Upload from LoanOfficer`). When the contract lands, the AI-reviewed IR Contract block posts and the deal can move to IR_IN_ESCROW. If clients aren't actually under contract on the IR yet, the deal sits here. Edge cases get heavy: title issues ("titleholder doesn't match buyer", "father gifting equity through BBYS", VA financing conflicts).

**Artifacts.** Auto soft credit check (`Green TU / Yellow EX / Orange EX`), iSoftPull report link, BBYS Final Agreement, IR purchase contract (PDF), HOA and title review trigger to the `S020NNR6YCE` subteam, FAQs about the 120-day guarantee, maintenance reserves, contingency removals.

**Where info lives.** Sales App lead detail (stage column drives everything), iSoftPull reports for credit, Google Drive deals folder for contract PDFs.

**Blockers.** Client hasn't found an IR yet; LO can't upload the purchase contract (portal error); title doesn't match buyer (need quitclaim or restructure); HOA/title issues discovered on review; client backs out (VA financing conflict → fail).

**Presentation.** Soft-credit block is a structured bot post; humans add short "PC coming soon"/"waiting for PC" notes. HOA/title escalations use `<!subteam^...>` paging.

**Stage-exit signal.** LRM moves the stage via `Stage UPDATED to Ir Contract` (note: HubSpot stage label is "IR_IN_ESCROW" / 998755445 but bot text says "Ir Contract"). Channel renames `*-as-*` → `*-iruc-*`. Triggers: BBYS LO order-inspection email, IR Contract Questionnaire email to LO, EU notary scheduling link, Inspectify task creation. Listing Ops (Derek Lupien, Penny, Patricia Pinckard, Cheryl Funk, etc.) join the channel here.

---

## IR_IN_ESCROW (998755445) — aka "IRUC" / "IR Contract"

**Conversation.** The most operationally dense stage. The Client Manager / Listing Ops takes over and posts a `*CA Notes:*` block (template: IR Address, COE, Financing, EU Amount, Contingency Removal Only, Second Lien, Solar, Trust, HOA, Situation). Then a rolling "UC Notes" block tracks pending items strikethrough-style:

```
5/12: Pending
-Inspection - date
-EUID - not sent
-Final BBYSA + RPA - not sent
-schedule EU Closing est:
-CTC?
```

Inspectify webhook updates (`Home Inspection: Requested / Scheduled , [date] / Reschedule Requested / Completed`) chain through. LOs are pinged for the IR Closing Detail Confirmation. Property Condition status blocks list `pre funding - required repair - Foundation / HVAC / Well / Pest`, `pre hl purchase - required repair`, `pre funding - hoi - HOI`. The DR side ramps up: agent confirms total offers received, lowest/highest offer, whether using HLCS (HomeLight Closing Services), DR contract gets uploaded and AI-reviewed.

**Artifacts.** Inspectify order, BBYS Notary scheduling link, IR Contract Questionnaire, EUID (Equity Unlock Initial Disclosure), Final BBYSA + RPA (Real Property Addendum), Mortgage Statement, HOI declaration with limit floor (`If hazard property damage limits are lower than $X, HomeLight will require it raised to $X`), 2nd-lien payoff, title/escrow company info, T&E rep name/email/phone, agent contact details, LO Closing Detail Link, Cash to Close confirmation, DocuSign Client Info Sheet (CIF), Credit Report, Wholesale lender (if brokered), 1003, DR Backup RPA, well tests (state-specific), Meridian Link uploads ("under EDocs in Meridian Link"), DR contract.

**Where info lives.** Sales App, Inspectify portal, Meridian Link (loan file under EDocs), Google Drive deal folder, Snapdocs (notary signing orchestration), `homelightlender.inspectify.com`, `equity.homelight.com/bbys/document-signing`.

**Blockers.** Open property-condition repairs (foundation/HVAC/pest/well), HOI insurance limit too low, 2nd lien payoff not received, mortgage statement missing, well water-quality test results pending, CTC (Clear to Close on IR loan) not yet from external lender, DR not yet UC, title/vesting cleanup, agent-contact info missing, no IR Closing Detail Link from LO.

**Presentation.** Big strikethrough-checklist messages from the Client Manager pinned to the channel; emojis: `:extreme-teamwork:`, `:heart:`, `:fire:`. Tasks closed by name show as `[Task name] Task completed by [Person]`. Cross-team pings to `<!subteam^S020NNR6YCE>` for HOA/title.

**Stage-exit signal.** `Stage UPDATED to Clear To Fund` + channel rename `*-iruc-*` → `*-ctc-*`. Precondition stack: HOI cleared, signed closing docs received (Snapdocs `closed status: Closing is closed`), 1003/credit/mortgage statement in, IR Proof of Clear to Close uploaded, Wire Instructions Received, Request Funding task completed.

---

## CLEAR_TO_FUND (998755446)

**Conversation.** Funding mechanics. Listing Ops / Lender Ops orchestrates prelim funding date and disbursement-to-IR-title date. Recurring template from "Prelim Funding Confirmation" bot:

```
BBYS Loan #:  2026040230
Warehouse: JPM
Prelim Funding Date:  5/8/2026
Note Amount: 125,870.00
Total EU Amount: 100,000.00
Disburse to IR Title: 5/11/2026
```

Then a follow-up "Amount to IR Title / IMAD to IR Title / Disbursed Date" confirms the wire actually moved. Snapdocs webhook chatter (Closing/Documents/Esign/Signing Appointment status). Notary order completion. Light human chatter — mostly `<@Lender Ops Person> Please release wire to IR title for 5/14` and `:heart:` acknowledgements.

**Artifacts.** BBYS Loan #, JPM warehouse line, Note Amount, EU Amount, IR title wire IMAD, signed Note/Deed/Loan Terms PDF, HOI clearance, Meridian Link EDocs upload, Simplifile recording package URL.

**Where info lives.** Meridian Link (loan file 2026XXXXX), Snapdocs `homelighthomeloansinc.snapdocs.com/closings/{uuid}`, Simplifile `simplifile.ice.com/sf/ui/submitter/package/...`, Google Drive deal folder.

**Blockers.** IR Proof of CTC not received from external lender (recurring `BBYS Lender - Additional Document Request sent`), HOI not yet cleared, wire instructions not received, 2nd-lien payoff coordination, prelim funding date slip.

**Presentation.** Highly bot-dominated; humans use 1-line release approvals. Snapdocs status webhook fires many times per closing (signed docs available × N, then "Closing is closed").

**Stage-exit signal.** `BBYS Lender - CTC Wire Confirmation email sent` + `Amount to IR Title: $X` confirmation → `Confirm IR Closed Task completed` with `IR COE Date: MM/DD/YYYY` → `Stage UPDATED to Ir Closed` + channel rename `*-ctc-*` → `*-irx-*`.

---

## IR_CLOSED (998755447)

**Conversation.** Funding done, IR purchase closed, focus shifts entirely to the DR (Departing Residence). `Email sent: BBYS_Day_0_all_status` fires day-of-close, then `BBYS_Day_7_post_EU_Funding_DR_not_listed_agent_check_in_email` at day 7. Simplifile recording status flips to `RECORDED` (occasionally with document errors like `DATA_FIELD_REQUIRED_MISSING: Tax Exempt? is required` that need manual fix). The listing agent updates the DR listed date + listing price. Listing Ops (Candice Jenkins) emails the agent with pricing advice ("be mindful of listing just over a common hard line buyer ceiling such as $300,000"). MERS registration tasks complete.

**Artifacts.** Simplifile recording confirmation, MERS task, DR listing price + listing date, listing agent email feedback, Listing Surcharge ($0 typical), HOA dues, agent's MLS confirmation.

**Where info lives.** Simplifile, MLS (via agent), Listing Ops mailbox/Slack.

**Blockers.** DR not yet listed (agent stalling), recording errors at county (Tax Exempt missing, missing document images), price misjudgment by agent, listing delays (client wedding, etc.).

**Presentation.** Mostly day-N email logs and recording webhooks. Listing Ops adds the human commentary in long quoted emails.

**Stage-exit signal.** DR goes under contract → AI-reviewed DR Contract block posts → `Email sent: BBYS_DR_in_escrow_payoff_request` → `Stage UPDATED to Dr Closed` on actual COE.

---

## DR_CLOSED (998815822)

**Conversation.** Wrap-up. Sale price, program fee %, COE, HOA, Listing Surcharge, CMF, title company recap. Payoff (PO) is sent. Maintenance Reserve Holdback noted whether utilized or returned. Reconveyance / lien-release filed (loan paid off — sometimes errors out: `Lien Release Failure Alert: Lender or Trustee missing from loan file`). Post-close feedback emails fire (agent + client).

**Artifacts.** Final sale price, program fee (1% DTI Drop / 2.4% standard / variable), Maintenance Reserve Holdback utilization, CMF, T&E company, Payoff document, Reconveyance recording package, MERS registration final, `Homelight Paid Date`.

**Where info lives.** Simplifile (lien release), Meridian Link (close-out), Sales App lead detail.

**Blockers.** Reconveyance can't fire because lender/trustee field missing in Meridian Link; payoff lost in transit; closing attorney requests check instead of wire; HOA verification.

**Presentation.** PO summaries are dense bullet blocks tagged to the Client Manager + Lender Ops.

**Stage-exit signal.** Terminal stage — deal is done.

---

## CROSS-STAGE PATTERNS

### Universal bots and what they signal
- **`HomeLight Sales App`** — single firehose source-of-truth bot; renames the channel on every stage move (`-new- → -rev- → -appr- → -as- → -iruc- → -ctc- → -irx- → -drx-`, or `-term-` on failure). Stage transitions always quote the actor: `[Person]: Stage UPDATED to [Stage]`.
- **`HomeLight Homie`** — confirms the Client Welcome Onboarding email was sent by the LO (a key NEW-stage gate).
- **`HomeLight Hub`** — joins channels just to populate deal context; no chatter.
- **`TitleBot`** — pulls owner / vesting / legal description / year-built at intake.
- **`Solar Panel Detection`** — fires at intake with Google Maps + annotated image.
- **`Easy Automations Service Account`, `Zapier`, `Zapier - Post-Contract Email Bot`** — handoff emails and DocuSign client info sheets.
- **`Prelim Funding Confirmation`** — funding webhook bot active in CTC stage.

### Universal artifacts referenced across all stages
Sales App lead URL (`sales-app.homelight.com/buy-before-you-sell/leads/{id}`); Google Drive "Deals folder"; Extend.ai workflow runs (`dashboard.extend.ai/workflows/...`) for OCR of IR/DR contracts and asset docs; Meridian Link loan file (under EDocs); Snapdocs closing UUID; Simplifile recording package URL; isoftpull credit reports; `equity-app.homelight.com/client/*` and `equity.homelight.com/bbys/document-signing/{id}` client-facing surfaces.

### Standard channel topic
Always: `Client: [name] // LRM: [name] // Lender Ops: [name] // LO: [name] // BBYS (Lender Platform) // [Sales App URL]`. After agreement-signed, listing-ops names get added.

### Role-by-stage ownership
- **LRM** (Ashlee Kim, Ian Pardo, Barb Griego, Angelica Espinosa): owns NEW + IN_REVIEW + APPROVED handoff; LO-facing.
- **Lender Ops** (Michelle Guarino, Jessie Guiang): underwriting, photo review, agreement generation.
- **Jake Vogel**: variable-pricing / fee-exception approvals.
- **Client Manager / Listing Ops** (Patricia Pinckard, Cheryl Funk, Melissa Garcia, Derek Lupien, Brent Lehman, Penny Nierras, Madeline Cypert, EJ Cendana, Candice Jenkins): joins at IR_IN_ESCROW; owns the rest.
- **Sarah Coyne, Renee Cook**: funding / warehouse.
- **John Baltazar, Johnna Gallego, Ann Torres, Rhyna Coloma**: lender CTF / funding tasks.

### Common templates
- *Pending items* (LRM↔Lender Ops): `<@person> Pending photos / Pending email for [name] / Confirm lien balance / lien info`.
- *CA Notes block* (Client Manager, IR_IN_ESCROW intake): `IR Address / COE / Financing / EU Amount / Contingency Removal Only / Second Lien / Solar / Trust / HOA / Situation`.
- *UC Notes block* (Client Manager, rolling status, IR_IN_ESCROW → CTC): strikethrough items as they complete (`~-Inspection - 5/9~`, `-EUID - not sent`, `-CTC?`).
- *Property Condition status*: structured `Satisfied / Open` lists with `pre funding - required repair - [item]` and `pre hl purchase - required repair - [item]`.

### Emoji vocabulary
`:alert::alert:` (needs EB), `:rotating_light:` (already-UC at intake), `:bell:` (rule-matched bot notification), `:eyes:` (DocuSign viewed), `:lower_left_ballpoint_pen:` (signed), `:white_check_mark:` (all signed), `:credit_card:` (soft credit), `:sunny:` (solar detection), `:fire:` (final BBYSA out), `:heart:` (acknowledgement), `:extreme-teamwork:` (LSM pending list).

### Failure modes / archival
Stage moves to `Failed`, channel renames to `*-term-*`, archived. Common reasons: client got alternative funding (HELOC, Social Security award), VA financing conflict, title can't be resolved by COE, LO unresponsive past deadlines.

### The single best "what does this deal need" signal
Read the most recent `HomeLight Sales App: [Person]: Stage UPDATED to [Stage]` line + the most recent Client Manager CA/UC Notes block + any open Inspectify / `BBYS Lender - Additional Document Request` items. Those three together describe the live blocker stack better than any generic checklist.
