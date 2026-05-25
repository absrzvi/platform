# 04: Elena's ACK + DEFER Loop on Shared Device

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** 3
**Persona:** Elena the ECM 4 Technician — Electrical (Secondary)

---

## Transaction (Q1)

**What this scenario covers:**
Elena is on depot floor mid-shift. She has just finished fitting a replacement door motor on Unit 4815 and noticed a non-blocking camera firmware mismatch on the same unit. She picks up the shared depot tablet, selects her role (Electrical), authenticates with her IdP per action, logs the resolution ACK for the door motor and a DEFER (with reason) for the camera firmware — both propagate to the ECM Manager's pre-arrival brief without a phone call.

---

## Business Goal (Q2)

**Goal:** ECM audit-trail credibility — closing the ACK loop so the ECM Manager's brief is populated, supporting >80% pre-arrival review rate
**Objective:** Trigger Map Objective 2 (supporting)

---

## User & Situation (Q3)

**Persona:** Elena the ECM 4 Technician — Electrical (Secondary)
**Situation:** Wien Westbahnhof depot, Bay 5, 03:18 night shift. Elena has just fitted a replacement door motor part (78-2299-A) on Unit 4815 and run the test (PASS). On her diagnostic kit she also sees a camera firmware mismatch on VLAN 5 — Cat IT-tier warning, not blocking the door issue. The depot tablet is mounted on the bay wall.

---

## Driving Forces (Q4)

**Hope:** Log two actions in fewer taps than the alternative (phone call to ECM 1 cover) — and have her name attached to each so it counts in the brief.

**Worry:** That she'll be held accountable for an ACK made under someone else's IdP because the shared device cached a previous tech's role + identity together.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Shared depot tablet (mounted, Bay 5; PWA installed; no stored identity; role selector clears every 4 hours)
**Entry:** Elena finishes the bay work and walks to the tablet. The tablet shows the role selector screen (Electrical / Mechanical / IT / ECM Manager — 4 large touch targets).

---

## Best Outcome (Q7)

**User Success:**
Elena logs the door motor ACK with one tap (after IdP auth) and the camera firmware DEFER with one tap + reason selection (after IdP auth). Both propagate to Klaus's pre-arrival brief for Unit 4815 within seconds. No phone call. Two actions, ~45 seconds total.

**Business Success:**
The shared-device identity model holds: role is shared per shift, identity is per-action (Elena's IdP is bound to the specific ACK and DEFER rows, not to the tablet's cached state). The brief Klaus reads tomorrow morning at 05:50 is accurate to the field.

---

## Shortest Path (Q8)

1. **Role selector** — Elena taps "Electrical" (one tap, no confirmation, large touch target).
2. **Electrical role view — vehicle list** — PWA opens role-filtered view. List shows vehicles in active bays. Elena taps "Unit 4815".
3. **Vehicle card stack — Electrical-filtered** — Cards visible to Electrical: door motor fault (CRITICAL — pending ACK), camera firmware mismatch (WARNING — pending DEFER or ACK). Brake wear alert is not in her view (filtered by `role_relevance`).
4. **Door motor card → ACK** — Elena taps the door motor card. Card expands. She taps "ACK — resolution logged". PWA prompts for IdP auth (her ÖBB SSO) — single biometric or PIN tap because the device is shared, not personal. After auth, the ACK action surface shows pre-populated technical detail (fault code, port, last-known-good state from IntentPacket data), with a single "Logged by Elena Müller at 03:18" confirmation row. She confirms.
5. **Camera firmware card → DEFER with reason** — Elena taps the camera firmware card. Card expands. She taps "DEFER". A reason selector appears: dropdown with common reasons ("Schedule with next maintenance window", "Awaiting vendor confirmation", "Service impact — investigate further") + free-text option. She picks the first option. PWA prompts for IdP auth again (per-action, not per-session). After auth, the DEFER action surface confirms: "Deferred by Elena Müller at 03:19. Reason: Schedule with next maintenance window."
6. **Vehicle card stack — return** — Elena returns to the card stack. Both cards now show their current state (door motor: RESOLVED, camera firmware: DEFERRED) with her IdP-bound identity attached. ✓

(Background: Klaus's pre-arrival brief for Unit 4815, scheduled for HAFAS arrival tomorrow at 06:11, now reflects both updates. No phone call needed.)

---

## Trigger Map Connections

**Persona:** Elena the ECM 4 Technician — Electrical (Secondary)

**Driving Forces Addressed:**
- ✅ **Want W1:** Role-filtered view — brake wear not present, only electrical-system cards
- ✅ **Want W2:** Log ACK once and propagate to Klaus automatically — no phone call
- ✅ **Want W3:** DEFER with reason logged, accountability without overhead
- ❌ **Fear F1:** Re-selecting role every shift — role clears every 4 hours, not every action (Elena's 12-hour shift = 3 role selections, tolerable)
- ❌ **Fear F2:** Identity confusion on shared device — IdP auth is per-action, never cached. The role is shared, the identity is per-row.
- ❌ **Fear F4:** Over-filtered blind spots — `role_relevance[]` is multi-tag (open decision in `ux-decisions-2026-05-23.md` Q1 — pre-pilot blocker)

**Business Goal:** Trigger Map Objective 2 (supporting — populates Klaus's brief)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 04.1 | `04.1-role-selector/` | 4-target shared-device entry with 4-hour clear | Elena taps Electrical |
| 04.2 | `04.2-electrical-vehicle-list/` | Role-filtered list of vehicles with open actions in her domain | Elena taps Unit 4815 |
| 04.3 | `04.3-vehicle-card-stack-electrical/` | Cards filtered to Electrical role; brake alert absent | Elena taps door motor card |
| 04.4 | `04.4-card-detail-with-ack/` | Technical-depth view (fault code, port, LKG) with "ACK" action | Elena taps ACK → IdP auth fires |
| 04.5 | `04.5-idp-auth-per-action/` | Per-action IdP auth challenge (biometric/PIN) | Auth succeeds |
| 04.6 | `04.6-ack-confirmation/` | ACK row bound to Elena's IdP, propagating to brief | Elena returns to card stack |
| 04.7 | `04.7-card-detail-with-defer-reason/` | Reason selector (dropdown + free-text) for DEFER action | Elena picks reason → IdP auth fires again |
| 04.8 | `04.8-defer-confirmation/` | DEFER row with IdP + reason captured | ✓ |

---

## Open Questions (for Phase 4)

- **IdP auth per-action UX cost** — biometric is fast (sub-second), PIN is slower. On a shared device without biometric (e.g., locked-down depot tablet), per-action PIN may push Elena back to clipboard. Phase 4 needs an answer: is biometric a hard requirement for the depot tablet build? If not, fallback UX needs design.
- **`role_relevance` multi-tag for cross-domain faults** — a camera-power fault that's both Electrical and IT — does it show in both queues? Same card or two cards? Pre-pilot blocker (open decision Q1).
- **Vehicle list sort order for Electrical** — pending-ACK first? Severity-sorted? Chronological? Elena's persona detail Q "Sort order per role" — sub-question Phase 4.
- **DEFER reason taxonomy** — needs ECM Manager sign-off on the dropdown options. Pre-pilot blocker.
