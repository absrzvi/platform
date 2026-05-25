# 03: Klaus's Pre-Arrival Brief → Shadow Sign-Off

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** 2 + 3
**Persona:** Klaus the ECM 1 Manager (Primary — regulatory)

---

## Transaction (Q1)

**What this scenario covers:**
Klaus receives a push notification 14 minutes before Unit 4722's HAFAS scheduled arrival. He opens the Depot Briefing PWA on his personal phone, reads the role-filtered pre-arrival brief, walks to Bay 3, confirms with the technician on site, signs the paper sign-off per existing ÖBB process, then performs the 3-second-hold gesture in the platform to record his shadow sign-off — the same gesture that will become authoritative in Phase 2.

---

## Business Goal (Q2)

**Goal:** ECM audit-trail credibility (100% paper↔shadow match) + Pre-arrival review rate (>80%)
**Objective:** Trigger Map Objective 2 + supporting Objectives 5 (Phase 2 readiness)

---

## User & Situation (Q3)

**Persona:** Klaus the ECM 1 Manager (Primary — regulatory)
**Situation:** Klaus is an ECM 1 Manager at Wien Westbahnhof depot. He arrived cold this morning at 05:30 — not on shift when Unit 4722 came in last night. He has IdP-bound identity (ÖBB SSO) on his personal phone PWA, role pre-selected (stored from first selection). HAFAS scheduled arrival for Unit 4722 is 06:04. Time is 05:50.

---

## Driving Forces (Q4)

**Hope:** Walk to Bay 3 already knowing the fault chain, confirm with the technician, sign paper, and leave a digital shadow record that proves what he authorised — without the platform ever competing with paper for which is the sign-off.

**Worry:** That the platform's presence implies pressure to sign faster than his paper process supports, or that the shadow record will diverge from paper in a way that comes back at him in audit.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Personal phone (iOS, PWA installed; stored role: ECM Manager)
**Entry:** Klaus is at the depot office, drinking his second coffee at 05:50. Push notification fires from the Depot Briefing PWA: "Unit 4722 — pre-arrival brief ready (HAFAS arrival 06:04)."

---

## Best Outcome (Q7)

**User Success:**
Klaus reads the executive-summary view of the brief (plain English, what failed, who resolved, what was deferred, what was logged). He walks to Bay 3, confirms with the technician — the technician's account matches the brief's resolution chain. Klaus signs paper at 06:08. Then he performs the hold-to-record gesture in the platform: 3-second hold → 5-second countdown → atomic write. The shadow row captures his IdP identity, UTC timestamp, full vehicle state snapshot, directive reference EU 2019/779, HMAC, hash chain.

**Business Success:**
One more paper-shadow pair toward the 100% match-rate dataset Renate Fischer reviews at pilot manday 15. The pre-arrival review rate increments toward the >80% target. The platform's audit-trail credibility builds, one sign-off at a time, with no pressure on Klaus to change his process.

---

## Shortest Path (Q8)

1. **Push notification** — "Unit 4722 — pre-arrival brief ready (HAFAS arrival 06:04)." Klaus taps it.
2. **Pre-arrival brief — vehicle card** — PWA opens. Brief shows: Unit 4722, HAFAS arrival 06:04, 2 fault cards sorted by severity (CRITICAL: door motor, WARNING: SNMP camera firmware). Each card shows severity band + system + plain-English summary + resolution status. Role: ECM Manager (executive-summary depth — no fault codes, no port numbers).
3. **Card detail — CRITICAL door motor fault** — Klaus taps the door motor card. Card expands: "Door motor fault detected 02:11. Conductor resolved at 07:14 (intermediate part-swap by night-shift technician at 03:22). Replacement part 78-2299-A fitted. Test result: PASS. ACK by Elena Müller (Electrical), 03:22." Resolution chain visible as a vertical timeline.
4. **Card detail — WARNING camera firmware** — Klaus taps the camera card. Card expands: "SNMP camera firmware version mismatch on VLAN 5. Deferred by Markus Becker (IT) 04:11. Reason: 'Schedule with next maintenance window.' No service impact in current state."
5. **Bay walk (external — not in platform)** — Klaus walks to Bay 3. Confirms with the technician on site. Resolution chain matches what the brief said. He reviews the part, signs paper at 06:08.
6. **Return to PWA — Record sign-off entry** — Klaus returns to the brief on his phone. He taps "Record sign-off". A confirmation surface appears: "You are recording the platform's shadow entry for this sign-off. The paper sign-off you have just completed remains authoritative under EU 2019/779. Hold the button below to confirm."
7. **Hold-to-record gesture** — Klaus presses and holds the record button. 3-second hold triggers initiation. A 5-second countdown begins (visible ring around the button + countdown number). At any point during countdown Klaus can release to cancel — no write occurs. At countdown expiry, single atomic write fires. Shadow row created: IdP identity, UTC timestamp, vehicle state snapshot at sign-off, directive reference EU 2019/779, HMAC, hash chain.
8. **Sign-off success** — PWA shows confirmation: "Shadow sign-off recorded. Idempotency key: ...". The brief card for Unit 4722 visually transitions to "Cleared to service — shadow record matched paper at 06:08." Klaus pockets his phone. ✓

---

## Trigger Map Connections

**Persona:** Klaus the ECM 1 Manager (Primary — regulatory)

**Driving Forces Addressed:**
- ✅ **Want W1:** Arrive at bay already knowing what failed, who resolved it, what was deferred
- ✅ **Want W2:** Digital shadow that proves what he authorised, if paper is ever disputed
- ✅ **Want W3:** Process unchanged, downside risk dropped
- ✅ **Want W4:** Build the 100% shadow-match dataset for Renate's Phase 2 sign-off
- ❌ **Fear F1:** Paper-vs-shadow ambiguity — confirmation surface in step 6 explicitly labels paper as authoritative, shadow as companion
- ❌ **Fear F2:** Pressure to sign faster — no timers, no countdowns until *Klaus* initiates the gesture
- ❌ **Fear F4:** Skipping the bay walk — the brief's confirmation surface reinforces "confirm with technician on site"

**Business Goal:** Trigger Map Objective 2 (ECM audit credibility, paper↔shadow match) + Objective 5 (Phase 2 readiness composite)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 03.1 | `03.1-push-notification/` | Klaus's PWA push at HAFAS arrival − 14min | Klaus taps notification |
| 03.2 | `03.2-prearrival-brief/` | Vehicle-level card stack, role-filtered (ECM Manager depth) | Klaus opens a fault card |
| 03.3 | `03.3-card-detail-critical/` | Resolution chain timeline for the CRITICAL fault | Klaus opens WARNING card |
| 03.4 | `03.4-card-detail-warning/` | DEFER reason + service-impact statement for the WARNING | Klaus walks to bay |
| 03.5 | `03.5-return-record-entry/` | Confirmation surface labels paper authoritative + shadow companion | Klaus initiates hold gesture |
| 03.6 | `03.6-hold-to-record/` | 3s-hold → 5s-countdown → atomic write — with cancellation at any point | Atomic write fires at countdown expiry |
| 03.7 | `03.7-signoff-success/` | Shadow record confirmed, brief card transitions to "Cleared (shadow matched)" | ✓ |

---

## Open Questions (for Phase 4)

- **Push notification timing tolerance** — PRD says "14 minutes before HAFAS scheduled arrival." If the brief generation has a delay (e.g., IntentPacket arrived late), does the push fire late or skip? Need a Phase 4 spec for the "brief was generated less than 14min before arrival" case.
- **Cancellation during countdown — UI feedback** — when Klaus releases mid-countdown, does the UI show a cancellation toast? Or silent? Influences how he experiences accidental release vs. deliberate cancel.
- **Sign-off success → brief card state** — does the brief stay open (so Klaus can review other faults on the same vehicle), or auto-close? Phase 4 spec.
- **`role_relevance` taxonomy sign-off** — open decision Q1 in `ux-decisions-2026-05-23.md`. The card-detail depth in this scenario assumes "ECM Manager view = executive summary." Locked taxonomy is a Phase 4 prerequisite.
