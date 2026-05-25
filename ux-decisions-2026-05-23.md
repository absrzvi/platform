# UX Decisions — Party Mode Session
**Date:** 2026-05-23
**Participants:** Freya (UX), Saga (Analyst), Mary (Business Analyst)

---

## Q1 — Conductor App: Degraded Sensor State

### Design Approach
Coverage gap framing, not system status framing. Never show silence — always show system state.

### Train Schematic Coverage Indicator (persistent header)
- **Green segment** — inference running, confidence above threshold
- **Amber segment** — partial coverage (one camera offline, SNMP slow), inference running but flagged
- **Red segment** — blind spot, no inference for this segment
- Segment pulses once on state change, then settles. No banner. No push interruption.

### Tap-to-Detail (one screen, no nav depth)
"Car 4 — camera VLAN 5 unresponsive. Occupancy estimate unavailable. Last known: 68% (12 min ago)."

### Stale IntentPacket Rendering
- Fresh packet: normal card rendering
- Approaching expiry: timestamp label "Recommendation (42s ago)"
- Expired (`expires_at` passed): muted card, "Expired — conditions may have changed." Card stays visible — do not delete or suppress.

### Must Specify Before Story
1. Confidence score thresholds — what value moves segment amber vs red? ⚠️ OPEN DECISION
2. Staleness rendering spec — visual treatment for expired vs fresh cards
3. Tap-to-detail copy per degraded source type (camera, SNMP, APC)
4. Explicit product decision: no push interruption for degradation — passive indicators only

### Psychology (Saga)
Conductor's deepest fear: being wrong in public. Silence reads as green — that's the danger. The system is their data alibi. Advisory with qualifier ("Occupancy estimate: moderate — camera unavailable, door counts only") preserves their judgment. They still decide, but they know they're deciding with partial data.

---

## Q2 — Depot Briefing Interface: Four Roles, One Interface

### Design Approach
One information model, role-filtered presentation. Not tabs — progressive disclosure by role.

### Role Selection
- Shared depot devices: role selector on every open (4 large touch targets, one tap, no confirmation, clears after 4 hours)
- Personal phones: stored role preference after first selection
- Role selector: ECM Manager | Electrical | Mechanical | IT

### Card Structure (all roles see same cards, filtered + sorted differently)
Each card has:
- Severity band (left edge, colour-coded)
- System label
- Plain-language summary
- Detail chevron

**Sort order per role:**
- ECM Manager → by severity (critical first)
- Electrical Technician → electrical systems first (power, cameras, doors)
- Mechanical Technician → mechanical systems first (brakes, HVAC, door mechanisms)
- IT Technician → network/firmware first (camera firmware, SNMP, switch ports)

⚠️ OPEN DECISION: Sort order per role needs ECM Manager sign-off

**Card depth per role:**
- ECM Manager: executive summary (what failed, impact, recommended action)
- Technicians: technical detail (fault code, port, last known good state, diagnostic step)

**Card visibility:**
- Brake wear alert not shown to IT Tech — not hidden, not present
- Defined by `role_relevance[]` field on each card

### Card Schema
```
severity: CRITICAL | WARNING | INFO
system: str
summary: str                    # plain English — for ECM Manager
technical_detail: str           # technical — for technicians
role_relevance: list[Role]      # which roles see this card
location_ref: str | None        # e.g. "Car 3 Door 2L"
```

### Accountability Chain (Mary)
- **Named ACK per role** — each fault requires explicit ACK or DEFER with reason. Timestamped record, not a checkbox.
- **Fault ownership at triage** — ECM Manager assigns primary owner for cross-domain faults (e.g. camera power fault spanning Electrical + IT)
- **Audit trail** — ECM Manager view shows per fault: who assigned, when acknowledged, what action taken. Visible before a miss happens, not reconstructable after.

### LLM Prompt Requirements
One briefing document generated per train per depot arrival. Role lens applied client-side (or API layer — specify in story). LLM generates both `summary` (plain English) and `technical_detail` (technical) fields per card. Tone differentiation is a prompt requirement, not a frontend concern.

### Must Specify Before Story
1. Role persistence model — session-only on shared device, stored on authenticated personal device
2. Card schema finalised (above is draft)
3. Sort order per role — ECM sign-off required
4. Two depth levels max — summary stack + single card expanded, nothing deeper
5. LLM prompt spec for tone differentiation

---

## Q3 — Fleet Manager AI Interface: Task Model + Three Entry States

### Three Modes

| Mode | Frequency | Session | AI role | Entry trigger |
|---|---|---|---|---|
| Morning triage | Daily | 10–20 min | Proactive summary | Time-based, pre-service window |
| Live crisis | 2–3×/week | 2–5 min bursts | Reactive assistant | Active incident detected |
| Trend review | Weekly | 30–60 min | Analysis partner | Intentional nav |

### Entry State Logic
⚠️ OPEN DECISION: Pre-service time window — suggested 05:30–07:00, confirm with ÖBB ops schedule

- **Pre-service window + no active incidents** → Morning Brief. 70/30 layout (fleet view left, overnight digest right). AI chat closed by default — single line "Ask about the fleet."
- **Active incident detected** → Modal on open (not banner). "Train 4712 — critical door fault, Car 6. View incident?" Full-width incident view, AI chat pre-seeded with incident context at bottom. Modal fires every open with active unresolved incident until dismissed or resolved.
- **Intentional trend navigation** → "Trends" in nav bar. 30/70 layout (map left, analysis right). AI chat expanded by default. User is asking, not reading.

### Navigation
Three items: **Fleet** (triage default) | **Incidents** (crisis view) | **Trends**

### AI Chat State Per Mode
- **Triage mode**: closed by default. Single line "Ask about the fleet." Tap to expand.
- **Crisis mode**: open, pre-seeded with incident context. Ready to answer "what caused this."
- **Trend mode**: fully expanded. Primary interface.

### Three Layout States
1. **Triage** — 70/30 fleet view / overnight digest
2. **Crisis** — full-width incident + AI chat panel at bottom
3. **Trend** — 30/70 map / analysis, AI chat primary

⚠️ Each layout state needs a wireframe spec before dev starts

### Must Specify Before Story
1. Entry state detection logic — document as state machine (time of day + active incident flag + last session)
2. Pre-service time window — confirm with ÖBB ops schedule
3. Incident modal — fires every open with active unresolved incident
4. AI chat pre-seeding template for crisis mode — backend spec required
5. Three layout wireframes before dev

### Fleet Manager — Evidence Layer for Commercial Sign-off (Mary)

Five success criteria → five evidence panels for Roland Ruiz and Martin Lerch:

| Criterion | Evidence Panel |
|---|---|
| Fault detection ≤60s | Detection latency distribution (P50/P95/P99) by fault type. Not just average. |
| False positives <5% | False positive rate by fault category, by week. Trend matters more than snapshot. |
| ECM structured summary before arrival | % of arrivals with pre-arrival ECM review. Target: >80%. Log IntentPacket generated time vs docking time vs ECM open time. |
| >50% dispatcher adoption | Adoption by action type — of alerts surfaced, % resulting in dispatcher-initiated action within 30 min. Opens without action ≠ adoption. |
| Rollout recommendation within 6 months | Pilot health score — composite of 4 metrics above, updated weekly, with visible threshold line. |

**Counterfactual evidence panel (key to closing the deal):**
"Prevented incidents" — faults detected before service disruption, cross-referenced against historical delay data for same fault class on unmonitored units. Example: "7 faults prevented. Historical baseline: 23-min delay per incident. Estimated delay-minutes prevented: 161." This is the business case artifact Roland and Martin need to sign.

The depot accountability ACK trail makes it credible — every fault has a timestamped detection → assignment → resolution record.

---

## Open Decisions (must resolve before stories)

| # | Decision | Owner | Blocker for |
|---|---|---|---|
| 1 | Confidence score threshold for amber vs red coverage segment | Abbas + data science | Conductor App story |
| 2 | Depot card sort order per role | ECM Manager sign-off | Depot Briefing story |
| 3 | Fleet Manager pre-service time window | ÖBB ops schedule | Fleet Manager story |
| 4 | Fleet Manager entry state machine | Abbas | Fleet Manager story |
| 5 | Incident modal — dismiss vs resolve behaviour | Abbas | Fleet Manager story |
