# 01: Stefan's Crisis Redeployment

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** 1
**Persona:** Stefan the Disponent (Primary)

---

## Transaction (Q1)

**What this scenario covers:**
A live fleet disruption fires while Stefan is on shift. He moves from default triage view into crisis full-width state, reads a pre-seeded Kondukt recommendation, asks one follow-up cost question, reviews a Recommended Action Object, executes the rotation update in HAFAS himself, and logs the action back into the platform — leaving behind a complete recommendation→action→outcome shadow audit chain.

---

## Business Goal (Q2)

**Goal:** Recommendation acceptance threshold (RAO acceptance ≥70% within 30 pilot mandays) + Prevented delay-minutes ROI (≥7 evidenced incidents)
**Objective:** Trigger Map Objectives 1 + 3

---

## User & Situation (Q3)

**Persona:** Stefan the Disponent (Primary)
**Situation:** Stefan is on shift at the Wien operations centre, 07:23, mid-shift. He has multiple monitors open — Power BI, HAFAS, ServiceNow, vendor portal. The Fleet Manager AI Interface is on his primary monitor in default 70/30 triage view. He has logged in via ÖBB SSO at shift start (06:00); IdP-bound identity active.

---

## Driving Forces (Q4)

**Hope:** Make a sound redeployment call quickly with cost visible before he commits — and have the recording chain that protects him if it's ever escalated.

**Worry:** Acting on a phantom recommendation, or producing a decision trail that can't be reconstructed if challenged.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Desktop (Wien operations centre workstation, dual-monitor; primary 1920×1080+)
**Entry:** Stefan is already in the Fleet Manager AI Interface, triage view. A disruption fires — Unit 4722 door motor fault — and the incident modal appears over his current view.

---

## Best Outcome (Q7)

**User Success:**
Stefan makes a €2,400 decision in under 90 seconds, executes the rotation update in HAFAS himself, confirms with the conductor by radio, and returns to triage with the action logged. The platform now holds a complete chain: RAO generated → displayed to Stefan → Stefan modified-or-accepted → Stefan logged "executed via HAFAS" → platform background-polled HAFAS state and verified.

**Business Success:**
The recommendation→action→outcome chain captures one accepted RAO with verified downstream outcome — feeding the Phase 2 trust-criteria dataset and one count toward the ≥70% acceptance threshold. If the fault would have escalated to a service-disrupting delay, the counterfactual is logged toward the prevented-delay-minutes ROI claim.

---

## Shortest Path (Q8)

1. **Triage view (70/30 default)** — Stefan is monitoring the fleet feed. Cost-impact one-liners visible on each card. Disruption fires for Unit 4722.
2. **Crisis modal** — "Unit 4722 — Door Motor Fault, Car 4. Active unresolved incident. View?" Stefan taps yes.
3. **Crisis full-width** — Layout transitions to full-width chat. Kondukt chat pre-seeded with incident context: "Unit 4722 door motor fault detected at 06:14. Conductor resolved manually at 07:14. Vehicle cleared by ECM at 06:08. Replacement unit available: Unit 4819."
4. **Free-form query → Rail MCP tool call** — Stefan types: "What does deploying 4819 cost versus keeping 4722 on route?" Kondukt calls `get_maintenance_cost_estimate`. Tool-call pill expands; result rendered in chat with confidence indicator.
5. **RAO card** — Stefan asks: "What does it take to redeploy 4819 to take 4722's slot on the next rotation?" Kondukt generates a Recommended Action Object: structured payload (HAFAS rotation update), justification, step-by-step procedure ("1. Open HAFAS, 2. Navigate to Rotation 22:15, 3. Replace 4722 with 4819, 4. Confirm with conductor by radio"), three-tier confidence indicator, and a "Mark executed" button. NO "Execute" button — the verb is operator-action-only.
6. **External execution (HAFAS — not in platform)** — Stefan reads the RAO, judges the cost (€2,400 maintenance overhead, confidence: medium), opens HAFAS in his other tab, executes the rotation update himself, confirms with the conductor by radio.
7. **Mark executed** — Stefan returns to the platform. He taps "Mark executed — confirmed via HAFAS at 07:24". The platform records: RAO generated_at, displayed_to_user_at, user_action_taken (executed), action_logged_at. Background poll against HAFAS read endpoint kicks off to verify state change.
8. **Triage view (return)** — Stefan returns to triage. The new RAO is visible in his RAO execution log for this shift. Outcome verification status: "Pending downstream confirmation". A few seconds later it flips to "Confirmed via HAFAS state poll". ✓

---

## Trigger Map Connections

**Persona:** Stefan the Disponent (Primary)

**Driving Forces Addressed:**
- ✅ **Want W1:** Cost consequence visible before confirming — cost-impact one-liner on triage card, refined in RAO
- ✅ **Want W2:** Receive structured proposals he can judge, not raw telemetry — RAO renders payload + justification + step-by-step
- ✅ **Want W3:** Build muscle memory for gesture that becomes authoritative in Phase 2 — "Mark executed" is the Disponent analog of the hold-to-record gesture
- ❌ **Fear F1:** Autopilot accusation — addressed by verb grammar ("Mark executed" not "Execute"), no platform-side HAFAS write
- ❌ **Fear F2:** Reconstructability gap — every RAO row captures the full chain (generated_at, displayed, action, logged, outcome)

**Business Goal:** Trigger Map Objective 1 (Prevent delay-minutes — counterfactual logged) + Objective 3 (RAO acceptance threshold — one acceptance recorded)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 01.1 | `01.1-triage-view/` | Default monitoring state with cost-impact triage cards | Disruption fires → crisis modal appears |
| 01.2 | `01.2-crisis-modal/` | Single-question modal — confirm switch to crisis view | "View" tap → crisis full-width opens |
| 01.3 | `01.3-crisis-fullwidth/` | Pre-seeded Kondukt chat with incident context | Stefan types free-form query |
| 01.4 | `01.4-tool-call-render/` | Inline tool-call pill + result rendering | Stefan reads and asks follow-up |
| 01.5 | `01.5-rao-card/` | RAO card with payload, step-by-step, confidence, "Mark executed" | Stefan switches to HAFAS externally |
| 01.6 | `01.6-mark-executed/` | Stefan logs his completed action back to the platform | RAO row updates; background poll starts |
| 01.7 | `01.7-execution-log/` | Triage view with RAO in execution log, outcome pending → confirmed | ✓ |

**First step** (01.1) includes full entry context (Q3 + Q4 + Q5 + Q6).
**On-step interactions** (chat composer, tool-call expand, confidence-indicator tooltip) are documented as storyboard items within each page spec.

---

## Open Questions (for Phase 4)

- **Crisis modal vs full-page transition** — is the crisis modal a 1-2 second intermediate state (giving Stefan a "do I want this now?" beat), or does it morph straight into full-width? UX scenario locks the gesture; Phase 4 specifies the timing curve.
- **Confidence indicator placement on RAO card** — three-tier grammar is required; where does the indicator live on the card (top-right corner? band along the left edge? inline with justification?). Phase 4 spec.
- **"Mark executed" — modify-before-mark vs accept-then-mark** — if Stefan executed a slightly different HAFAS payload than the RAO proposed (e.g., effective_at 22:20 instead of 22:15), does he type the modification before tapping Mark executed, or after? Captures modification rate signal — see scenario 02.
- **Background poll cadence** — how often does the platform poll HAFAS read endpoint to verify outcome? PRD says "subsequent polling cycles attempt reconciliation" — Phase 4 spec.
