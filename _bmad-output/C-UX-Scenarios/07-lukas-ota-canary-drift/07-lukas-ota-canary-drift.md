# 07: Lukas's OTA Canary + Drift Monitoring

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** 4
**Persona:** Lukas the Teil-Projektleitung Telematik (Primary — commercial wedge)

---

## Transaction (Q1)

**What this scenario covers:**
Lukas is running a phased OTA software rollout to the Cityjet DOSTO Neu fleet. He opens the OTA rollout view, inspects canary cohort health (first 3 units), reviews drift across firmware versions across the broader fleet, and identifies one unit that has drifted from the canary's expected post-update configuration. He flags it for follow-up before vendor sees it.

---

## Business Goal (Q2)

**Goal:** Vendor warranty leverage (preventative — catch drift before vendor uses it as dispute leverage) + project management surface
**Objective:** Trigger Map Objective 4 (preventative side)

---

## User & Situation (Q3)

**Persona:** Lukas the Teil-Projektleitung Telematik (Primary — commercial wedge)
**Situation:** Lukas's office, weekly OTA review cadence (Friday morning). Phased rollout — canary cohort 3 units deployed Monday, production cohort scheduled for next Wednesday subject to canary success.

---

## Driving Forces (Q4)

**Hope:** See drift early enough to triage it before production rollout — and have a structured record that surfaces it before vendor flags it as evidence in a dispute.

**Worry:** Drift that goes undetected until a vendor uses it as a wedge.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Desktop (Lukas's workstation)
**Entry:** Lukas opens the Telematics Control Board, navigates to the OTA rollout view.

---

## Best Outcome (Q7)

**User Success:**
Lukas reviews canary + drift in one screen, identifies one drifted unit, and creates a follow-up action (assigned to the responsible technician) — all before lunch. The production rollout is informed: he holds it pending drift root-cause.

**Business Success:**
One drift caught proactively. Vendor cannot use it as dispute leverage because the platform recorded it first, with timestamp + assigned owner.

---

## Shortest Path (Q8)

1. **OTA rollout view — landing** — Lukas opens the OTA rollout view. Top panel shows: rollout name (DOSTO Neu firmware v3.4.7), canary cohort (3 units, deployed Monday), production cohort (28 units, scheduled Wednesday), overall status (CANARY MONITORING).
2. **Canary cohort detail** — Lukas clicks the canary cohort. List of 3 units shown with: firmware version installed, time since install, telemetry health signals (memory utilisation, bus controller status, reservation service health), and a green/amber/red indicator per unit.
3. **Drift detection panel** — Below the cohort detail, the drift detection panel surfaces: "Configuration drift detected — Unit DN-009: bus controller config differs from canary expected post-update. Detected 2026-10-15 11:14. No corresponding ticket." Inline reference link to the underlying config diff.
4. **Drift detail — config diff** — Lukas clicks the drift detail. View shows: canary expected config (left), DN-009 actual config (right), diff highlighted (one parameter: `bus_controller.heartbeat_interval = 5000ms` expected, `7000ms` actual). Timestamp of last config change on DN-009: 2026-10-13 22:47 — before the canary deployment (Monday).
5. **Action — create follow-up** — Lukas clicks "Create follow-up". Form pre-populated: unit (DN-009), drift description (config diff summary), assignment dropdown (responsible technician list filtered to DOSTO Neu programme). Lukas selects Markus Becker (IT). He adds a note: "Root-cause why heartbeat interval drifted before canary install. Hold production rollout pending."
6. **Confirmation + record** — Lukas submits. Platform creates: a follow-up record bound to Lukas's IdP, assigned to Markus Becker's IdP, timestamped, with a status of "Open — pending root-cause." The OTA rollout view's status panel updates: "CANARY MONITORING — 1 drift item, production rollout held."
7. **OTA rollout view — return** — Lukas returns to the rollout view top panel. The production-rollout-held status is visible. He emails Markus a heads-up (external — not in platform). ✓

---

## Trigger Map Connections

**Persona:** Lukas the Teil-Projektleitung Telematik (Primary — commercial wedge)

**Driving Forces Addressed:**
- ✅ **Want W3:** OTA rollouts with canary + drift detection, not Excel
- ❌ **Fear F1:** Vendor picks apart timestamps — the drift detection record is timestamped + IdP-bound at creation
- ❌ **Fear F2:** Kondukt wrong, Lukas wears it — this scenario does not involve Kondukt inference; it's deterministic drift detection from config diff (less risk surface)

**Business Goal:** Trigger Map Objective 4 (vendor warranty leverage — preventative)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 07.1 | `07.1-ota-rollout-landing/` | Top-level rollout state + cohort summary | Lukas opens canary detail |
| 07.2 | `07.2-canary-cohort-detail/` | Per-unit telemetry signals + health indicators | Lukas opens drift panel |
| 07.3 | `07.3-drift-detection-panel/` | Surfaces drift items with timestamps and references | Lukas opens drift detail |
| 07.4 | `07.4-drift-detail-diff/` | Config diff view (expected vs actual, highlighted) | Lukas creates follow-up |
| 07.5 | `07.5-create-followup/` | Pre-populated form, IdP-bound assignment | Lukas submits |
| 07.6 | `07.6-confirmation-return/` | Status panel updated, production rollout held | ✓ |

---

## Open Questions (for Phase 4)

- **Drift detection scope** — config drift only (this scenario), or also firmware drift, telemetry-shape drift? PRD scope: "configuration/firmware drift detection across the fleet at the commissioning stage." Phase 4 needs to enumerate drift types and per-type surfaces.
- **Follow-up assignment routing** — is the responsible-technician list per-programme (filtered as in this scenario), or per-unit? Phase 4 spec.
- **Production-rollout hold logic** — is this a manual gate (Lukas's action holds it) or a policy gate (any open drift item holds it automatically)? Implications for autonomy of the rollout system. Phase 4 spec.
- **OTA rollout dependency** — does this scenario require an OTA cycle to overlap the pilot window? Like Journey 6 / scenario 06, this is opportunistic. Confirm with Lukas's programme team.
