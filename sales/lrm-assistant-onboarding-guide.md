---
date: 2026-06-02
type: sop
owner: Tulli
audience: [lrm-assistant, saint-lucia]
last_reviewed: 2026-06-02
tags: [lrm, lrm-assistant, saint-lucia, priority-queue, bbys, onboarding, sop]
source: HomeLight-Vault/sops/2026-06-02-lrm-assistant-onboarding-guide.md
imported: 2026-09-29
---

# SOP: LRM Assistant — How to Work the Priority Queue

> This guide assumes you've **never worked on BBYS before**. By the time you finish reading it, you'll know what BBYS is, how to open and read a deal, how to email the LO (both manually and with AI assist), how to mark your work complete, and when to snooze a deal.
>
> Read this once end-to-end. Then keep it open as a reference for your first week.

---

## Section 1 — What is BBYS (5 minutes)

**BBYS** stands for **Buy Before You Sell**. It's HomeLight's program that lets a homeowner buy their next home BEFORE they sell their current one. We give them the cash to put down on the new home, secured against their old one. When the old home sells, we get paid back.

### The people in a BBYS deal

| Term | Who they are |
|---|---|
| **Client** | The homeowner. We almost never talk to them directly. |
| **LO** (Loan Officer) | The mortgage broker who sent us the client. They're at an outside company like UWM, Fairway, CrossCountry. **Your main contact every day is the LO.** |
| **LRM** (Lender Relationship Manager) | The HomeLight teammate who owns the deal. Brandi, Ashlee, Deborah, Ian, Kyle, Angelica, or Carter. **You're helping the LRMs by re-engaging quiet deals.** |
| **You** (LRM Assistant) | Re-engage deals that have gone quiet, so the LRMs can spend their time closing the active deals. |

### The two homes in every deal

- **IR** = **Incoming Residence** — the new home the client wants to buy
- **DR** = **Departing Residence** — the old home they're selling

When you see "IR address" on a card, that's the new home. The deal name is usually formatted as `IR street - city - state - client name`.

### The money

- **EU** = **Equity Unlock** — how much cash we're advancing
- **EB** = **Equity Boost** — extra cash beyond standard EU (rare)
- **COE** = **Close of Escrow** — the closing date

### The stages a deal moves through

Left to right is "good progress":

```
New → In Review → Approved → Agreement Signed → IR In Escrow → Clear to Fund → IR Closed → DR Closed
```

**You only work the first 4 stages** (New, In Review, Approved, Agreement Signed). Once a deal goes to IR In Escrow ("under contract"), a different team takes over.

### Your job in one sentence

> If a deal has been sitting in the same stage for a few weeks with no recent activity, **wake it back up** by reaching out to the LO.

---

## Section 2 — Opening the queue

1. Go to `https://app.homelight.com/queue` (bookmark this)
2. Sign in with your HomeLight Google account
3. The page should auto-load to the **LRM Assistant** view — the header reads **"Priority Queue · LRM Assistant — Aged Deals"**

If the header says anything else, look for a dropdown near the top that says "Role" or "View" — switch it to **LRM Assistant**.

If you don't see anything, your access isn't set up yet. **Slack Tulli.**

### What you see

A list of deal cards stacked top-to-bottom, numbered 1, 2, 3… The deal at **position #1 is the most important deal to work first.** You don't have to choose — the queue has already sorted them for you.

You and your teammate **share the same list**. There's no "this deal is yours, that deal is theirs". You'll see exactly the same cards. (We'll cover how to avoid working the same deal twice in [Section 7](#section-7--working-as-a-team).)

---

## Section 3 — Reading a deal card

Each card looks roughly like this:

```
1   ┌────────────────────────────────────────────────────────────┐
    │ P0 · 125  Long stale (6+ wk) · 57d in stage    [Approved] │
    │ Likely stalled · Target equity hit · LO quality — mid +5  │
    │                                                            │
    │ → Call Mark Hairston to push for agreement signature       │
    │   Why now: 57d in Approved (operator typical 5d)           │
    │   First step: 📞 Call works best — Approved deals need     │
    │                a real conversation                          │
    │                                                            │
    │ ⏱ 57d in stage · 📅 Last touch: 1d ago    📍 COE: 5/17/26  │
    │ 412 Buck Ridge Rd — Cedar Park — Don Stough               │
    │   🏠 $0.86M  💰 $325,097 EU  ✓ $250K target met  📊 24% conv│
    │ 🏛 Mark Hairston · Texas Mortgage Source LLC               │
    │   📞 (512) 789-6967  📧 mark@markhairston.com              │
    │                                                            │
    │ • Touch History (4)                                        │
    │                                                            │
    │ [📧 Email LO] [✨ Draft AI Follow-up] [📞 Call LO]         │
    │ [👁 View Deal] [✉️ Reach-out completed] [z Snooze 7d]      │
    └────────────────────────────────────────────────────────────┘
```

Here's what each piece means.

### Top row

| Element | Meaning |
|---|---|
| **`1`** (left of card) | Position in the queue — work top to bottom |
| **`P0 · 125`** | **P0** = freshest aged deal (last 2 weeks in stage). **P1** = 2–6 weeks in stage. **P2** = 6+ weeks in stage. The **125** is the priority score — higher = more urgent. |
| **`Long stale (6+ wk) · 57d in stage`** | How long this deal has been sitting at the current stage |
| **`Likely stalled · Target equity hit · LO quality — mid +5`** | The top 3 reasons this deal is in your queue. **+5** means 5 more reasons are clickable. |
| **`Approved`** (right side) | The deal's current stage |

### What to do action

```
→ Call Mark Hairston to push for agreement signature
  Why now: 57d in Approved (operator typical 5d)
  First step: 📞 Call works best — Approved deals need a real conversation
```

This is the AI's suggested next action. **Read it but don't blindly follow it.** Always cross-check against the deal popup before deciding (Section 4).

### Metadata row

| Element | What it tells you |
|---|---|
| **`⏱ 57d in stage`** | Days the deal has been sitting at the current stage |
| **`📅 Last touch: 1d ago`** | When the most recent touch was — across email, SMS, calls, and Slack |
| **`📍 COE: 5/17/2026`** | Close of Escrow date if known. Red if past-due. |

### Deal line

```
412 Buck Ridge Rd - Cedar Park - Don Stough
  🏠 $0.86M  💰 $325,097 EU  ✓ $250K target met  📊 24% conv
```

| | |
|---|---|
| 🏠 | DR (old home) value |
| 💰 | Equity Unlock approved |
| ✓ | "Target met" — we approved at least as much as the client asked for. Good signal. |
| 📊 | BBYS application conversion probability (an LO-quality signal, not a deal-outcome prediction — don't over-index on it) |

### LO line

```
🏛 Mark Hairston · Texas Mortgage Source LLC
  📞 (512) 789-6967  📧 mark@markhairston.com
```

The LO's name, company, phone, and email. You'll use this every time you reach out.

### Last touch chip (top-right corner of the card)

Look for a small chip next to the stage badge that looks like:

```
📤 Last: outbound slack · Automation (HomeLight Sales App) · 1d ago
```

This tells you:
- **Direction** — `📤 outbound` = we reached out, `📥 inbound` = they reached out to us
- **Channel** — `slack`, `email`, `sms`, `call`
- **Actor** — `Automation` (a bot like HomeLight Sales App), `Deborah Shutt` (a teammate), or `Unknown` (someone external)
- **How long ago**

> If the chip and the **`📅 Last touch:`** date row don't agree (e.g. chip says 1d, date row says 1w), the page just hasn't finished refreshing. Wait 5 minutes and reload. If still wrong after a day, tell Tulli.

---

## Section 4 — Working a deal, step by step

Plan to spend **5–10 minutes per deal**. Don't rush; the win is making each touch count.

### Step 1 — Click the card

The deal popup slides in from the right and fills most of the screen.

### Step 2 — Read the left panel (Associated Contacts + Internal Team)

```
Loan Officer
  Mark Hairston · Entrepreneur
  mark@markhairston.com
  +15127896967

Client
  Don Stough
  dstough60@gmail.com
  (512) 413-6111

Internal Team
  LSM: Tejas
  LRM: Deb   ← this is who you'd escalate to in Slack

Companies
  Texas Mortgage Source LLC

Activity (65 items)
  Including LO-tagged activity from the past 14 days
  ✨ Summarize with AI
  ⤿ recent email / SMS / call rows
```

**Things to notice:**
- The **LRM line** tells you who owns the deal. If you need to escalate, you'll tag this person in Slack.
- **Activity (N items)** — the recent communication history. Scroll through the last 5–10 to get a feel for what's been said.

### Step 3 — Read the middle panel (Stage Tracker + Synced Fields)

```
[NEW] [IN REVIEW] [APPROVED] [AGREEMENT SIGNED] [IR IN ESCROW] [CLEAR TO FUND] [IR CLOSED] [DR CLOSED]
 Apr 1  —          Apr 5                                                                              
```

The stage row shows when the deal entered each stage. Empty entries mean the deal hasn't gotten there yet.

Below that you'll see things like:
- **Departing Residence Address** — the old home
- **Client Name**
- **HomeLight Value / LO Value / Total Appr. EU + EB + HELOC**
- **Max Approved EU / Max Approved Equity Boost**
- **LO's Application Number**

You don't need to memorize these. **The two things to look at:**
1. **Stage entered dates** — how long the deal has been stuck
2. **Anything red, blank, or weird** — missing fields might explain why the deal stalled

### Step 4 — Read the right panel (Deal Slack)

This is the most important panel. The deal's Slack channel is shown in real time. Scroll up to see recent messages.

**Look for these patterns:**

| What you see | What it means | Your move |
|---|---|---|
| Recent message from **Deborah Shutt / Ian / Kyle / etc.** (a teammate) saying "Per LO: …" | The LRM already talked to the LO. The note tells you what the LO said. | **Don't re-touch the LO yet.** The deal will resurface in your queue automatically. Move to the next card. |
| Message from **HomeLight Sales App** saying "email sent" / "Called loan officer" | An automated system already sent something | Check what was sent. If it was recent and the LO might respond, give it a day. |
| Message from **HomeLight Sales App** saying "Its been 14 business days and the Loan Officer has not sent the agreement" | Reminder ping, not a touch — the agreement is still pending | **Time to push the LO.** Send an email. |
| Message from **the LO themselves** (someone outside HomeLight) | The LO actually replied | **Read what they said.** Adjust your outreach accordingly. |
| Message about **document upload / signed agreement / etc.** | Deal status update | Cross-reference with the stage tracker. |

> **Common gotcha:** if the most recent message is from a teammate saying *"Per LO: Hold on sending DS"* (DS = DocuSign), the LO has asked us to pause. **Do not email the LO.** Move on.

### Step 5 — Decide your action

You have three real choices:

1. **Email the LO** — primary action. Section 5.
2. **Call the LO** — only if you have Aircall access (Section 7), and only when email won't work. Otherwise escalate to the LRM.
3. **Escalate to the LRM** — when the deal needs a phone call from someone with relationship context, or when there's a sensitive issue (client compliance, repair dispute, agent conflict).

If escalating:
- Open Slack `#homes-bbys-pod`
- Tag the LRM listed on the card (e.g. `@deborah`)
- Write a 2-sentence brief: what the deal needs + what you saw in the channel that prompted you
- Example: *"@deborah — 412 Buck Ridge has been in Approved 57 days, LO Mark hasn't responded to the last two emails. Could you give him a call?"*

---

## Section 5 — Sending an email (Email LO button)

This is your most-used action. The Email LO drawer is a real email composer that sends through Gmail and **automatically logs the email to HubSpot** so the LRM can see it.

### Walkthrough

1. On the queue card, click **`📧 Email LO`**
2. A drawer slides in from the right. You'll see:
   - **To:** field — pre-filled with the LO's email
   - **CC:** field — empty by default. Add the LRM if it's a deal that the LRM should be aware of (see "When to CC" below)
   - **Subject** field — empty
   - **Body** field — large text area
   - **Template** dropdown — pre-written email shells for common situations (intros, agreement nudges, COE check-ins). Pick one as a starting point.
   - **Send** button at the bottom

3. Pick a template if relevant. **Always edit it** — never send the template verbatim. The LO can tell. Personalize:
   - Use the client's first name in context if you have it
   - Reference a specific date or fact from the channel ("I saw you got the agreement back on May 4 — any update on signing?")
   - Don't write more than 4–5 sentences

4. Click **Send**

The email goes through your Gmail. The LO sees a real email from you. HubSpot logs it as outbound activity on the deal. The queue updates the "Last touch" within ~30 seconds.

### When to CC the LRM

- ✅ CC the LRM when the deal looks like it needs an LRM follow-up after you (escalation by email)
- ✅ CC the LRM if you're sending information they specifically need to see
- ❌ Don't CC them on every email — they'll start ignoring the channel

### What NOT to do

- ❌ Don't ask for personal info (SSN, account numbers) over email
- ❌ Don't make promises about timeline or approval amounts — those decisions belong to the LRM
- ❌ Don't send the same template to the same LO twice in a week — vary your angle
- ❌ Don't write a wall of text. 4–5 sentences max.

---

## Section 6 — Using AI Draft (✨ Draft AI Follow-up button)

This is the **smart version** of Email LO. The AI reads the deal's history (recent Slack messages, prior emails, Coach's read, matched Creative Scenarios) and drafts an email for you. You land on a partial draft, not a blank.

### When to use it vs the regular Email LO

| Use AI Draft when… | Use plain Email LO when… |
|---|---|
| You're unsure what angle to take | You know exactly what you want to say |
| The deal has matched "creative options" (amber pill on the card) | The deal is a simple "where are we?" check-in |
| You want the AI to pull in a specific recent fact from Slack/email | You have a template that's right for the situation |
| You're early in your shift and want speed | You're at the end of your shift wrapping up |

### Walkthrough

1. On the queue card, click **`✨ Draft AI Follow-up`**
2. The Email LO drawer opens — same as Section 5 — **but the AI immediately starts drafting** (you'll see a spinner for 5–10 seconds)
3. When it finishes, you'll see:
   - The **Subject** pre-filled
   - The **Body** pre-filled with a draft email — usually 4–6 sentences
   - An "Add context for the AI" text box near the top, showing what the AI was given as context. You can edit this and click "Regenerate" if the draft missed the mark.

4. **Always edit the draft before sending.** The AI is good but not perfect. Things to check:
   - **Tone** — does it sound like you? Adjust word choice.
   - **Facts** — does every claim match what you saw in the deal? Wrong facts kill trust.
   - **Length** — too long? Cut it.
   - **Ask** — does the email have ONE clear ask? If there are 3 asks, the LO will answer none.

5. If the draft is way off, click **Regenerate** — sometimes the second pass is better.

6. Click **Send**.

### What the AI knows about

The AI reads:
- The deal's HubSpot fields (stage, EU, COE date, LO + client info)
- The last ~30 days of emails sent on the deal
- The recent Slack channel messages (real content, not just timestamps)
- Any **Creative Scenarios** that matched this deal (visible as the `✨ N creative options` amber pill)
- The Coach's read summary

### What the AI is NOT allowed to suggest

- HubSpot updates ("update the deal to Approved")
- Slack pins or internal team actions
- Calendar invites
- Anything the AI can't actually do from your seat

If you see the AI suggest these things, **don't follow them and tell Tulli with the deal ID** — that's a prompt bug.

---

## Section 7 — Using Aircall (when you're added)

> **In v1 of the LRM Assistant pivot, you don't have Aircall access yet.** Calls escalate to the LRM via Slack. Once you're added to Aircall, this section applies.

### How Aircall is wired up

Aircall is HomeLight's phone system. When you're added, you'll have:
- A HomeLight phone number assigned to you
- The Aircall dialer embedded in the admin app — pops up when you click `📞 Call LO`
- Automatic call logging to HubSpot (no manual entry needed)

### Walkthrough (once you have access)

1. On the queue card, click **`📞 Call LO`**
2. The Aircall dialer opens in a small pop-up with the LO's number pre-filled
3. Click the green call button
4. Talk to the LO. Stick to the action the queue suggested + anything you noticed in the channel.
5. When the call ends, Aircall shows a quick "Wrap-up" screen:
   - **Status** — Connected / Voicemail / No answer / Wrong number
   - **Notes** — short notes about what was said. Keep them brief but useful — the next person who works the deal will read them.
   - **Tags** (optional)
6. Save the wrap-up. The call is logged to HubSpot automatically.

### Phone etiquette

- Say your full name + that you're calling from HomeLight on behalf of `<LRM name>`
- State the deal up front: *"Calling about Don Stough — 412 Buck Ridge — we have him in Approved and wanted to check in on the agreement signature."*
- If you reach voicemail, leave a 20-second message + send a follow-up email immediately

### What to do if Aircall isn't working

- Refresh the page
- Try once more
- If still broken, escalate by email (Section 5) and Slack Tulli — don't sit there fighting the dialer

---

## Section 8 — Marking a reach-out complete (✉️ Reach-out completed button)

**You MUST click this after every contact attempt.** It's how your work gets tracked and how the queue knows to stop showing the deal.

### When to click it

- You sent an email
- You made a call (even if voicemail / no answer)
- You sent an SMS (if you have that capability)
- You sent a follow-up across multiple channels (mark all that apply)

### What NOT to click it for

- Just opening the deal popup without contacting anyone
- Escalating to the LRM in Slack (the LRM logs their own touch when they reach out)

### Walkthrough

1. After your email sends (or call ends), click **`✉️ Reach-out completed`** on the deal card
2. A modal opens with channel chips:
   - **📞 Call** — check if you called
   - **✉️ Email** — check if you emailed
   - **💬 SMS** — check if you texted
   - You can check multiple — common pattern is **Email + Call** for "I left a voicemail then sent an email"
3. (Optional) **Note field** — one sentence about what happened, especially if anything's unusual
   - *"LO said hold off until next week — client is travelling"*
   - *"Voicemail; sent recap email with COE check-in"*
   - Leave blank for routine touches
4. Click **Submit**

### What happens after you submit

- An invisible **3-day snooze** kicks in just for you — the deal disappears from YOUR queue. (Your teammate still sees it.)
- A bot posts a short summary to the deal's Slack channel:
  ```
  👋 *Janice* logged a reach-out
  Channels: *Email + Call*
  Note: LO said hold off until next week — client is travelling
  ```
- The LRM sees the Slack post and knows you took action
- Your work shows up on the lift dashboard (managers see this — see Section 11)

If the deal needs another touch after 3 days, it auto-reappears.

---

## Section 9 — When and why to snooze (z Snooze 7d button)

Snoozing tells the queue "I've looked at this deal and I want it to come back later." **Only you stop seeing it** — your teammate still sees the deal on their queue.

### When to snooze

| Situation | Use this |
|---|---|
| You're about to email an LO and don't want your teammate to also email them at the same time | **Snooze 1 day** (use the dropdown). Stops double-touch. |
| The LO told us they're OOO until next week | **Snooze 7 days** |
| The deal has a scheduled closing 5 days from now — wait for COE | **Snooze 5 days** |
| You contacted the LO and they need a few days to gather info | The auto-3-day from "Reach-out completed" handles this — don't snooze on top |

### When NOT to snooze

- ❌ You don't feel like working the deal today — just skip it; the queue's order doesn't change because of YOUR snoozes
- ❌ The deal is annoying — the LRM and managers can see what you're hiding via the audit page
- ❌ You think the deal should be killed entirely — use **Remove from Queue** instead, OR escalate to the LRM

### What "Snooze 7d" actually does

- The deal disappears from your queue for 7 days
- It auto-reappears on day 8
- Your teammate is unaffected — they still see the deal normally
- A small banner appears at the top of your queue: **"Hidden by you (3)"** — click it to see what you've hidden and restore any item

### Remove from Queue (the nuclear option)

Below the Snooze button there's an `⋮` menu with **Remove from Queue**. Use this only when:
- The deal genuinely doesn't belong in your queue (e.g. you can see the LRM is actively working it solo and doesn't need help)
- You're confident your teammate also shouldn't see it (but remember, this only hides it for YOU)

Restore via the "Hidden by you" banner.

---

## Section 10 — What "Last touch" actually counts

Important to understand because it affects when deals come back to your queue.

**These count as "we reached out" (outbound touches):**
- Email we sent (logged in HubSpot)
- Call we placed (logged in Aircall → HubSpot)
- SMS we sent
- A Slack message in the deal channel from anyone on the HomeLight team (any LRM, LSM, Lender Ops, Operator)
- An automated Slack post from "HomeLight Sales App" that confirms an action — e.g.
  - *"BBYS Approval Expiration Refresh Confirmation email sent"*
  - *"System: Called loan officer (Buy Before You Sell lead)"*
  - *"System: Emailed loan officer (Buy Before You Sell lead): refreshed EU/send DS?"*

**These DON'T count as outbound:**
- Status updates in Slack ("Stage UPDATED to Approved", "channel joined", "TitleBot pulled")
- Reminder pings ("Its been 14 business days and the LO hasn't sent the agreement") — these are nags, not touches
- Inbound messages from the LO / agent / client (they're real activity but they don't reset YOUR cooldown timer)

### Why this matters for you

> If you see a teammate's Slack note from 2 days ago saying *"Per LO: Hold on sending DS"*, that **counts as a touch**. The deal is correctly NOT in your queue right now. When the cooldown expires it'll resurface.

If you DO see a stale-looking deal in your queue (says "Last touch: 2w ago" but Slack has a teammate note from yesterday), **refresh the page once**. The two indicators sometimes lag by a few hours but should align within a day. If they don't align after 24h, tell Tulli the deal ID.

---

## Section 11 — How your work is measured (Lift dashboard)

You don't need to look at this — your manager will. But you should know it exists.

Every time you click **Reach-out completed**, a row is logged. Managers see:

- **Total reach-outs** (today / 7d / 30d)
- **Moved forward** — deals that progressed to a later stage after your touch
- **Converted** — deals that reached DR Closed after your touch
- **Failed** — deals that died after your touch (not necessarily your fault — the cause is investigated)
- **Still in stage** — deals that didn't move

The metric that matters most is **moved forward** — your job is to wake deals up so they progress.

**Implication for how you work:** every reach-out should have a clear ask. *"Just checking in"* emails don't move deals.

---

## Section 12 — End of shift (5 minutes)

Before you log off:

1. Send a 2-bullet summary in your assigned Slack channel:
   - **Deals touched today** — count + 2–3 standouts (deal name + what you did)
   - **Anything escalated to an LRM** — link the Slack thread

2. Clear any "Hidden by you" items you don't intend to keep hiding (click the amber banner)

3. Close the queue. (No "log out" action needed.)

---

## Section 13 — Common situations

**Q: Two of us touched the same deal because we didn't notice.**
A: Use the "Snooze 1 day" trick BEFORE you start emailing — that gives you a window to work the deal exclusively. If it still happens occasionally, no harm done — just don't both send emails in the same hour. Try to scan top-of-list together at the start of shift.

**Q: I'm on a deal where the LO never responds to anything. What do I do?**
A: After 3 contact attempts spread over 10–14 days with no response, escalate to the LRM. The LRM may decide to pause the deal or have someone senior step in.

**Q: I see a deal that should be killed (client is no longer interested).**
A: Don't kill it yourself. Slack the LRM in `#homes-bbys-pod` with the deal name + reason. They'll move it to Failed or Nurture in HubSpot.

**Q: The "Coach's read" text says "generating…" forever.**
A: Reload the page once. If still missing, tell Tulli.

**Q: The AI Draft suggested I do something inside HomeLight (update HubSpot, pin a Slack message, etc.).**
A: Don't follow it. Send a normal email instead and tell Tulli the deal ID — the AI is supposed to suggest LO-facing actions only.

**Q: I see a "+N creative options" amber pill on a card. What is that?**
A: The system found this deal might be a candidate for the BBYS Exception Playbook (e.g. layering HELOC on top of EU). Click it to see the matched scenarios. Use **Draft AI Follow-up** — the AI will weave the scenario language into the email.

**Q: A deal I closed (moved to IRX or DR Closed) is still in my queue.**
A: It'll fall off within 15–30 minutes. If it's still there after 2 hours, ping Tulli.

**Q: I see deals from LRMs other than the one I work most with — is that wrong?**
A: No. The queue is org-wide. The "LRM:" line is informational — telling you who to escalate to. You can touch any deal in the queue regardless of LRM.

**Q: What's the difference between "Snooze" and "Reach-out completed"?**
A: Reach-out completed = "I did the work, hide for 3 days while LO responds". Snooze = "I haven't done the work yet, hide because [reason]". Don't substitute one for the other.

---

## Section 14 — Escalation paths

| Situation | Where to go |
|---|---|
| **LO needs a phone call** | Tag the LRM on the card in Slack `#homes-bbys-pod` with a 2-sentence brief |
| **Client compliance or sensitive issue** | Brandi Cirell (LRM Lead) or Jake Vogel (Head of Lender Relations) |
| **Tool / workflow / queue bug** | Tulli in Slack DM or `#homes-data-bridge` |
| **AI Draft producing bad output for a specific deal** | Tulli with the deal ID |
| **You don't know what to do** | Tag your teammate first. Then Tulli. |
| **Aircall not working** | Email the LO as fallback, then Slack Tulli |

---

## Section 15 — Quick reference cheat sheet

**Daily flow:**
1. Open the queue
2. Work top → bottom
3. For each deal: read card → open popup → read activity + Slack → decide → act → mark Reach-out completed
4. End of shift: post summary in Slack

**Buttons on a deal card:**
- **📧 Email LO** — plain email composer
- **✨ Draft AI Follow-up** — AI pre-drafts the email
- **📞 Call LO** — Aircall (when you have access)
- **👁 View Deal** — open the deal popup
- **✉️ Reach-out completed** — log the touch (3d auto-snooze) — **CLICK AFTER EVERY CONTACT**
- **z Snooze 7d** — manual hide for 7 days (or use dropdown for 1–30)

**When to snooze:**
- Snooze 1d — before starting an email, to avoid teammate double-touch
- Snooze 7d — LO is OOO
- Auto-3d — happens automatically after "Reach-out completed"

**When NOT to follow the AI:**
- AI suggests editing HubSpot
- AI suggests an internal HomeLight action
- AI cites a fact you can't verify in the deal

---

## Owner & Review

- **Owner:** Tulli (Nicholas Santulli, Head of Revenue Operations)
- **Audience:** Saint Lucia LRM Assistant team
- **Review cadence:** Weekly during the first month, then monthly
- **Created:** 2026-06-02
- **Pilot start:** 2026-06-01

## Related notes

- BBYS product overview: [[bbys-overview]]
- Slack channel vocabulary (the acronyms you'll see in deal channels): [[bbys-deal-channel-vocabulary]]
- Deeper operational reference (for Tulli + managers, not for assistants): [[lrm-assistant-saint-lucia-priority-queue]]
- Team roster: [[team]]
