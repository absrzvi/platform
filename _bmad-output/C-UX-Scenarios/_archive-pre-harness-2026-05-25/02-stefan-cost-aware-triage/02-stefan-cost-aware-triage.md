# 02: Stefan's Cost-Aware Non-Crisis Triage

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** 1
**Persona:** Stefan the Disponent (Primary)

---

## Transaction (Q1)

**What this scenario covers:**
Stefan is in default triage state with no live crisis. A non-blocking warning surfaces a fleet pattern Kondukt thinks is worth attention. Stefan reads the cost-impact one-liner, queries a recommendation, *modifies* the proposal slightly before executing, and logs the modification with reason — capturing the modification-rate signal that is as important to the Phase 2 trust dataset as plain acceptance.

---

## Business Goal (Q2)

**Goal:** Recommendation acceptance threshold + modification-rate signal capture
**Objective:** Trigger Map Objective 3 (RAO acceptance) + the modification-as-signal sub-objective from Stefan persona detail

---

## User & Situation (Q3)

**Persona:** Stefan the Disponent (Primary)
**Situation:** Stefan is on shift at the Wien operations centre, 14:47, mid-shift. No active crisis. Triage view shows fleet status across his section. He has just finished his coffee.

---

## Driving Forces (Q4)

**Hope:** Use the platform to surface things he might otherwise miss, judge them on his own terms, and have his judgment captured even when it differs from the recommendation.

**Worry:** That the platform's modification-handling is binary (accept/reject) and his nuanced judgment gets coded as "rejection" — eroding the acceptance rate metric and his trust signal.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Desktop (Wien operations centre workstation)
**Entry:** Stefan is already in the Fleet Manager AI Interface, default triage 70/30 view. A WARNING card surfaces on the feed: "Unit 4831 — brake wear approaching threshold; recommended G-Check this maintenance window."

---

## Best Outcome (Q7)

**User Success:**
Stefan reads the cost-impact one-liner ("Estimated maintenance window: 4h, cost €380. Delay impact if not scheduled: ~12 minutes per affected service over 2 weeks"), reviews the RAO, decides to modify the proposed maintenance slot (Friday → Saturday early window), logs his modification with reason ("Friday rotation has higher passenger load — Saturday early less disruptive"), and the platform captures the original + modification + reason.

**Business Success:**
The modification-rate signal feeds the Phase 2 trust composite — modifications are *not* counted as rejections; they are counted as trust-with-judgment-overlay, which is the right signal for "platform is close-but-not-yet-trusted-to-dispatch." One more data point toward the 30-manday acceptance threshold.

---

## Shortest Path (Q8)

1. **Triage view (70/30 default)** — Stefan is monitoring. WARNING card surfaces for Unit 4831 with cost-impact one-liner visible.
2. **Card expand** — Stefan clicks the card. Inline expansion shows fault detail + cost-impact summary + "Generate recommendation" affordance.
3. **RAO generation** — Stefan taps "Generate recommendation". Kondukt calls Rail MCP tools (`get_maintenance_window_options`, `get_passenger_load_for_rotation`). RAO renders with proposed maintenance slot (Friday 22:00–02:00), payload, justification, three-tier confidence, "Mark executed" button.
4. **Modify** — Stefan reads the recommendation. He clicks "Modify" (visible on the RAO card alongside "Mark executed"). A modification panel opens inline — payload fields become editable; original values shown alongside; a "Reason for modification" field is required.
5. **Modify in-place** — Stefan changes the maintenance slot to Saturday 04:00–08:00. He types the reason: "Friday rotation has higher passenger load — Saturday early less disruptive." Modification panel shows the diff (Friday 22:00 → Saturday 04:00) before save.
6. **Mark executed with modification** — Stefan taps "Mark executed (with modification)". Platform records: original RAO, Stefan's modification, reason text, IdP identity, timestamp. The execution log row carries both the original recommendation and the modification, linked. Background process schedules the SAP G-Check ticket export for the new slot.
7. **Triage view (return)** — Stefan returns to triage. The RAO in his execution log this shift shows "1 accepted with modification" alongside the other RAOs for the shift. ✓

---

## Trigger Map Connections

**Persona:** Stefan the Disponent (Primary)

**Driving Forces Addressed:**
- ✅ **Want W1:** Cost consequence visible before confirming — both on triage card and refined in RAO
- ✅ **Want W2:** Structured proposals he can judge, not raw telemetry
- ✅ **Want W4:** Be the operator whose modifications and judgments are captured — the platform records his nuance, doesn't flatten it
- ❌ **Fear F1:** Autopilot accusation — "Mark executed (with modification)" verb explicit about who is acting
- ❌ **Fear F3:** "Co-pilot is slower than HAFAS" — modify-in-place is faster than rejecting + re-querying

**Business Goal:** Trigger Map Objective 3 (RAO acceptance composite — modification is its own signal type, not a rejection)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 02.1 | `02.1-triage-view/` | Default state with surfaced WARNING card | Stefan clicks card |
| 02.2 | `02.2-card-expand/` | Inline expansion with cost-impact + "Generate recommendation" | Stefan generates RAO |
| 02.3 | `02.3-rao-card/` | RAO with payload, step-by-step, confidence, "Modify" + "Mark executed" | Stefan modifies |
| 02.4 | `02.4-modify-panel/` | Inline modification panel with diff view + required reason | Stefan marks executed with modification |
| 02.5 | `02.5-execution-log-with-modification/` | RAO row shows original + modification + reason | ✓ |

---

## Open Questions (for Phase 4)

- **"Modify" button placement vs "Mark executed"** — both are operator actions on the RAO. Co-located, or separated by visual weight? Phase 4 spec.
- **Modification diff rendering** — side-by-side, unified, or "field-by-field with original strikethrough"? Affects how Stefan reads his own change at speed.
- **Reason-text requirement** — is it free-text, or does it offer a dropdown of common reasons (Stefan's persona detail F3: "logging structured detail he'd otherwise scribble in 3 seconds")? Dropdown-plus-free-text is the likely pattern.
- **SAP G-Check export trigger** — when does the export queue, on Mark Executed or after Stefan's actual external execution? PRD scope: "structured export/copy for manual entry" — Phase 4 spec.
