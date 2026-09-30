---
date: 2026-05-13
type: sop
owner: Tulli
last_reviewed: 2026-05-14
tags: [bbys, priority-queue, operator-guide, lsm, lrm]
source: HomeLight-Vault/sops/2026-05-13-priority-queue-operator-guide.md
imported: 2026-09-29
---

# SOP: Using the BBYS Deal Priority Queue

## Purpose

The Priority Queue is a **smart, ranked to-do list of deals** for LSMs and LRMs. Instead of scrolling the HubSpot pipeline view and guessing which deal to touch next, the queue tells you:

1. **Which deals are most urgent** (ranked, with a P0 / P1 / P2 / P3 tier)
2. **Why each deal is urgent** (the firing signals — see below)
3. **What to do next** on each deal (a one-sentence AI-generated next action)

It refreshes itself automatically as deals move stages, get touched, or go quiet. You don't need to re-run anything.

## When to Use

- **Every morning** — start your day at the top of your queue
- **After a long meeting block** — re-check what shifted while you were heads-down
- **Before logging off** — make sure no P0 slipped through the day
- **When you don't know what to work on** — the queue is the answer

## Where to Find It

Admin Console → **Priority Queue** (left sidebar). Or direct: `/admin/queue`.

## How the Queue Works (Plain English)

### Two queues per deal — yours, automatically

Every active BBYS deal scores **twice** — once for the LSM, once for the LRM, because they care about different things. The page auto-detects your role from your team profile:

- **If you're an LSM** → you see the LSM queue, no role picker shown
- **If you're an LRM** → you see the LRM queue, no role picker shown
- **If you're a manager / admin / SRM** → a Role dropdown appears so you can switch between the two views

### What gets ranked — role-aware (updated 2026-05-13)

Active stages are now **role-specific**:

- **LSM scores: New, In Review, Approved, Agreement Signed** (4 stages — LSM hands off when deal goes Under Contract)
- **LRM scores: same 4 pre-UC stages as LSM** (updated 2026-05-14 PM-2: also dropped IRUC per operator rule "once a deal is Under Contract, the closing-agent (CA) team owns it — LRM can't materially move it forward"; cumulative same-day shrink: 7 AM → 6 (drop IR Closed) → 5 PM (drop CTF) → 4 PM-2 (drop IRUC). **Both roles now share the same 4-stage allow-list.** Role-aware UNION in the orphan-delete RPC is preserved so they can diverge again if rules change. The "if [other team] now owns the deal, drop it" rule fired 3× in one day — IR Closed → CTF → IRUC.)
- **Both roles exclude: DR Closed, Failed, Nurture, test leads, duplicate flags, orchard non-UC deals**

This means an LSM sees only deals they actually own (no fake-urgent IRUC deals from the LSM's old view). LRMs see further down the funnel because they're involved longer.

### How scoring works

Each deal accumulates **points** from any signals that fire on it. Points come from a configurable weights table (operator-tunable — see "Tuning weights" below).

The total score maps to a **tier:**

| Tier | Score range (default) | What it means |
|------|----------------------|---------------|
| **P0** | 80+ | Work today, ideally now |
| **P1** | 60–79 | Work today |
| **P2** | 40–59 | Work this week |
| **P3** | 0–39 | Background priority |

The thresholds are tunable via `/queue/weights` — adjust if everything ends up in the same tier.

### The 8 + N signals that drive scoring

Some signals are about **deal mechanics** (stage, equity, timing). Others are about **engagement patterns** (who's emailed who, who's gone quiet, who clicked an email). Below is the current list grouped by source. The "Default points" is the starting weight; operators tune them via `/queue/weights`.

#### Mechanics signals (Phase 1 + 2)

| Signal key | Fires when | Default points |
|------------|-----------|---------------:|
| `under_contract` | Deal is in IRUC or CTF stage AND has an IR contract date — OR `manual_uc_override = true` | 50 |
| `uc_stagnant_48h` | Under-contract deal hasn't been touched in 48h+ | 25 |
| `stall_at_risk` | Days in current stage > 1.5× typical for that stage | 15 |
| `stall_likely_stalled` | Days in current stage > 2× typical | 30 |
| `target_equity_hit` | Approved equity ≥ requested equity (operator should push closing) | 30 |
| `partner_tier` | Strategic partner deal (UWM / TLS / Fairway / CC / APM / Lennar) | 10 |
| `lo_quality_tier` | LO is Challenger ICP (top tier) | 15 |
| `lo_quality_tier_mid` | LO is Entrepreneur or Learner ICP | 10 |
| `lo_elite_obsidian` / `_diamond` / `_elite` / `_active` | LO carries an Elite badge tier | 5–15 |
| `inactivity` | Deal hasn't been touched in N days (uses `notes_last_updated`) | varies |
| `closing_window_14d` | IR closing date is within 14 days | varies |
| `conversion_probability` | BBYS application conversion probability is high | varies |
| `revenue_potential` | EU/DR ratio < 40% — potential HELOC opportunity (Phase 2 placeholder for true HELOC tracking) | 15 |

#### Engagement signals (Phase 3a Wave 1 — fire on existing data)

| Signal key | Fires when | Default points |
|------------|-----------|---------------:|
| `signal_unresponded_inbound_email` | Latest inbound email is 24h–14d old AND no later outbound email on this deal | 10 |
| `signal_unanswered_inbound_call` | Latest inbound call 24h+ old AND no later outbound call/email/SMS on this deal | 10 |
| `signal_going_dark` | Zero engagement of any direction across email/call/meeting/note/SMS in 10+ days while active | 15 |
| `signal_meeting_no_followup` | Completed meeting in past 48h AND no note logged after | 6 |
| `signal_inbound_surge` | 3+ inbound rows on this deal in last 48h | 15 |
| `signal_outbound_only` | 3+ outbound from us in 7d AND zero inbound in same window | 10 |

#### Engagement signals (Phase 3a Wave 2 — require new HubSpot ingestion)

| Signal key | Fires when | Default points |
|------------|-----------|---------------:|
| `signal_portal_login_unmet` | Borrower logged into portal in last 7d AND no outbound from us within 3 days after | 12 |
| `signal_email_engagement_unfollowed` | Recipient OPENED or CLICKED our email in last 7d AND no inbound from them since | 12 |

**Caveat on email engagement:** only emails sent through the **HubSpot Sales Gmail/Outlook extension or Sequences** are tracked. Plain Gmail sends don't fire this signal. Coverage will be partial until extension adoption is universal — that's a non-issue, the signal just gracefully no-fires for reps without the extension.

### What you see on each card (updated 2026-05-13 — card layout)

The queue switched from a flat table to a vertical stack of rich deal cards. Each card shows top-to-bottom:

1. **Tier · Score badge** + top 3 firing signals as inline pills + (clickable `+N` for more) + **Stage badge** on the right
2. **Next-action sentence** (large, bold) — the GPT-generated CTA, e.g. "→ Call Jeremy now to expedite funding"
3. **Top reasons subtitle** (gray, smaller)
4. **Metadata row:** ⏱ days-in-stage · 📅 last contact · 📍 COE date with days-until (red if past-due)
5. **Financial line:** Deal name (address+borrower) · 🏠 DR value · 💰 EU approved · ✓ "target met" indicator · 📊 BBYS conv % — hidden on **5 stages** where the probability is meaningless: IR In Escrow, IR Closed, DR Closed (effectively converted), Failed, Nurture (dead). CTF intentionally still shows the badge as a funding-time risk reality-check on stuck deals. Backend `signal_conversion_probability` already gates on PRE_UC stages so this is purely defensive UI — `bbys_application_probability` is a stored HubSpot property that retains its last-active value after the deal moves stages, so the badge needed its own gate (PR #185, 2026-05-14).
6. **LO line:** 🏛 LO name · company · 📧 mailto link
7. **💡 Deal summary** — gpt-5-mini 1-2 sentence summary of recent comms (italic amber line). Now sources **real Slack channel messages + activity bodies** via `buildDealConversationContext` (PR #154, 2026-05-13). Previously fed only bare engagement timestamps and produced ~10% generic-text junk; cache-busted via new `deal_summary_hash` column so old frozen summaries clear on next recompute.
8. **✨ Creative options pill** (when applicable) — clickable popover listing matched scenarios from the [Project Getting Creative playbook](#) with why-matched + recommended action + link to the full 24-scenario knowledge article
9. **Action buttons:** 📞 Call LO · 📧 Email LO · 👁 View Deal · ⋮ (Snooze / Remove from queue)

The position number (1, 2, 3...) sits to the LEFT of each card.

**Filter row above the cards:**

| Filter | What it does |
|--------|--------------|
| Owner | Show queue for a specific person, or "All" for org-wide |
| Priority | P0 / P1 / P2 / P3 dropdown (renamed from "Tier") |
| Role | LSM / LRM / All — auto-set + hidden for single-role users; admins/managers default to All |
| Stage | Multi-select: hardcoded to **role-eligible stages** (4 for LSM, 7 for LRM/All) — server-side `?stage=` filter applied to the underlying query, not just to the rendered top-25 (PR #159, 2026-05-13). Previously the dropdown auto-derived from the current top-25 pool, which meant if all visible deals were CTF/IRUC the only choices were CTF/IRUC. |
| Signals | Multi-select OR semantics — "show me only deals firing `going_dark` or `unresponded_inbound_email`" |

(Score min/max numeric range was removed in PR #146 — Priority dropdown covers the band-filter use case.)

### View toggle: Priority Queue ⇆ Stalled Deals (added 2026-05-13)

Next to the page title there's a segmented toggle. **Priority Queue** (default) shows the ranked card list. **Stalled Deals** shows the stall-risk widget that previously lived on the Dashboard. Owner filter applies to both views. View choice persists per-user across sessions.

**Stalled Deals scope (tightened PR #162, 2026-05-13):** the widget + the Monday digest cron now filter by `deal_stage_id IN (New, In Review, Approved, Agreement Signed)` — i.e. **PRE-UC only**. Previously used `ir_closed_date IS NULL AND failed_terminated_date IS NULL`, which leaked 16,571 Failed deals plus 322 Nurture, 261 IR In Escrow, 32 CTF, and 6 DR Closed into the widget because those date columns aren't reliably populated. If a deal is past Agreement Signed, an LSM/LRM is already actively working it via the Priority Queue's Under-Contract treatment — it doesn't need stall-risk radar coverage.

### Per-card actions: Snooze + Remove from Queue (added 2026-05-13)

Each card's `⋮` button opens:
- **Snooze for N days** — disappears from your queue for 1-30 days, auto-reappears after expiry
- **Remove from queue** — hides indefinitely

**Both actions are scoped to YOU.** Other users (your LSM partner, LRMs, managers viewing org-wide) see the deal normally. A "Hidden by you" amber banner appears above the queue when you have any active hides, with a Manage modal listing them and a Restore button per row.

### Call LO + Email LO + ✨ Draft AI Follow-up (updated 2026-05-14)

- **📞 Call LO** opens the in-app Aircall dialer pre-populated with the LO's phone number (from `hub_contacts.phone` / `mobilephone`). Button gracefully hidden when no phone on file.
- **📧 Email LO** opens a **right-side slide-in drawer** hosting the canonical `EmailComposerPanel` — the same composer the deal popup uses (templates, ✨ AI Draft, field inserter, send-via-Gmail, **HubSpot logging**). LO is pre-seeded as the recipient. Replaced the old `mailto:` link in PR #166 (2026-05-14) so emails now log to HubSpot Activity instead of dumping the rep into Apple Mail. Closes via X button, ESC, or backdrop click.
- **✨ Draft AI Follow-up** opens the same drawer **AND auto-fires AI Draft** on mount with a context string built from `deal.next_action` + matched Creative Scenarios. Rep lands on a draft, not a blank composer. The auto-context goes into the visible "Add context for the AI" textarea so the rep can see what was sent and tweak + regenerate. Particularly powerful on cards with matched Creative Scenarios — the AI weaves the scenario language directly into the draft (the operator-suggested next step + scenario titles + recommended actions). Cards with matched scenarios show a `+N options` chip on the button.

### Creative Options popover (added 2026-05-13)

When a deal triggers any of 7 auto-detected scenarios from the BBYS Exception Playbook (program fees ≥$1M, EB above 85% CLTV, no-IR-purchase, FHA/VA/Jumbo/Construction loans on IR), an amber `✨ N creative options` pill appears in the top badges row. Click to see the matched scenarios + recommended actions + link to the full 24-scenario playbook. Same panel appears prominently in the deal popup with "before rejecting" framing.

## When the Queue Refreshes

You don't have to do anything to refresh — three layers do it automatically:

1. **Real-time (seconds)** — every time a deal property changes in HubSpot (stage moves, agreement signed, last-contacted updates, manual UC override toggled), HubSpot pings our system and that one deal re-scores immediately. Webhook URL: `/hub/webhook/deal-priority-recompute`.
2. **Every 15 minutes** — a sweep checks for any deals whose data updated but didn't trigger a real-time ping (engagement rows, portal events, email events). Re-scores those. Cron: `cron-priority-score-catchup`.
3. **Every night at 6am AZ** — a full re-score of every active deal. Cron: `cron-priority-score-nightly`.

If a single deal looks stale, click **Recompute now** (admins only) to force a single-page recompute.

## Settings → Backend → Priority Queue (added 2026-05-14)

The previously-hidden `/queue/weights` URL is now surfaced under **Admin Console → Settings → Backend → Priority Queue**. Three pages:

| Page | Purpose |
|------|---------|
| **Signal Weights** | Edit every signal's points value + tier thresholds (P0/P1/P2/P3). Same controls as the old `/queue/weights` URL — that URL still works as a redirect. **Recompute Now** button in the header forces a full org-wide re-score (admin only); skips the wait for the next /queue load past the 6h staleness window. **Last Edited** column shows who tuned each row + when (hover for full ISO timestamp). Save button stays disabled until the row is dirty so you can't accidentally bump `weights_version` with a no-op save. |
| **Weights History** | Changelog of every weights edit — when, signal_key, what changed (`points: 10 → 25`, `enabled: off → on`), who. Backed by an AFTER UPDATE Postgres trigger that captures the diff automatically. No-op saves are skipped. Edits made before 2026-05-14 don't appear (trigger added that day). |
| **Hidden Deals Audit** | Admin-only view of every active per-user snooze + indefinite remove across the org. Joined with user display name + freshest deal name from `hub_deals`. Lets a manager answer "is anyone burying a deal that should be worked?" without View-as-ing each LSM/LRM. **Read-only by design** — to surface a buried deal, ask the user to clear it from their own "Hidden by you" banner on the queue page. Preserves attribution (manager doesn't silently override a user's hide). |

## Tuning Weights

The operator UI for editing every signal's points value, threshold, and on/off state lives at **Settings → Backend → Priority Queue → Signal Weights** (or directly via `/queue/weights` which redirects there).

### When to tune
- After 1–2 weeks of real usage — once you've seen what kinds of deals end up in P0
- If everything is P0 / P3 (tier thresholds need adjusting)
- If a signal fires too often without being actionable (lower its weight or disable)
- If a signal never fires when you wish it would (raise its weight or disable a competing signal)

### How to tune (UI)
1. Open `/queue/weights`
2. Find the signal row (rows are alphabetized; threshold rows have the `threshold_*` prefix)
3. Edit the points value or toggle Enabled
4. Save
5. The next re-score (within ~15 min) will use the new weights

### Optimistic-lock safety
If two operators edit the weights table at the same time, the second save will get a "Weights changed since you loaded the page — please reload" error. This is a feature, not a bug — prevents one operator from accidentally clobbering another's tuning.

## The "Manual UC Override" Field

Every BBYS deal in HubSpot has a checkbox property called **Manual UC Override** (`manual_uc_override`).

- **Unchecked (default)** → deal scores as Under Contract only when stage is IRUC or CTF AND IR contract date is set
- **Checked** → deal scores as Under Contract regardless of stage
- **Use cases:** force a deal into the high-priority "work this daily" treatment when the stage hasn't caught up yet, or suppress UC scoring on a deal that's stuck in IRUC for a known reason

The toggle re-scores the deal within 30 seconds via the property-change webhook.

## Kill Switches

If something is misbehaving, granular kill switches let you flip individual signals or whole subsystems off without losing the rest. Toggle via Admin Console → Settings → Kill Switches.

| Switch | What it disables |
|--------|------------------|
| `feature:engagement-signals` | All 6 Wave 1 engagement signals; Phase 1+2 mechanics signals + scoring still run |
| `feature:priority-deal-summary` | Disables the gpt-5-mini deal-summary generation (the italic 💡 line at the bottom of each card); existing summaries stay until cleared by a recompute |
| `feature:priority-cta-gpt` | The GPT-generated next-action sentences (scores still write, just no italic CTA line) |
| `cron:priority-score-catchup` | The 15-min catchup sweep (real-time webhook + nightly still run) |
| `cron:hub-portal-events` | Portal-event ingestion stops (existing rows still queryable) |
| `cron:hub-email-events` | Email-event ingestion stops |
| `feature:portal-login-signal` | Signal 7 specifically (separate from cron switch) |
| `feature:email-engagement-signal` | Signal 8 specifically |
| `feature:priority-queue-dm` | Currently OFF; flip ON in 1–2 weeks once weights are tuned to enable a Slack DM "your top 5 deals" section in the morning prep |

**Rule of thumb:** flip the **cron** switches if ingestion is broken. Flip the **signal** switches if the signal is noisy but you still want the data collected.

## Common Issues & Troubleshooting

| Issue | Fix |
|-------|-----|
| "No deals assigned to you" banner | The queue is showing org-wide top deals because you have no LSM/LRM-assigned deals. This is normal for managers / SRMs. |
| Queue is empty | Check `/queue/weights` — if all weights are disabled, nothing scores. Or click "Recompute now" if the nightly cron failed. |
| All deals show the same tier | Tier thresholds need tuning — open `/queue/weights` and adjust `tier_threshold_p0` / `_p1` / `_p2` / `_p3`. |
| A signal you expect isn't firing | Confirm the signal is enabled in `/queue/weights`, then check the firing condition (see signal table above). For engagement signals, confirm the engagement table actually has rows for that deal (may take ~15 min for the catchup cron to pick up new activity). |
| `signal_email_engagement_unfollowed` never fires for some reps | Those reps aren't on the HubSpot Sales Gmail/Outlook extension. Plain Gmail sends aren't tracked. |
| `signal_portal_login_unmet` never fires for any deal | HubSpot Web Analytics tracking pixel may not be on the borrower portal. Check via DevTools → Network → look for `js.hs-scripts.com/{portalId}.js` request on the portal page. |
| The queue updated but my deal looks stale | Click "Recompute now" (admin only) for a single forced refresh. Otherwise wait up to 15 minutes for the catchup cron. |
| **I clicked Recompute Now but stale deals (e.g. IRUC, IR Closed) are still in the queue** | **Recompute UPSERTs new scores; only the orphan-delete sweep deletes stale rows.** They're separate operations. The orphan-delete runs nightly; if you need it sooner (e.g. just after an active-stage rule change), an admin needs to run `SELECT * FROM delete_orphan_priority_score_rows();` directly via Supabase. **Backlog**: Recompute Now button to also trigger orphan-delete in the same call. See [[2026-05-14-pm2-lrm-drop-iruc-roles-converge]] "Lesson recorded". |
| Two of us edited weights at once | Whoever saved second gets the "weights changed since you loaded — reload" warning. Reload, re-apply your change. |
| The deal popup card doesn't open | Usually a stale browser cache. Hard-reload the page. If persistent, check that `EntityModalHost` is mounted (it should be — globally in `App.tsx`). |

## Owner & Review

- **Owner:** Tulli (Nicholas Santulli)
- **Review cadence:** Quarterly — re-tune weights based on operator feedback + actual deal-touch outcomes
- **Last reviewed:** 2026-05-14 PM-2 (PR #204: **LRM scope shrinks 5→4 stages dropping IRUC** per operator rule "once a deal is Under Contract, the closing-agent (CA) team owns it" — both roles now converge on the same 4-stage allow-list. Cumulative same-day LRM shrink: 7→6→5→4 across THREE operator clarifications. 283 stale rows cleaned in prod via MCP (273 LRM IRUC/CTF + 10 already-stale LSM the orphan-delete had missed). Same PR sized stage badge in card top-right 4× larger (fontSize 11→20, padding 2/10→8/16, weight 600→700) per same-turn operator request. **Lesson surfaced this turn — Recompute UPSERTs scores, only orphan-delete sweep deletes stale rows**; new troubleshooting entry above explains the distinction. Backlog: Recompute Now button should also trigger orphan-delete in the same call. **Earlier 2026-05-14 PM** (PR #193): Drop CTF from LRM scope (6→5) per operator rule "funding team owns CTF onward, not LRM"; 29 stale CTF rows cleaned via MCP. Same PR fixed Creative Scenarios "View the full 24-scenario playbook →" link landing on a blank page (wrong route prefix `/admin/knowledge` vs actual `/knowledge`); both callsites fixed; KnowledgePage gained `?search=` query param support. **Earlier 2026-05-14**: Day 2 sprint = Email LO drawer + ✨ Draft AI Follow-up [#166] replace mailto with canonical EmailComposerPanel; Settings → Backend → Priority Queue section with Signal Weights + Weights History + Hidden Deals Audit pages [#178, #181] + Recompute Now button + Last Edited column + AFTER UPDATE trigger-backed changelog table; impersonation header injection so View-as auto-filters the queue [#168]; conv-badge hide-list expanded to 5 stages adding Failed + Nurture [#185]; AM **LRM scope shrinks 7→6 stages dropping IR Closed** [#188] per operator rule "if it's closed then it's not of priority to LRMs or LSMs" — handoff to CA happens AT the IR Closed transition, not after; 435 stale rows cleaned in prod via MCP. Hotfixes same day: React #310 white-screen Rules-of-Hooks fix [#170], AI Draft maxTokens 1500→6000 [#171]. **Previous: 2026-05-13** Phase 3a launch + same-day follow-ups. _Original 2026-05-14 PM entry below preserved for reference:_ PR #193: **LRM scope shrinks 6→5 stages dropping Clear to Fund** per operator rule "once a loan is approved-to-fund, the funding/closing team owns the deal, not LRM" — cumulative same-day LRM shrink 7→6→5 across two scope clarifications; 29 stale CTF rows cleaned in prod via MCP. Same PR fixed Creative Scenarios "View the full 24-scenario playbook →" link landing on a blank page — wrong route prefix `/admin/knowledge` vs actual `/knowledge`; both callsites fixed; KnowledgePage gained `?search=` query param support so the deep-link actually pre-filters. **Earlier 2026-05-14**: Day 2 sprint = Email LO drawer + ✨ Draft AI Follow-up [#166] replace mailto with canonical EmailComposerPanel; Settings → Backend → Priority Queue section with Signal Weights + Weights History + Hidden Deals Audit pages [#178, #181] + Recompute Now button + Last Edited column + AFTER UPDATE trigger-backed changelog table; impersonation header injection so View-as auto-filters the queue [#168]; conv-badge hide-list expanded to 5 stages adding Failed + Nurture [#185]; AM **LRM scope shrinks 7→6 stages dropping IR Closed** [#188] per operator rule "if it's closed then it's not of priority to LRMs or LSMs" — handoff to CA happens AT the IR Closed transition, not after; 435 stale rows cleaned in prod via MCP. Hotfixes same day: React #310 white-screen Rules-of-Hooks fix [#170], AI Draft maxTokens 1500→6000 [#171]. **Previous: 2026-05-13** Phase 3a launch + same-day follow-ups: card layout, role-aware stages, Snooze/Remove, Stalled Deals toggle, Creative Scenarios, Call LO, deal summary, BBYS prob badge; PM correctness pass: real Slack content in summaries [#154], server-side stage filter [#159], stall-risk PRE-UC scope [#162])

## Related Notes

- [[2026-05-12-deal-priority-queue]] — engineering project doc with phase history + technical detail
- [[2026-05-12-priority-webhook-200-on-error]] — decision log on why the webhook returns 200 even on internal failure
- [[2026-05-14-lrm-handoff-at-ir-closed]] — decision log on AM LRM scope shrink (7→6 stages, IR Closed dropped)
- [[2026-05-14-pm-lrm-drop-ctf-funding-team-owns]] — decision log on PM LRM scope shrink (6→5 stages, Clear to Fund dropped — funding team owns CTF onward)
- [[2026-05-14-pm2-lrm-drop-iruc-roles-converge]] — decision log on PM-2 LRM scope shrink (5→4 stages, IRUC dropped — CA team owns IRUC onward; both roles now converge on the same 4-stage allow-list). Includes the "Recompute vs orphan-delete" lesson recorded.
- [[2026-05-13-hub-deals-as-active-set-source]] — sibling decision: "is this deal currently active?" reads `hub_deals` not `partner_report_deals`
- [[hub-mirror-gotchas]] — Gotcha 5 (canonical state column over derived/cached fields) is the umbrella pattern this SOP follows
- [[hubspot]] — HubSpot config including `manual_uc_override` property and the property-change workflow
- [[data-bridge]] — Data Bridge endpoints + crons + kill switches
- [[bbys-overview]] — BBYS product overview and stage funnel
