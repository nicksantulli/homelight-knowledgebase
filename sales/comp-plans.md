---
last_updated: 2026-09-24 (comp app now pays pod share on Retail / Wholesale / Builder-Client starting with September 2026. Pod share went from $20.5K to $34.5K and Head-LR rose $2.2K; see [[2026-09-24-comp-app-pods-live-september]]). Previous: 2026-09-09 (new §9 — DTI Drop closes go to flat $100/$200 on comp-able value and
stop consuming a slot in the 6/10/13/2 ladder, for AEs, AMs and Builder reps. Built, NOT
published, no cutover date. Pod share DOES apply and stays at 50%, confirmed same day — and it is
the second-largest piece of the impact, $19.6K of a $50.1K reduction, because pod credit is an
override per podmate rather than a division. Measured at ~51% reduction in total comp on DTI Drop
volume across every role; the ladder exclusion offsets far less than it appears because the
4th-transaction cliff absorbs most shifted ordinals. Open questions 1 and 7 partially answered;
new Q9 (pod share — answered same day, it survives at 50%) and Q10 (do the derived/override plans
follow reps onto flat?). See
[[2026-09-09-dti-drop-flat-comp]]. Previous: 2026-08-29 — Loop-3 reconciliation: §3's D2C-branch
callout corrected, "25% agent-sourced haircut" was itself ambiguous prose; flagged 🔴 as a 3×
conflict against [[bbys-pricing-engine]]:198's "25% comp discount" reading, cross-cited both
directions. See [[numbers-that-disagree]] row 33, [[risk-register]] row 46.)
status: current
source: Google Drive — "2026 Compensation Plan" (jake@homelight.com, Drive modified 2026-06-30) · "2026 Q2 Compensation Plan App Notes" (nicholas.santulli@, Drive modified 2026-03-27) · "2025 Compensation Plan" (jake@, modified 2025-11-04) · "AE Comp Plan" (john.labrada@, 2025-02-12) · legacy LRM/LOM plans (nick.friedman@, 2023-03)
scope: How the Homes/BBYS revenue org is actually paid — LSM, LRM, RevOps and Lender-Relations-lead plans; the comp-able-value formula; the quota accelerator; and the HubSpot fields the whole thing keys off.
source: HomeLight-Vault/context/comp-plans.md
imported: 2026-09-29
---

# Comp plans — how the BBYS revenue org gets paid

> 🔑 **This file exists because the vault previously had none.** [[glossary]] carried `BIPs`
> and `IRAX` as undefined stubs, [[bbys-unit-economics]] §Open-questions asked *"is 'BIPs of DR
> volume + IRAX bonus (threshold 200)' still live post-reorg? What are the actual bps tiers?"*,
> and [[INDEX]] listed comp under "deliberately NOT documented." The source artifacts were in
> Drive the whole time. **Answer: no, that model is dead.** See §0.
>
> Related: [[bbys-unit-economics]] · [[lo-lifecycle]] · [[sales-ops-operating-model]] ·
> [[team]] · [[glossary]]

⚠️ **Scope boundary.** This documents **plan mechanics** — rates, formulas, fields, cadence.
It contains no base salaries, no OTE, and no individual earnings. Keep it that way.

⚠️ **Vintage caution.** The 2026 plan document's own footer reads: *"This LSM compensation plan
is valid until the end of Q2 (end of June)."* It is **2026-08-29**. Whether the Q2 plan was
extended, replaced, or renegotiated for H2 is **not answerable from Drive** — see Open questions.

---

## 0. 🔴 Correction — BIPs/IRAX is superseded

| | Old model (as recorded in [[glossary]], [[q2-priorities]]) | 2026 model (observed in source) |
|---|---|---|
| Structure | "BIPs of DR Volume + IRAX bonus, threshold 200" | bps per **Nth closed transaction for that LO**, escalating then collapsing |
| Bonus | Discrete IRAX threshold bonus | Continuous **quota-attainment accelerator** on bps |
| Basis | DR volume | **Comp-able value** (a derived figure, not DR sale price — §2) |

The old glossary entries are not wrong as history; they are **stale as of 2026-04-01**. The
`IRX` root survives in the field names (`lo_bbys_irx_number`, `lo_s_irx_count`) — see §5.

---

## 1. Who is on which plan

| Plan | Population | Deal field that identifies them | Effective |
|---|---|---|---|
| **LSM** (→ AE post-Sep-1) | Lender Sales Managers | `lsm_collaborator` | 2026-04-01 |
| **LRM** (→ AM post-Sep-1) | Lender Relations Managers | `lender_relations_manager` | 2026-04-01 |
| **Head of RevOps** | Nick Santulli | — (all closed deals) | 2026-04-01 |
| **RevOps Specialist** | Gui Batista | — (all created apps) | 2026-04-01 |
| **Head of Lender Relations** | Jake Vogel | — (derived from team) | 2026-04-01 |

**LSM pods, as named in the comp source (2026-03-27):**

| Pod | Members |
|---|---|
| 1 | Richie Helali · Tierney Izar · Matt Leddy |
| 2 | Tejas Narkhede · TJ Sims · Brian Banes |
| 3 | Marisa Drake · Kara Kleingarn |

**LRMs named in the comp source:** Ian Pardo · Brandi Cirell · Kyle Bradish · Deborah Shutt ·
Ashlee Kim · Angelica Espinosa.

⚠️ These rosters are the **comp** rosters as of March 2026. They differ from the Sep-1 reorg
rosters — [[sales-ops-operating-model]] §7 records Barb Griego and Carter in AM scope discussion,
neither of whom appears here, and the Wholesale allocation (§7 below) collapses the pods
entirely. **Do not use this table as the current org chart** — [[team]] owns that.

**Comp app pods from September 2026 onward (live 2026-09-24):**

| Pod | Members |
|---|---|
| Retail | Richie Helali · TJ Sims · Marisa Drake |
| Wholesale | Tierney Izar · Tejas Narkhede · Brian Banes · Kara Kleingarn |
| Builder/Client | Tiffany Traxler · Michael Coffey (Builder role, so no pod share) |

Because pod credit is a 50% override *per podmate*, the 4-person Wholesale pod pays each AE
on three podmates. September pod share went from $20,468 to $34,464. **Head-LR (Jake) is 16% of
(LSM total including pod share + LRM total)**, so it rose too, from $10,228 to $12,467. Jan–Aug 2026
are locked and unchanged. Rollback and per-rep detail: [[2026-09-24-comp-app-pods-live-september]].

---

## 2. 🔑 Comp-able value — the base everything multiplies

Not DR sale price. Not `homelight_value` straight. A three-branch formula, identical for LSM
and LRM:

```
IF   hl_deals_lp_bbys_lo_est_dr_value IS NULL   → homelight_value
ELIF hl_deals_lp_bbys_lo_est_dr_value <= 1,000,000 → hl_deals_lp_bbys_lo_est_dr_value
ELSE                                             → homelight_value * 1.05
```

🔑 **Read what this does:** below $1M, the **LO's estimated DR value** governs. Above $1M, the
formula abandons the LO's estimate and pays on HomeLight's own valuation with a 5% uplift — a
deliberate cap on LO-supplied numbers at the high end. If the LO estimate is absent entirely,
HL's valuation is used flat.

⚠️ This is a **third** revenue-adjacent basis, alongside the ones [[bbys-unit-economics]] §8
already tracks (`bbys_revenue`, `actual_program_fee_amount`). Comp-able value will not tie to
either. Add it to the list before anyone reconciles comp against revenue.

---

## 3. LSM plan — bps by the LO's Nth close

Rates are **per closed transaction, counted per loan officer**, not per LSM.

| LO's closed transaction # | 2025 plan | 2026 plan (eff. 2026-04-01) |
|---|---|---|
| 1st | 4.5 bps | **6 bps** |
| 2nd | 9 bps | **10 bps** |
| 3rd | 12 bps | **13 bps** |
| 4th and beyond | 2 bps | **2 bps** (unchanged) |

🔑 **The 4th-transaction cliff is the load-bearing design decision.** Comp collapses from 13 bps
to 2 bps — an ~85% cut — once an LO is producing repeatably. The plan pays for *activating* LOs,
not for *farming* them. Any analysis of LSM behavior (which LOs get attention, which get
neglected) has to start here.

⚠️ **Source ambiguity on the D2C branch.** The formula in the app-notes doc pays **zero** when
`bbys_opportunity_source = "Client (D2C)"`, and applies a `* 0.25` multiplier when source =
`"Agent"` — i.e., read literally, the formula pays the LSM **a quarter (25%) of headline bps**
on agent-sourced deals. Neither the 0% D2C rule nor this multiplier appears anywhere in the
narrative plan document Jake circulated — they exist **only in the formula**. Confirm before
quoting: a rep reading the plan doc would not know either rule exists.

🔴 **2026-08-29 — this is a conflict, not just an undisclosed rule, and it is unresolved.**
[[bbys-pricing-engine]]:198 (citing `sops/2026-06-22-discount-exception-policy.md`, draft,
unratified) independently documents the same `bbys_opportunity_source = "Agent"` field as a
**"25% comp discount"** — plain reading of that phrase is the LSM is paid **three quarters
(75%)**, the opposite direction from the `* 0.25` multiplier above. **Two readings, 3× apart,
on the same live comp field, from two different source documents, and neither document cites
the other.** This file previously compounded the ambiguity by itself calling the rule "the 25%
agent-sourced haircut" in prose (a phrase that reads the same way both sources do — ambiguous
between "cut by 25%" and "cut to 25%"); that phrasing is corrected above. Do not pick a winner
here — neither source is code, and no query against `bbys_opportunity_source` payout data was
run this pass. **Needs Jake Vogel + Nick Friedman to adjudicate.** See [[numbers-that-disagree]]
row 33 and [[risk-register]] row 46.

### Pod commission

| Situation | Credit |
|---|---|
| You are the `lsm_collaborator` on the deal | **100%** of comp-able value at your bps |
| A **podmate** is the `lsm_collaborator` | **50%** (was 100% in the prior plan) |

The 2025→2026 change halved shared credit. Reporting requirement noted in the source: LSM
reports need a column showing the pod share at 50%.

### Quota-attainment accelerator (LSM only)

Quota is measured in **applications submitted**, tracked against the HubSpot **goal named
"BBYS Applications."** Any application submitted *after* quota is hit carries an accelerated bps
for whatever deal it eventually becomes — **regardless of when that deal closes**.

| Attainment | Accelerator | 6 bps becomes |
|---|---|---|
| 100–125% | +25% | 7.5 bps |
| 125–175% | +50% | 9 bps |
| 175%+ | +100% | 12 bps |

Worked example from the source: a 3rd-close deal at the +100% tier pays **26 bps** (13 × 2).

⚠️ **Two boundary bugs sitting in the published tiers.** 125% and 175% each appear in *two*
brackets. At exactly 125% attainment the plan does not say whether you get +25% or +50%. Nobody
has adjudicated this in the source.

🔴 **The accelerator carries an unbounded tail.** An application submitted in a blowout month
accelerates its deal *whenever* it closes — potentially two quarters later, into a period whose
own quota was missed. Combined with the comp-period-lock gap ([[sales-ops-operating-model]] §7,
and §7 below), a repod can destructively recompute closed periods that contain these tails.

---

## 4. LRM, RevOps and leadership plans

| Plan | Rate | Basis | Trigger date |
|---|---|---|---|
| **LRM** | **3 bps** flat | Same comp-able value formula (§2) | `ir_closed_date` |
| **Head of RevOps** | **0.5 bps** | ALL closed deals, same comp-able value | `ir_closed_date` |
| **RevOps Specialist** | **$1 per application** above a **1,000/month floor** | Deal count | `createdate` |
| **Head of Lender Relations** | **16%** of the summed individual LSM + LRM comp | Derived | `createdate` |

🔑 **Note the trigger-date split.** LSM/LRM/RevOps-head pay on `ir_closed_date`; the Specialist
and Head-of-Lender-Relations plans key off `createdate`. Two different clocks in one payroll
run — a deal can contribute to one person's August and another's May.

**Cadence, all plans:** monthly, paying the **previous** month. August's payment covers deals
whose trigger date falls in July.

**Population filter, all plans:** BBYS Applications pipeline, **HubSpot pipeline ID
`681595694`**, trigger date after **2026-04-01**.

The RevOps-Specialist floor is worth naming: at 1,000 apps/month the plan pays $0. The
[[bbys-unit-economics]] 2026 plan model (§below and there) targets 1,800–2,900 apps/month in
H2 — so the floor is set roughly at H1 run-rate and the plan is a growth kicker, not a salary
component.

---

## 5. Fields the comp engine reads

| Field | Object | Role in comp |
|---|---|---|
| `lsm_collaborator` | Deal | Identifies the LSM to pay; drives pod 50% share |
| `lender_relations_manager` | Deal | Identifies the LRM to pay |
| `hl_deals_lp_bbys_lo_est_dr_value` | Deal | Primary comp-able-value input (LO's estimate) |
| `homelight_value` | Deal | Fallback + the >$1M basis (×1.05) |
| `lo_bbys_irx_number` | Deal | The LO's Nth close → selects the bps tier |
| `lo_s_irx_count` | Deal | Stated in source as *"field for # of closed transaction"* |
| `bbys_opportunity_source` | Deal | `"Client (D2C)"` → 0; `"Agent"` → ×0.25 per this formula. 🔴 **[[bbys-pricing-engine]]:198 reads the same field as a "25% comp discount" (i.e. ×0.75) — 3× apart, unresolved, see §3 above.** |
| `ir_closed_date` | Deal | Period assignment for LSM/LRM/RevOps-head |
| `createdate` | Deal | Period assignment for Specialist + Lender-Relations-head |

🔴 **`lo_bbys_irx_number` vs `lo_s_irx_count` are two different field names for the same
concept in one document.** The formula reads `lo_bbys_irx_number`; the prose says the field is
`lo_s_irx_count`. Both coalesce blank/null to `1` (i.e. an unknown history pays the *first-close*
rate). Resolve which field is authoritative before building anything on this — paying the wrong
tier is a direct dollar error, and null-defaulting to tier 1 systematically **underpays** on
repeat LOs whose history didn't backfill.

---

## 6. The comp reporting app (in-flight)

The 2026-03-27 app-notes doc is a build spec, not just a plan. Durable facts:

- Data source: HubSpot deals, pipeline `681595694`, mirrored locally.
- Reuse target named: `github.com/nicksantulli/homelight-hub` (existing comp-reporting feature).
- Auth: Google OAuth. Four permission tiers: **Owner** (edit plans) → **Admin** (view all, no
  edit) → **Manager** (their reports) → **User** (self only).
- A `users` table storing **HubSpot owner ID + Aircall user ID + Slack member ID** per person —
  i.e. the same cross-system identity join [[sales-ops-operating-model]] and [[aircall-system]]
  each solve separately. 🔑 If this table exists, it is a candidate canonical identity map.
- Report line items: IR closed date, deal name, comp-able value, LO value, HL value, LO name,
  comp amount, who is being comped. PDF export. Manual one-off non-deal payments, month-attributed.
- Explicit requirement: *"Ensure any hubspot internal IDs are always transformed to actual
  names"* — an acknowledgement that raw HubSpot IDs leak into reports today.

---

## 7. 🔴 Comp-period locks are not deployed

From the 2026-08-19 Broker/Retail sync prep (see [[sales-ops-operating-model]] §7 and
[[projects/2026-08-29-drive-mining]]):

> *"Comp-period locks are NOT deployed. Fix sits on branch `reorg/comp-period-lock`. Until it
> ships, any repod can destructively recompute closed comp periods. **Hard no-go gate for
> automated cutover.**"*

This is the single highest-consequence comp fact in the vault: the Sep-1 reorg **reassigns
`lsm_collaborator`-adjacent ownership at scale**, and the system will recompute already-paid
periods when it does. Belongs in [[risk-register]] §1 if not already there.

---

## 8. Legacy plans (historical, do not quote as current)

| Doc | Owner | Date | Note |
|---|---|---|---|
| "AE Comp Plan" | john.labrada@ | 2025-02-12 | Predates the Sep-1 LSM→AE rename; "AE" here is the *older* usage. Do not conflate. |
| "Lender Relations Manager Comp Plan" (2 copies) | nick.friedman@ | 2023-03 | Pre-dates the 3 bps flat structure |
| "Lender Operations Manager Comp Plan" (2 copies) | nick.friedman@ | 2023-03 | Role no longer in [[team]] |
| "2025 Compensation Plan" | jake@ | eff. 2025-12-01 | 4.5/9/12/2 bps; included a higher-of-two-plans true-up for November production |

⚠️ **"AE" is now a collision.** The 2025 Labrada doc and the Sep-1 reorg both use "AE" for
different populations. Add to [[glossary]] §Collisions.

---

## 9. 🔑 DTI Drop goes flat (2026-09, built but NOT published)

**Source: Jake Vogel, Slack, 2026-09-09.** The first concrete plan change since the Q2 document
sunset — see Open question 1, which this partially answers: the plan is still being amended, so it
evidently did not lapse at end of Q2.

Two changes, both scoped to **DTI Drop** closes ([[dti-drop]]):

1. **Flat comp replaces bps** — **$100** under $750K comp-able value, **$200** at or above.
2. **DTI Drop closes stop consuming a tier slot** — the 6 / 10 / 13 / 2 ladder in §3 counts BBYS
   closes only.

Jake's worked example is the authoritative statement of intent:

> if LOs first close is bbys they would get 6 bps, 2nd close is DTI drop $100/$200, 3rd close is
> BBYS that would get paid as the 2nd closed so 10 bps.

A DTI Drop close still happens chronologically and still pays — it is just **invisible to the
ordinal counter**.

**Population:** AEs (LSM) and AMs (LRM) per Jake; **Builder reps added** by Tulli 2026-09-09.
**Per person, not per deal** — the AE gets $100 *and* the AM gets $100 on the same sub-$750K deal.

**Band is `>=`:** a deal at exactly $750,000.00 pays $200. Jake's "under / above" phrasing left it
undefined; pinned deliberately rather than adding a third boundary defect alongside the 125% /
175% brackets in §3.

**Basis is comp-able value** (§2), not DR sale price — so the >$1M / ×1.05 branch still applies
before the band is evaluated.

**DTI Drop applications still count toward quota.** The exclusion is from *closed-transaction*
counting only; quota is measured in apps submitted, a different clock.

**Pod share applies, still at 50%** — a podmate earns $50 / $100. Note §3's pod credit is an
override *per podmate*, not a division, so a 3-person pod turns one $100 close into $100 + $50 +
$50 of LSM-side comp.

### 🔑 This is a real reduction, not a restructuring

Across every comp role on the first 100 DTI Drop closes since 2026-04 (partial sample):

| Role | Paid today | Under new plan | Change |
|---|---|---|---|
| AE (LSM) | $38,046 | $11,000 | **−$27,046** |
| **AE pod share** | **$27,968** | **$8,350** | **−$19,618** |
| AM (LRM) | $13,039 | $11,500 | −$1,539 |
| Builder rep | $2,505 | $600 | −$1,905 |
| Derived/override roles (unchanged) | $17,021 | $17,021 | — |
| **Total** | **$98,579** | **$48,471** | **−$50,108 (−51%)** |

Plus Head-LR (Jake, §4) at 16% of the pool, which moves automatically: ~**$14.2K → $5.8K**.

**Pod share is the second-largest piece of the cut** — $19.6K of $50.1K. Per close, LSM-side comp
goes from **$725** ($418 AE + $307 podmates) to **$213**.

The AM barely moves because 3bps against the **$375K comp-able floor already floors them at
$112.50** — so flat $100 is marginally *below* today's AM minimum. Possibly unintentional.

The tier-slot exclusion offsets far less than it sounds: ~18 closes/month shift ordinal, but only
**7–9 actually change bps tier**, because most shifts land inside the 4th-and-beyond band where
everything pays 2bps regardless. **The 4th-transaction cliff (§3) absorbs most of the benefit.**

Volume is material: 19 / 34 / 25 / 17 / 5 DTI Drop closes in Apr–Aug 2026.

### ⚠️ The ladder is one un-dated column — do not re-rank it

The bps tier reads from `deals.calculated_irx`, a single global
`ROW_NUMBER() OVER (PARTITION BY lo_contact_id ORDER BY ir_closed_date)` with **no date scoping and
no history.** Re-ranking it to exclude DTI Drop would retroactively re-tier every past close for
every LO who has ever had one — including paid periods. Same class of failure as the comp-period-lock
gap in §7, except the lock protects `deal_comps` rows, not the column feeding them.

Implemented as a **second column** (`calculated_bbys_irx`), selected per-deal by the close-date
rule set. Nothing historical moves.

⚠️ **`lo_s_irx_count` is a trap here.** It is computed HubSpot-side and **counts DTI Drop closes**,
so leaving it in the fallback chain would silently restore the slot consumption — and only on deals
with a missing computed ordinal. Excluded when the BBYS-only ordinal is in force. (Coverage makes
this nearly moot: 2 of 1,220 closes since 2026-03 lack `lo_contact_id`.)

### Status and cutover

**Built, not published.** Gated on an optional `dti_drop_comp` key in `comp_rule_sets.config`;
absent the key, behavior is unchanged. Cutover is therefore a Comp Rules edit, not a deploy.

**No date set.** Recommended anchor: **close date, 1st of a month, ~6 weeks' notice.** The reason
is not convenience — the *ordinal* half cannot be grandfathered coherently, because it is a per-LO
sequence: contract-date grandfathering would let two BBYS deals for the same LO closing in the same
month disagree about that LO's close count. ~407 deals are under contract right now and would
close under the lower rates unless lead time covers them (contract→close is 24d median, 42d p90).

Full detail, release order and open questions: [[2026-09-09-dti-drop-flat-comp]].

**Still open:** the cutover date, and whether the derived/override plans (§4, plus Brandi's
override and the Khovnanian carve-out) stay on bps while reps go flat. Default is unchanged.

---

## Open questions

1. **Did the Q2-only LSM plan get extended past June 2026?** The source document explicitly
   sunsets itself at end of Q2. No successor document exists in Drive under any comp-adjacent
   title. **Partially answered 2026-09-09:** Jake amended the plan in Slack (§9, DTI Drop goes
   flat), which is only coherent if the 6/10/13/2 ladder is still live — so the plan did **not**
   lapse at end of Q2. What still doesn't exist is a *document*: the plan is now "the Q2 doc plus
   an accumulating pile of Slack amendments," which is a worse state than either a renewal or a
   replacement. **The live rates are most reliably read out of `comp_rule_sets` in the comp app,
   not out of Drive.**
2. **Are the D2C-zero and Agent-×0.25 rules actually live?** They exist only inside the formula,
   never in the plan narrative. If live, agent-sourced deals pay LSMs a quarter of headline bps —
   a very large undisclosed haircut. 🔴 **And which reading is correct at all** — see §3's
   2026-08-29 addition: [[bbys-pricing-engine]]:198 independently asserts a "25% comp discount"
   (×0.75), 3× apart from this formula's ×0.25. Unresolved; needs Jake Vogel + Nick Friedman.
3. **`lo_bbys_irx_number` or `lo_s_irx_count`?** (§5) One document, two names, no adjudication.
4. **What happens at exactly 125% / 175% attainment?** (§3) Overlapping bracket boundaries.
5. **Does the Sep-1 reorg change any of these plans?** LSM→AE and LRM→AM are role renames; the
   comp fields are named for the old roles. Nothing in Drive says whether `lsm_collaborator`
   survives the rename or gets a new field (which would break every historical comp period).
6. **Did the comp reporting app ship?** No repo evidence checked in this pass.
7. **What do CAs, Builder reps and SRMs get paid on?** No plan document for any of them exists in
   Drive. **Builder reps are now partly answered** — the comp app pays them 7 / 9 / 13 bps by
   *contract*-quota attainment (not the LO's Nth close), plus flat DTI Drop comp per §9. Still
   nothing written down outside the code. CAs and SRMs remain unanswered.

9. ~~**Does pod share survive?**~~ **Answered 2026-09-09 — yes, and still at 50%.** Tulli: *"the
   new structures will be paid with the comp split - still 50%."* So flat DTI Drop comp is subject
   to the pod split like any other close; a podmate earns $50 / $100. Pod share is **not** going
   away. Worth restating the mechanic since it dominates the DTI Drop impact math: §3's pod credit
   is an **override per podmate, not a division** — each podmate independently earns 50%, so a
   3-person pod turns one $100 close into $100 + $50 + $50 of LSM-side comp.

10. **Do the derived/override plans follow reps onto flat comp?** New 2026-09-09. Head-LR,
    RevOps-NS, Brandi's `LRM-Override`, `Builder-Mgr-*` and `LSM-Khovnanian` all still earn bps on
    DTI Drop deals where the rep now earns $100. Default is unchanged; nobody has confirmed it.
    `RevOps-NS` is the sharpest case — DTI Drop books **no revenue at IR close at all**
    ([[dti-drop]]), so paying 0.5bps of comp-able value on it is arguably already anomalous.
8. **How does the accelerator interact with quota changes mid-year?** The 2026 quota model
   ([[bbys-unit-economics]] §2026-plan) ramps monthly quotas from ~120 to ~340 apps per LSM. A
   flat attainment % against a 3x-varying denominator is not the same incentive month to month.

## Related Notes

- [[bbys-unit-economics]] — the revenue side these bps multiply
- [[sales-ops-operating-model]] — payments ops, Trolley, the reorg
- [[lo-lifecycle]] — what "the LO's Nth close" means operationally
- [[glossary]] — BIPs/IRAX (now corrected here)
- [[team]] — current roster (this file's rosters are March-2026 comp rosters)
- [[projects/2026-08-29-drive-mining]] — provenance for this file
