---
date: 2026-03-26
type: project
status: draft-complete
tags: [lo-channel, lead-funnel, bbys]
source: HomeLight-Vault/projects/2026-03-26-lo-qualification-system-justification.md
imported: 2026-09-29
---

# LO Qualification System Justification

## Summary

Built a business justification memo for a new lead funnel model that pre-qualifies Loan Officers before they enter the LSM pipeline. The current model pushes all 6,322 leads to LSMs, but ~71% of LOs never close a single BBYS deal. The new model scores and filters LOs upstream, sending only ~2,213 SQLs to LSMs.

## Key Numbers (Base Case: 20% SQL→App Rate)

- **443 apps** projected (vs. 467 current — fewer but higher quality)
- **111 IR closings** (vs. 103 current — +8 per 6 months, +16/yr)
- **$292K/yr** in direct savings + incremental revenue
- **$4.18M total** when including attribution correction ($3.9M)
- **Yield improvement:** 1.6% → 5.0%
- **LSM time savings:** 2,054 hours over 6 months (4,109 annualized) across the team

## Build Effort

- **Team:** Tulli (design/validation) + Guy (HubSpot build)
- **Timeline:** ~19 weeks across phases
- **Effort:** ~220–260 total hours, averaging 6–8 hrs/week per person
- **No engineering dependency** — all HubSpot native

## Sensitivity Analysis

- Breakeven SQL→App rate: **19%**
- Conservative case (20%): $292K/yr direct
- Upside cases at 25% and 30% also modeled

## Deliverable

Word doc: `LO_Qualification_System_Justification.docx`

## Open Items

- [ ] Validate LTV/tier numbers with internal data
- [ ] Confirm LSM headcount for per-person time savings framing
- [ ] Review and finalize before presenting to leadership

## Related

- [[2026-03-26-aircall-dynamic-routing]]
