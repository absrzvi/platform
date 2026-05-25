# 08: Roland + Martin's Manday-15 ROI Conversation

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** 5
**Personas:** Stefan (as evidence) + Renate (gatekeeper signal)
**Real-world participants:** Roland Ruisz (operations) + Martin Lerch (Technical Services)

---

## Transaction (Q1)

**What this scenario covers:**
Pilot manday 15. Roland and Martin sit down with the pilot evidence dashboard. They are not assessing features — they are assessing whether to sign a Phase 2 rollout recommendation. Composite Phase 1 health score with threshold line. Composite Phase 2 trust criteria with separate threshold line. Prevented delay-minutes counterfactual. They make a verdict.

---

## Business Goal (Q2)

**Goal:** Phase 1 ROI proof + Phase 2 readiness verdict at pilot manday 15 + reference customer / fundable event
**Objective:** Trigger Map Objectives 1 + 5 + 6

---

## User & Situation (Q3)

**Personas:** Roland Ruisz (real-world Stefan equivalent — operations) + Martin Lerch (Technical Services). Renate's prior endorsement (from scenario 05) is one of the trust-criteria inputs surfaced here, but she is not in the room.
**Situation:** Conference room at ÖBB Wien operations centre, pilot manday 15. Abbas (Nomad) is in the room. The pilot evidence dashboard is up on the conference screen.

---

## Driving Forces (Q4)

**Hope:** See evidence rigorous enough to support a written Phase 2 endorsement they can take to their CFO without arguing the case.

**Worry:** A dashboard that looks great but ducks one of the five trust criteria.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Conference-room display (HDMI from a laptop; Abbas drives)
**Entry:** Abbas opens the pilot evidence dashboard. Roland and Martin are sitting at the table.

---

## Best Outcome (Q7)

**User Success:**
Roland and Martin verify (1) Phase 1 health composite at or above threshold, (2) Phase 2 trust composite at or above threshold, (3) prevented delay-minutes counterfactual honestly framed, (4) infrastructure cost vs delay-minute savings produces a clear payback figure. They sign a Phase 2 rollout recommendation memo before leaving the room.

**Business Success:**
Trust criteria #3 + #4 (Roland verbal + Martin verbal endorsement) — earned. Phase 2 commercial proposal becomes a contract renewal with pre-mapped path, not a fresh procurement fight. Reference customer locked. Fundable-event package complete.

---

## Shortest Path (Q8)

1. **Pilot evidence dashboard — composite landing** — Two big composite numbers at the top: PHASE 1 HEALTH (composite of ECM pre-arrival review rate, Disponent decision latency reduction, shadow audit-trail match rate) with threshold line; PHASE 2 TRUST (composite of RAO acceptance rate, shadow-match 100%, Renate sign-off status, outside-auditor verdict, no-unresolved-pilot-kill flag) with separate threshold line. Both showing AT-OR-ABOVE-THRESHOLD.
2. **Phase 1 component breakdown** — Click reveals: ECM pre-arrival review 83% (target >80% ✓), Disponent decision latency reduction 38% vs baseline ✓, RAO acceptance rate 76% (target ≥70% ✓), shadow audit-trail match rate 100% ✓. Roland glances at each. He nods.
3. **Phase 2 trust criteria — five-criterion grid** — Below: five tiles, one per criterion. (1) RAO acceptance rate — green (76% ≥ 70%). (2) Shadow audit match — green (100% across 47 sign-offs). (3) Renate Fischer endorsement — green (conditional endorsement received manday 15, condition: 100% match holds across full pilot). (4) Outside-auditor tabletop verdict — green (verdict received manday 24). (5) No unresolved pilot-kill triggers in final 30 days — green (zero triggers fired). All five green → composite at threshold.
4. **Prevented delay-minutes panel** — A counterfactual narrative panel: "7 faults detected pre-disruption (sourced from brain telemetry). Historical baseline for same fault classes on unmonitored units: 23-minute mean delay. Estimated delay-minutes prevented: 161. The platform's contribution was the speed of the recommendation. Stefan acted; rotation went through; delay was prevented." Roland reads. He recognises the honesty of the framing.
5. **Cost vs savings — payback panel** — Below: "Platform infrastructure cost estimate per consist per month: €X. Equivalent delay-minute savings at ÖBB cost-per-delay-minute (Roland-confirmed): €Y. Payback period at full fleet deployment: under 2 months." Martin reads. He nods.
6. **Export for executive presentation** — Abbas clicks "Export". A structured PDF generates: composite scores, trust-criteria grid, counterfactual narrative, payback. Includes Phase 2 readiness verdict line. Roland will take this to his CFO. Martin will take it to his TS leadership.
7. **Phase 2 endorsement (external — not in platform)** — Roland and Martin discuss briefly. They tell Abbas they will sign the Phase 2 rollout recommendation memo before end of week. ✓

---

## Trigger Map Connections

**Personas:** Stefan (as evidence carrier) + Renate (gatekeeper signal already captured)

**Driving Forces Addressed (cross-persona):**
- ✅ **Stefan W4:** Be the operator whose shift produced the prevented-delay-minutes number Roland walks into the board meeting with — this scenario *is* that meeting
- ✅ **Renate W4:** Audit-committee artefact — the exported PDF is the artefact Renate would have presented if she were in the room (she is not, but her endorsement is captured in the grid)
- ❌ **Stefan F1:** Autopilot accusation — counterfactual narrative panel explicitly frames "platform recommended; Stefan acted"

**Business Goal:** Trigger Map Objectives 1 (prevented delay-minutes proven), 5 (Phase 2 readiness composite verdict), 6 (reference customer + fundable event)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 08.1 | `08.1-composite-landing/` | Two big composite scores with threshold lines | Roland clicks Phase 1 |
| 08.2 | `08.2-phase-1-breakdown/` | Per-component evidence with target vs actual | Roland reviews Phase 2 |
| 08.3 | `08.3-phase-2-trust-grid/` | Five-tile trust-criteria grid, each with state + evidence link | Martin reads counterfactual |
| 08.4 | `08.4-counterfactual-panel/` | Prevented-delay-minutes narrative, honest framing | Roland reviews payback |
| 08.5 | `08.5-payback-panel/` | Cost-vs-savings panel with payback period | Abbas exports |
| 08.6 | `08.6-export-pdf/` | Structured PDF with all panels + Phase 2 readiness verdict | ✓ |

---

## Open Questions (for Phase 4)

- **Composite score formula transparency** — does the dashboard show the formula behind each composite, or just the result? Renate's audit-committee model favours transparency; Roland's executive-presentation model favours the result. Phase 4 spec — likely both, with formula-on-demand.
- **Threshold line for Phase 1 vs Phase 2** — Phase 1 is "pilot health" (binary go/no-go); Phase 2 is "readiness" (composite of five criteria). Different threshold semantics. UI must not flatten this distinction.
- **Pilot-kill trigger surface** — even if zero have fired, do they appear in the grid as "passed"? Or only fired triggers show? Phase 4 spec — affects how Renate's audit committee reads the artefact.
- **Export PDF design** — is the export the same layout as the screen, or a presentation-formatted deck? Roland needs CFO-presentable; Martin needs internal-TS-presentable. Phase 4 spec.
- **Outside-auditor tabletop verdict input** — by manday 15 this verdict has not yet been received (tabletop is mandays 20–25). The grid in this scenario assumes mandays 24+. Update scenario for actual manday-15 cut: that tile is "pending — scheduled manday 20–25" rather than green.
