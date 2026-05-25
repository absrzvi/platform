---
stepsCompleted: [step-01-init, step-02-discovery, step-02b-vision, step-02c-executive-summary, step-03-success, step-03-dr, step-04-journeys, step-04-journeys-p2, step-05-domain, step-06-innovation, step-07-project-type, step-08-scoping, step-09-functional, step-10-nonfunctional, step-11-polish]
releaseMode: single-release
inputDocuments:
  - platform/party-mode-critical-assessment-2026-05-23.md
  - platform/ADR-001-conductor-app-transport.md
  - platform/intent-packet-schema-v1.0.0.md
  - platform/ux-decisions-2026-05-23.md
  - platform/oebb-role-research-2026-05-23.md
  - platform/ux-decisions-update-1-2026-05-23.md
  - platform/rail-mcp-server-spec-2026-05-23.md
workflowType: 'prd'
classification:
  projectType: Context-differentiated decision-support platform (mobile for exception handling under time pressure; web for situational awareness and planning; AI agent interface for fleet analytics)
  domain: Rail operations — safety-adjacent, regulated (EU ECM Directive 2019/779, NIS2, GDPR)
  complexity: High + state-sync complexity — three independent state machines (offline mobile, edge inference, regulated audit trail) must stay consistent under degraded connectivity
  projectContext: greenfield — new product built on oebb-agent Phase 1 infrastructure
  deploymentModel: Single ÖBB instance (PoC); multi-tenant upgrade path at fleet rollout
  aiPosture: Advisory only — LLM suggests, human decides, system records. Platform is not liable for train safety decisions.
---

# Product Requirements Document — ÖBB Unified Rail Operations Platform

**Author:** AbbasRizvi
**Date:** 2026-05-23

---

## Executive Summary

ÖBB rail operators — conductors, depot technicians, fleet coordinators, dispatchers, and ECM managers — currently make high-stakes decisions with fragmented, delayed, or missing information. The platform closes that gap across all five roles, serving two fundamentally distinct modes:

**Live decision support** — conductors, dispatchers, and fleet coordinators acting under time pressure receive contextual, actionable information at the moment of decision, with no internet dependency. LLM inference runs onboard over the train's L2 network (Hailo-10H M.2), drawing from live data probed from the Stadler Diagnostic system (stadler IM). No cloud round-trip. No WAN latency. No blackout in a tunnel. When inference is unavailable or degraded, the system signals its state explicitly — the conductor always has last call, and silence is never presented as green.

**Post-incident forensics and compliance evidence** — ECM managers and technicians receive a pre-arrival fault brief, a role-filtered maintenance view, and an append-only audit trail anchored to EU Directive 2019/779. ECM sign-off records are append-only — corrections are new rows with a `supersedes_id`, `technician_id` is bound to ÖBB SSO, not a local session. When connectivity is unavailable for an extended period, the audit trail queues locally and syncs on reconnect. The platform does not replace regulatory judgment — it makes it legally defensible.

Onboard interfaces account for prolonged shift fatigue and movement — a conductor at hour 9 of a 10.5h shift reads UI differently than at hour 1. Landside tools assume desk stability. This distinction shapes information density, interaction cost, and alert interruption design across every interface.

The platform surfaces what humans need to decide faster. Every decision is made by a human. The platform records it.

### What Makes This Special

**Prevented delay-minutes and ECM compliance evidence** — measurable KPIs the Fleet Manager and ECM Manager are already accountable for — are counterfactual evidence legacy systems cannot produce. The platform makes them visible.

Every other rail AI platform requires cloud connectivity for inference. This one does not. The Hailo-10H M.2 runs a 1–2B parameter model onboard, drawing from live CCTV (VLAN 5), door sensors (VLAN 8), SNMP telemetry (VLAN 7), PIS/FIS (VLAN 3), and reservation data (VLAN 6). Target: sub-5-second safety query latency (to be confirmed under Hailo-10H load test).

The ECM audit trail is the second differentiator. It is not a convenience feature — it is the compliance record required under EU 2019/779 for every vehicle returning to service, structured to survive regulatory inspection.

Conductors have no existing digital decision support tools. This platform is entirely additive — no workflow replacement, no change management friction, no union risk. Trust is earned over the first 10 shifts through a confirmatory model: the system shows what the conductor already knows first, earns credibility, then layers advisory intelligence.

### Pilot Success Criteria

| Criterion | Target | Evidence method |
|---|---|---|
| Fault detection latency | ≤60s (P95) | Detection timestamp vs event timestamp, by fault type |
| False positive rate | <5% per fault category | Weekly trend, not snapshot |
| ECM pre-arrival review | >80% of arrivals | IntentPacket generated time vs docking time vs ECM open time |
| Dispatcher adoption | >50% of surfaced alerts result in dispatcher-initiated action within 30 min | Action type breakdown — opens without action ≠ adoption |
| Rollout recommendation | Within 6 months of pilot start | Composite pilot health score updated weekly with visible threshold line |

### Project Classification

- **Project Type:** Context-differentiated decision-support platform — mobile for exception handling under time pressure; web dashboard for situational awareness and planning; AI agent interface for fleet analytics/reports and intellegence information
- **Domain:** Rail operations — safety-adjacent, regulated (EU ECM Directive 2019/779, NIS2, GDPR)
- **Complexity:** High + state-sync — three independent state machines (offline mobile, edge inference, regulated audit trail) must stay consistent under degraded connectivity
- **Project Context:** Greenfield — new product built on oebb-agent Phase 1 infrastructure (Hailo-8 vision pipeline, PostgreSQL/FastAPI/React stack)
- **Deployment Model:** Single ÖBB instance (PoC); multi-tenant upgrade path at fleet rollout
- **AI Posture:** Advisory only — LLM suggests, human decides, platform records

---

## Success Criteria

### User Success

| Persona | Success moment | Measurable signal |
|---|---|---|
| Conductor | Acts on an alert that proves correct within 10 shifts — system has earned credibility | Conductor-initiated actions after advisory, tracked per shift number (confirmatory → advisory transition) |
| ECM Manager | Arrives at depot having already reviewed the fault brief before docking | >80% of arrivals with IntentPacket opened before vehicle arrives |
| Technician | Faults assigned to the right discipline on first triage — no cross-handover churn | Fault reassignment rate per role per vehicle |
| Fleet Manager | Makes a redeployment decision with cost consequence visible before confirming | Time-to-decision for redeployment calls, measured before/after |
| Dispatcher | Forwards a plain-language incident update to conductor without rewriting it | % of relay card forwards vs manual incident summaries written |

### Business Success

- **Prevented delay-minutes:** Faults detected before service disruption, cross-referenced against historical delay data for same fault class on unmonitored units. Target: produce counterfactual evidence for at least 7 incidents in pilot period.
- **ECM audit trail completeness:** Sign-off timestamp, briefing-read timestamp, and fault resolution summary present for >95% of vehicle returns to service during pilot.
- **Fleet Manager decision latency reduction:** Measurable reduction in time-to-decision for redeployment calls compared to baseline (pre-platform). Baseline measured in pilot weeks 1–2, post-platform measured weeks 4–6.
- **Rollout recommendation:** Composite pilot health score (fault detection latency + false positive rate + ECM review rate + dispatcher adoption) crosses threshold within 6 months. Presented to Roland Ruiz, Martin Lerch, and ECM-accountable role (Head of Vehicle Maintenance or CSO).

### Technical Success

- Fault detection latency ≤60s at P95, measured per fault type
- False positive rate <5% per fault category, tracked weekly as trend (not snapshot)
- IntentPacket delivery: QoS 1 retain=True ensures reconnecting consumers receive current state immediately — no missed alerts on tunnel exit
- ECM sign-off records: zero UPDATE/DELETE events on `ecm_signoff_events` table in any audit period
- Conductor App: offline-capable — all alerts visible with no internet connectivity; sync resumes on reconnect without data loss
- Hailo-10H inference: target sub-5-second safety query latency (to be confirmed under load test)
- Fleet Manager semantic search: natural language query returns relevant incidents within 3 seconds at P95

### Pilot-Kill Triggers

The following conditions pause the pilot pending investigation. Roland Ruiz and Martin Lerch must be notified within 24h of any trigger firing.

| Trigger | Threshold | Action |
|---|---|---|
| Hailo-10H inference P99 sustained | >10s for two consecutive measurement windows | Pause Conductor App LLM queries; revert to confirmatory-only mode |
| False positive rate | >10% for any fault category over two consecutive weeks | Suspend advisory for that alert class; review detection model |
| ECM sign-off write failure | Unresolved >24h | Halt depot sign-off via platform; revert to paper process |
| pgvector semantic search unavailable | Keyword fallback active >48h continuous | Flag to Martin Lerch; infrastructure review required |
| Conductor App offline gap | >72h without sync on any active train | Review edge event store and MQTT broker connectivity |

### Measurable Outcomes (Pilot Evidence Targets)

| Evidence | Method | Target |
|---|---|---|
| ECM trail completeness | Log IntentPacket generated time vs docking time vs ECM open time vs sign-off time | >80% pre-arrival review rate |
| Fleet Manager decision latency | Instrument time-to-decision for redeployment calls before/after | Measurable reduction |
| Audit trail usage correlation | Sign-off timestamp vs briefing-read timestamp — did reading the brief change the outcome? | Qualitative + quantitative |
| Conductor additive behaviour | 3+ documented new conductor-initiated tasks per week (not click-throughs — interaction log per alert type distinguishes chosen action from passive confirmation) | ≥3 new task types catalogued over 6-week pilot |

**Baseline methodology (prevented delay-minutes):** Historical delay data for each fault class on unmonitored units is collected in pilot weeks 1–2 before platform advisory is active. Attribution method: fault detected by platform → no service disruption → cross-reference against historical mean delay for same fault class on unmonitored fleet. Methodology agreed with Roland Ruiz before pilot week 3. Without this, post-pilot counterfactual argument is inadmissible.

**ECM completeness gap (80% → 95%):** The 80% pre-arrival review target is platform-delivered. The 95% business success target requires process change (ECM Manager opens brief before leaving for depot, not on arrival). Martin Lerch owns the process workstream. Platform cannot close this gap alone — must be agreed and signed off before pilot measurement begins.

**Leading success signals (per persona):**

| Persona | Leading signal | When measured |
|---|---|---|
| Conductor | Comprehension check — can conductor explain why an alert fired, unprompted? | Shifts 1–2 (confirmatory phase) |
| ECM Manager | Trust signal — does manager act on pre-arrival brief without re-verifying onsite? | Week 2 of pilot |
| Technician | First-assign accuracy — fault goes to correct discipline without reassignment | Week 1 onward |
| Fleet Manager | Decision with cost visible — redeployment confirmed before cost one-liner is dismissed | Week 2 onward |
| Dispatcher | Relay card used unprompted — without being shown the feature | Week 1 onward |

---

## Product Scope

### MVP — Minimum Viable Product

Three interfaces, delivered in priority order:

1. **Conductor App** — PWA served from onboard AP. Active alerts from IntentPacket feed, departure clearance checklist (brake test → doors → dispatch comms → hold-to-confirm), coach occupancy overview, degraded sensor state (coverage segment per carriage), shift-aware density reduction after 7h threshold. Confirmatory mode for first 10 shifts.
2. **Depot Briefing Interface** — PWA, mobile-first. Role-filtered fault feed (ECM Manager / Electrical / Mechanical / IT), ECM 1 blocking authorisation state, append-only sign-off with IdP-bound identity, pre-arrival brief generated from IntentPackets. BOOM integration descoped — structured export/copy for manual BOOM entry.
3. **Fleet Manager AI Interface** — The sole landside control surface for this platform. Hermes BYOK, Rail MCP Server (9 tools), three entry states (triage 70/30, crisis full-width, trend 30/70), pgvector semantic search over incident summaries, cost-impact one-liner per triage card, shift handover summary, relay card for Dispatcher forwarding. Note: the existing oebb-agent React dashboard is a separate product and is not extended by this platform.

**Infrastructure MVP:**
- `shared/schemas/intent_packet.py` — IntentPacket v1.0.0 frozen schema
- `rail-mcp-server/` — 9-tool HTTP MCP endpoint (spec frozen)
- `ecm_signoff_events` + `shift_handovers` tables in PostgreSQL (schemas defined)
- 4 Hermes spikes run and findings documented (pre-story gate)
- pgvector fallback ranking strategy defined and spiked before Fleet Manager stories (Winston gate)
- ECM schema immutability audit runs in parallel with Hermes spikes — not sequentially after

### Growth Features (Post-MVP)

- Conductor onboarding trust model: automated transition from confirmatory → advisory mode based on demonstrated accuracy over first 10 shifts
- Fleet Manager 48–72h forward planning view alongside real-time triage
- Shift handover cross-session AI conversation context (out of scope for PoC)
- BOOM API integration when API contract confirmed with ÖBB IT
- Multi-tenant deployment — credential isolation, per-operator Rail MCP Server instances
- Conductor App Capacitor wrapper if native device features required post-pilot

### Vision (Future)

- Hermes agents that proactively surface fleet patterns without query — push-based fleet intelligence
- ECM audit trail integration with ÖBB's internal compliance documentation system
- Platform extended to additional ÖBB roles: Driver Display, Bistro App, Station View
- Federated model updates: domain-specific fine-tuning pushed to Hailo-10H during depot sync windows
- Multi-operator rail network deployment (non-ÖBB carriers)

---

## Disaster Recovery

### Onboard (Edge)

| Failure | Behaviour | Recovery |
|---|---|---|
| Hailo-10H inference failure | Conductor App shows degraded state explicitly — coverage segment turns red, alert card shows "Inference unavailable". Silence is never presented as green. | Automatic restart via Docker restart policy. Conductor retains last known state until fresh packet arrives. |
| VLAN source loss (e.g. VLAN 5 CCTV) | IntentPacket emitted with affected `source_signals` absent. Conductor App coverage segment reflects partial coverage (amber). `location_ref` shows "Last known: Xmin ago". | No manual intervention required. Recovers when VLAN source resumes. |
| Onboard L2 network partition (SYS2 internal) | IntentPacket emission halts. Conductor App renders last retained packet (MQTT retain=True) with staleness label. Expired packets muted, not deleted. | System self-heals on network recovery. No data loss — edge event store holds local queue. |
| Edge event store (SQLite) corruption | Events since last sync lost. Landside audit trail has gap flagged by sync cursor. | Sync cursor detects sequence gap, logs anomaly, does not advance past gap. Gap flagged in Fleet Manager triage view. |

### Landside

| Failure | Behaviour | Recovery |
|---|---|---|
| PostgreSQL unavailable | ECM sign-off writes queue locally, sync on reconnect. Fleet Manager AI Interface shows stale data with timestamp. Rail MCP Server returns 503 with `recoverable: true`. | Depot assumed to have fixed network — extended outage triggers ops alert. ECM sign-off writes resume on reconnect; append-only guarantee preserved. |
| pgvector index failure | Fleet Manager semantic search degrades to keyword fallback ranking (Rail MCP Server `search_fleet_events` falls back to full-text). No silent failure — UI labels results "Keyword search active". | pgvector restart clears index; re-index from `incident_summary` embeddings. Spike required pre-Fleet Manager stories to define fallback behaviour and re-index SLA. |
| Hermes BYOK API unavailable (429 / 5xx) | Fleet Manager AI chat shows "Fleet intelligence temporarily unavailable". Triage and crisis views remain fully functional — they do not depend on Hermes. Retry with exponential backoff + budget cap. | BYOK 429 handling is Hermes spike 4 — must be run and findings documented before Fleet Manager stories begin. |
| Rail MCP Server tool failure | Hermes receives structured error `{error, detail, recoverable}`. Recoverable errors retried once. Non-recoverable surfaced to Fleet Manager as "Data source unavailable for [tool name]". | Each tool is independently recoverable — one tool failure does not bring down the MCP Server. |

### ECM Audit Trail Integrity

- `ecm_signoff_events` is append-only enforced at database level (row-level trigger raises exception on UPDATE/DELETE attempt)
- Corrections are new rows with `supersedes_id` pointing to corrected record — no in-place mutation ever
- `signature` field (HMAC) provides tamper evidence for each row
- Extended connectivity loss: sign-off writes queue locally at depot, sync to landside PostgreSQL on reconnect. Queue is durable — depot device loss before sync is an unrecoverable gap, flagged in audit as missing record with last-known timestamp
- Sign-off 5-second confirmation window: record is not committed until confirmation window elapses. Cancellation within window prevents write. After window, record is permanent — no undo

### Conductor App Offline Resilience

- All active IntentPackets cached in IndexedDB via Workbox service worker
- MQTT retain=True ensures reconnecting app receives current brain state immediately on subscribe — no polling gap
- Expired packets (`expires_at` passed) are muted in UI but not deleted from cache — available for post-incident review
- Extended offline (tunnel): app operates from cached state, staleness label shown, conductor notified of last-known data age
- On reconnect: WebSocket re-subscribes, fresh IntentPacket overwrites cache, staleness label clears

---

## User Journeys

### Journey 1 — Conductor: "The Door Alert at Speed" (Primary — Success Path)

**Meet Mia.** Mia is a Zugbegleiterin on the 06:42 Salzburg → Wien service. She is three hours into a 10.5-hour shift, moving through coaches checking tickets. She has worked this route for six years and has never had a digital decision support tool — every judgment call has been hers alone.

**The moment.** Car 4, Door 2L. The Hailo-8 vision pipeline detects a bag partially obstructing the door sensor zone. The Hailo-10H correlates this with a SNMP door motor fault on the same door. An IntentPacket is emitted: `DEGRADED_DWELL_RISK`, severity WARNING, confidence 0.84, expires in 120s.

**What Mia sees.** Her Conductor App — a PWA on the train WiFi — pulses once on the coverage segment for Car 4. No banner. No push interruption. She glances at it between tickets: amber segment, Car 4. She taps: "Car 4, Door 2L — door motor fault + possible obstruction. Last safe dwell: 34s ago." She is in Car 3. She knows this. She walks through.

**The climax.** She arrives at Car 4 before the next stop. The bag is there. She moves it. She taps the alert card: "Resolved — manual." The card clears. The coverage segment returns to green. No delay. No passenger complaint. No incident report.

**The new reality.** The system did not tell Mia what to do. It told her where to look. She made the call. The platform recorded it — detection timestamp, conductor action, resolution time. That is the counterfactual evidence Roland needs.

*Capabilities revealed: IntentPacket alert rendering, coverage segment per carriage, tap-to-detail, conductor ACK flow, confidence score display, stale card handling.*

---

### Journey 2 — Conductor: "Sensor Blind Spot at Night" (Primary — Edge Case / Degraded State)

**Meet Mia again.** Hour 9. The Hailo-8 CCTV feed on VLAN 5 for Car 6 drops — camera VLAN unresponsive. It is 23:15 on a long-distance night service. She is tired. The train is quiet.

**What Mia sees.** Car 6 coverage segment turns red. No inference for this segment. She taps: "Car 6 — camera VLAN 5 unresponsive. Occupancy estimate unavailable. Last known: 12% (18 min ago)." The UI is denser than it was at hour 1 — the shift-aware density reduction has already activated at hour 7, hiding secondary metrics and increasing contrast.

**The edge.** No alert fires for Car 6 — there is nothing to alert on. But Mia knows the system is blind there. She walks through Car 6 manually. Nothing unusual.

**The resolution.** The blind spot is logged. The landside event store records it. If something had happened in Car 6 during that window, the audit trail shows: system was blind, conductor completed manual walkthrough at 23:18. The platform's silence is documented, not assumed.

*Capabilities revealed: Degraded sensor state rendering, shift-aware density reduction, stale data labelling, manual walkthrough as documented action, blind spot logging.*

---

### Journey 3 — ECM Manager: "The Pre-Arrival Brief" (Secondary — Success Path)

**Meet Klaus.** Klaus is an ECM 1 Manager at Wien Westbahnhof depot. He is accountable under EU Directive 2019/779 for every vehicle that returns to service. He arrives cold — he was not on shift when the train came in last night. Unit 4722 is in Bay 3.

**Before the platform.** Klaus walked to Bay 3, asked the technician what happened, read a paper fault log, and made a judgment call. If he missed something and the vehicle failed in service, the liability was his and the paper trail was thin.

**Now.** At 05:50, fourteen minutes before Unit 4722 docks, the Hailo-10H generates a depot briefing from the journey's IntentPackets. Klaus opens the Depot Briefing Interface on his phone. Role: ECM Manager. He sees: two faults sorted by severity. CRITICAL: Door motor fault, Car 4 Door 2L — resolved by conductor at 07:14, replacement part fitted at 03:22 by Electrical team. WARNING: SNMP camera firmware version mismatch on VLAN 5 — acknowledged by IT technician, deferred to next maintenance window with reason logged.

**The climax.** Klaus taps "Review & Authorise" on the door motor fault. He reads the full resolution chain — detection → conductor ACK → technician assignment → part fitted → test result. He holds to confirm. The `ecm_signoff_events` record writes: his IdP identity, UTC timestamp, full vehicle state snapshot at sign-off, directive reference EU 2019/779.

**The new reality.** Unit 4722 clears to service at 06:08. Klaus has a legally defensible audit trail. The platform did not authorise the vehicle — Klaus did. The platform made his decision traceable.

*Capabilities revealed: Pre-arrival brief generation, role-filtered fault feed, ECM 1 blocking state, hold-to-authorise gesture, append-only sign-off, full resolution chain view.*

---

### Journey 4 — Fleet Manager: "The Crisis Redeployment" (Operations — Success Path)

**Meet Stefan.** Stefan is a Fleet Manager at ÖBB Wien operations. He is responsible for cost management through fleet deployment across a 12-hour shift. It is 07:23. He opens the Fleet Manager AI Interface.

**The modal fires.** "Unit 4722 — Door Motor Fault, Car 4. Active unresolved incident. View?" He taps yes. Full-width crisis view. AI chat pre-seeded: "Unit 4722 door motor fault detected at 06:14. Conductor resolved manually at 07:14. Vehicle cleared by ECM at 06:08. Replacement unit available: Unit 4819."

**Stefan queries.** "What does deploying 4819 cost versus keeping 4722 on route?" Hermes calls `get_maintenance_cost_estimate(unit_id="4722", replacement_unit_id="4819")`. Response in 2.1 seconds: "Replacing with Unit 4819 adds ~€2,400 estimated maintenance overhead. Confidence: medium. Rationale: 4819 last serviced 34 days ago, approaching mileage threshold." The cost one-liner was already in the triage card before he typed the question.

**The decision.** Stefan decides to keep 4722 on route — the fault is resolved, the ECM has authorised it. He closes the modal. Decision timestamp logged. Cost data available at decision time: confirmed.

**The new reality.** Stefan made a €2,400 decision in 90 seconds with evidence. Prevented delay-minutes: the 14-minute delay was contained at conductor level and never escalated to redeployment. The counterfactual is logged.

*Capabilities revealed: Crisis entry state, incident modal, pre-seeded AI chat, Rail MCP tool calls, cost-impact one-liner, triage card, decision timestamp logging.*

---

### Journey 5 — Dispatcher: "The Relay" (Secondary — Coordination Path)

**Meet Ana.** Ana is a Fahrdienstleiter at Salzburg control room. She monitors signal and switch systems across her section. At 14:32 an IntentPacket arrives: Train 4712, 14-minute delay, signal fault at Salzburg Hbf.

**What Ana sees.** In the Fleet Manager AI Interface, alongside the triage alert, a relay card appears: "Train 4712 is delayed 14 minutes due to signal fault at Salzburg Hbf. Expected recovery: on departure." Plain language. Passenger-safe. No fault codes.

**The action.** Ana taps "Forward to Conductor." The relay card is pushed to the conductor's PWA notification. Ana taps "Copy" and pastes it into the PA system announcement. She did not rewrite anything. She did not call the conductor. Thirty seconds from alert to passenger announcement.

**The new reality.** The relay card was generated by the LLM from the IntentPacket `incident_summary`. Ana's job is coordination, not translation. The platform handled the translation.

*Capabilities revealed: Relay card component, LLM-generated passenger-safe language, copy-to-clipboard, forward-to-conductor push notification, Dispatcher coordination loop.*

---

### Journey 6 — System Degraded: What Three Personas See (Edge Case — Degradation Path)

**The scenario.** 09:14. Hailo-10H inference stalls mid-journey — VLAN 7 SNMP polling spike causes inference queue backup. No IntentPacket has been emitted for 80 seconds. The previous packet's `expires_at` passes.

**What Mia sees.** The alert card does not disappear. It mutes — greyed out, timestamp label: "Recommendation (83s ago) — conditions may have changed." The coverage segment holds its last state. No new pulse. No banner. She knows the system is thinking slowly. She keeps moving. Silence is never presented as green.

**What Stefan sees.** In the Fleet Manager triage view, the fleet card for Train 4712 shows a staleness indicator — last updated timestamp, amber border. He queries Hermes: "What is the current status of Train 4712?" Hermes calls `get_fleet_summary`. The Rail MCP Server responds with `data_freshness_ts` showing 94 seconds ago. Hermes surfaces this: "Last known status 94 seconds ago — inference delayed. Data may not reflect current state." Stefan waits. He does not act on stale data.

**MCP timeout.** Stefan asks for the cost estimate on a redeployment. The Rail MCP Server call to `get_maintenance_cost_estimate` times out after 8 seconds. The triage card shows: "Cost estimate unavailable — data source timeout." Stefan makes the redeployment decision without the cost figure. The platform records his decision with a flag: cost data unavailable at decision time.

**Recovery.** Inference resumes at 09:17. Fresh IntentPacket emitted. Mia's card updates — staleness label clears, coverage segment pulses once. Stefan's fleet card updates. No data is lost. No decision is silently affected.

*Capabilities revealed: Stale packet muting with label, coverage segment holding last state, Fleet Manager staleness indicator, MCP timeout handling — structured error surfaced to UI, decision logging with data-availability flag, automatic recovery on inference resume.*

---

### Journey 7 — The ECM Lead Reviews the Platform (Commercial Confidence Journey)

**Meet Dr. Renate Fischer.** Dr. Fischer is Head of Vehicle Maintenance at ÖBB Technische Services — the ECM-accountable role. She was not in the pilot planning meeting. Martin Lerch brings her in at pilot week 3. Her question is not "does it work?" — it is "does our audit trail map to our internal ECM documentation requirements, and would it survive a regulatory inspection?"

**What she reviews.** She opens the ECM audit trail export for Unit 4722. She sees: IntentPacket generated at 05:50 (14 minutes pre-arrival), opened by Klaus at 05:53, fault resolution chain — detection → conductor ACK → technician assignment → part fitted → test result, sign-off at 06:08 by Klaus (IdP identity, not a local username), vehicle state snapshot at sign-off, directive reference EU 2019/779, HMAC signature on each row.

**Her first question.** "Can I export this in a format our documentation system accepts?" PoC answer: structured JSON export, copy-paste ready. BOOM integration deferred — noted, not a blocker.

**Her second question.** "What happens if Klaus signs off incorrectly — can we correct it without deleting the record?" She is shown the `supersedes_id` pattern — a new row points to the original, original is never touched. She is satisfied.

**The outcome.** Dr. Fischer does not object. She requests to be a reference customer for the compliance output. Roland gets his ECM sign-off for the rollout recommendation. Late-stage objection risk eliminated.

*Capabilities revealed: ECM audit trail export, full resolution chain view, supersedes_id correction flow, IdP-bound identity display, HMAC tamper evidence, structured JSON export.*

---

### Journey 8 — The Novel Fault (Conductor — Unknown Territory)

**Meet Mia once more.** Hour 4. Route she knows. But the alert is one she has never seen: `HARDWARE_PORT_DISCONNECT`, Car 2, Camera 3, VLAN 5. Confidence: 0.91. Severity: WARNING.

**The moment.** She has 40 seconds before the next stop. She does not know what a hardware port disconnect means in practice. The system knows she is in confirmatory mode — shift 7. It does not assume she knows.

**What she sees.** The alert card shows: "Car 2 — camera hardware disconnected. Vision coverage lost for this segment. This fault type has occurred 3 times on this fleet in the last 90 days — all resolved without service impact. No door or passenger safety implication confirmed." Below: severity band AMBER. Below that: "You may continue. Monitor Car 2 manually at next stop."

**The decision point.** Mia reads it. She has seen the system be right six times in seven shifts. She continues. At the next stop she walks through Car 2 — nothing unusual. She taps: "Checked manually — no issue found."

**What the platform records.** Novel fault type → conductor action → outcome. This data feeds the confirmatory model: another correct advisory. The trust curve advances. By shift 10, the system knows Mia has seen this fault class and acted correctly. Advisory mode is earned, not assumed.

**The alternative path.** If Mia had tapped "Stop — escalating," the platform would surface the escalation to Stefan's Fleet Manager triage view, flagged as conductor-initiated, with her reason. No silent escalation. No ambiguity about who called it.

*Capabilities revealed: Novel fault context panel — prior incidents for same fault class, severity band, recovery action, explicit continue/escalate choice, conductor-initiated escalation flow to Fleet Manager, trust curve data logging per fault type per conductor.*

---

### Journey Requirements Summary

| Capability area | Revealed by |
|---|---|
| IntentPacket alert rendering + expiry | Journeys 1, 2 |
| Coverage segment per carriage (green/amber/red) | Journeys 1, 2, 6 |
| Degraded sensor state — explicit, never silent | Journeys 2, 6 |
| Shift-aware density reduction | Journey 2 |
| Confirmatory mode — earned trust over 10 shifts | Journeys 1, 8 |
| Pre-arrival depot brief generation | Journey 3 |
| Role-filtered fault feed (ECM/Electrical/Mechanical/IT) | Journey 3 |
| ECM 1 blocking authorisation + hold-to-confirm | Journey 3 |
| Append-only sign-off with full resolution chain | Journeys 3, 7 |
| Crisis entry state + incident modal | Journeys 4, 6 |
| Rail MCP tool call (cost estimate, fleet summary) | Journeys 4, 6 |
| Cost-impact one-liner per triage card | Journey 4 |
| Pre-seeded AI chat with incident context | Journey 4 |
| Relay card + LLM passenger-safe language | Journey 5 |
| Forward-to-conductor push notification | Journey 5 |
| Stale packet muting with label | Journeys 2, 6 |
| MCP timeout — structured error to UI, decision flag | Journey 6 |
| ECM audit trail export + supersedes_id correction | Journey 7 |
| Novel fault context panel + prior incidents | Journey 8 |
| Conductor-initiated escalation flow | Journey 8 |
| Trust curve data logging per fault type | Journey 8 |
| Stale packet muting trigger definition | All journeys |
| ROI evidence dashboard — prevented delay-minutes + cost | Journey 9 |
| Damaged trust recovery — confidence score + prior accuracy + dismissal as legitimate choice | Journey 10 |

**Stale packet muting rule (Winston):** Muting triggers when `expires_at` passes. Muted state persists until next valid IntentPacket arrives for that train/carriage segment. Controlled entirely by `expires_at` field — no manual override. Muted cards are greyed with staleness label and remain visible; they are never deleted from cache.

---

### Journey 9 — The ROI Conversation (Commercial — Pilot Week 3)

**Meet Roland and Martin.** It is pilot week 3. Roland Ruiz (operations) and Martin Lerch (IT) sit down with the pilot evidence dashboard. They are not assessing features — they are assessing whether to sign a rollout recommendation.

**What they see.** The pilot health score composite: fault detection latency P95 at 44 seconds (target ≤60s ✓), false positive rate 3.2% (target <5% ✓), ECM pre-arrival review rate 83% (target >80% ✓), dispatcher adoption 54% (target >50% ✓). Below: prevented delay-minutes. Seven faults detected pre-disruption. Historical baseline for same fault classes on unmonitored units: 23-minute mean delay. Estimated delay-minutes prevented: 161.

**Martin's question.** "What does this cost us to run per train per month?" The platform surfaces infrastructure cost estimate: Hailo-10H amortised over fleet, container hosting, Rail MCP Server compute. The number is there. Roland cross-references against 161 delay-minutes prevented — at ÖBB's internal cost-per-delay-minute figure, the platform pays for itself in under two months of full fleet deployment.

**The outcome.** Roland does not need to argue the case to his CFO. The platform produced the evidence. Martin does not need to justify IT infrastructure spend — the ROI is in the dashboard. The rollout recommendation is signed at week 6, not week 24.

*Capabilities revealed: Pilot evidence dashboard — composite health score with threshold line, prevented delay-minutes counterfactual panel, per-criterion trend view, infrastructure cost estimate input, export for executive presentation.*

---

### Journey 10 — Damaged Trust Recovery (Conductor — After a False Alert)

**Meet Mia, shift 14.** Two weeks into using the platform. On shift 11, the system fired a `VESTIBULE_CONGESTION` alert for Car 3 — confidence 0.71. Mia walked through. The vestibule was empty. A camera obstruction had caused a false positive. She noted it. The system logged it.

**Now on shift 14.** A new `VESTIBULE_CONGESTION` alert fires for Car 5. Same alert type. Mia's trust in this alert class is lower than it was on shift 7.

**What she sees — differently this time.** The alert card shows the confidence score prominently: 0.62 — amber border, "Low confidence" label. Below the summary: "This alert type: 1 false positive in last 14 days on your route. Accuracy rate: 83%." Below that: two explicit choices — "Check manually" and "Dismiss — not credible." Dismissal is not hidden. It is not a warning. It is a legitimate option the platform presents without judgement.

**Mia dismisses.** She has context the system does not — she just walked through Car 5 thirty seconds ago during ticket checking. The alert fires 28 seconds after her walkthrough. She dismisses: "Already checked — clear." The platform records: alert fired, conductor dismissed with reason, time since last manual check: 28 seconds. No false positive logged — contextually valid dismissal.

**What the trust model learns.** Alert accuracy rate for this alert class on this route is updated. If the pattern holds — conductor dismissals correlate with camera obstruction conditions — the confidence threshold for this alert type on this route can be tuned. The platform gets better because Mia pushed back.

**The design principle this reveals.** Dismissal is not failure. It is signal. The platform must surface enough context (confidence score, prior accuracy, time since last manual check) for a conductor to make a *reasoned* dismissal — not a frustrated one. And it must never penalise dismissal in the UI.

*Capabilities revealed: Confidence score prominently displayed on low-confidence alerts, prior accuracy rate per alert type per route, explicit dismiss option as equal-weight choice (not secondary/hidden), contextual dismissal reason capture, trust model feedback loop — conductor dismissals feed alert threshold tuning, no UI penalty for dismissal.*

---

## Domain-Specific Requirements

### Compliance & Regulatory

**EU ECM Directive 2019/779 (Entity in Charge of Maintenance)**
- Every vehicle returning to service must have a documented ECM sign-off by an authorised ECM 1 entity
- Sign-off record must identify the authorising individual by their regulatory identity (ÖBB SSO IdP — not a local session username)
- Audit trail must be tamper-evident and append-only — no UPDATE or DELETE permitted on `ecm_signoff_events`
- Corrections are new rows with `supersedes_id` referencing the corrected record
- Vehicle state at moment of sign-off must be captured as a JSONB snapshot
- `directive_ref` field (`EU 2019/779`) present on every sign-off record

**GDPR**
- Raw video never leaves the train — Hailo-8 processes frames in-memory only; no video is written to disk or persisted in any store. Only anonymised event metadata (bounding box coordinates, occupancy counts, fault classifications) is transmitted landside
- No personally identifiable passenger data in any IntentPacket or event envelope
- `journey_id` scheme (`{vehicle_id}_{trip_number}_{YYYYMMDD}`) carries no personal identifiers
- Events tagged for deletion scope per GDPR Article 17 — retention policy required before fleet rollout

**NIS2 (Network and Information Security Directive)**
- Platform classified as critical infrastructure operator — ÖBB's NIS2 classification applies
- Incident reporting obligations: material cybersecurity incidents must be reported to national authority within 24h
- BYOK API keys for Hermes must never appear in source control, logs, or inference code paths
- Prompt injection threat model: all LLM output sanitised before reaching UI render layer
- VLAN isolation enforced onboard — credentials never pass through inference code paths

**ÖBB Internal Safety Framework**
- Platform advisory only — no automated safety commands issued to ETCS or train control systems (VLAN 2 ZFR is read-only)
- Conductor retains sole decision authority for all safety-adjacent actions
- Platform does not replace any existing ÖBB safety certification or TSI compliance obligation
- Safety advisory features require documented acknowledgement from ÖBB safety authority before pilot deployment (open gate — not yet obtained)

### Technical Constraints

**Onboard Edge**
- Raw video processing: Hailo-8 M.2 (26 TOPS) — always-on, handles detection/tracking/occupancy
- LLM inference: Hailo-10H M.2 (40 TOPS INT4) — bursty, handles safety queries and briefing generation
- Target inference latency: sub-5 seconds for conductor safety queries (to be confirmed under load test — spike gate before Conductor App LLM stories)
- No internet dependency for any onboard inference — operates entirely over train L2 network (VLANs 2,3,5,6,7,8,12)
- Onboard storage: SQLite event store — single-writer authority, sync-then-truncate cursor pattern, keep last 3 journeys as debug buffer
- Docker restart policies mandatory — all onboard containers must self-recover without operator intervention

**Data Transport**
- IntentPacket: MQTT QoS 1, retain=True — `oebb/{train_id}/intent` topic
- `retain=True` ensures reconnecting consumers receive current state immediately on subscribe
- Conductor App: IndexedDB cache via Workbox service worker — offline-capable, expired packets muted not deleted
- Landside sync: REST + SSE from cloud-backend; `packet_id` used as idempotency key on insert

**Landside**
- PostgreSQL for all operational state — no React component state or Hermes conversation memory for acknowledged alerts or status changes
- pgvector extension required — `CREATE EXTENSION vector` needs superuser or `pg_extension_owner` role; migration must include `IF NOT EXISTS` guard
- **pgvector P99 latency SLA: <500ms** — if exceeded, fallback to keyword ranking (`search_fleet_events` full-text), logged as WARN; UI labels results "Keyword search active — semantic search unavailable". No silent degradation. This SLA is a formal acceptance criterion for the pgvector spike story.
- Alembic migrations must be safe under concurrent reads — no table locks during pilot operations
- All API responses snake_case JSON; React converts to camelCase at API client layer only

**Security**
- BYOK API keys stored in environment variables (PoC) or Docker secrets (fleet) — never in source control
- All VLAN poller and Hailo ingest credentials via resource initialisation at container startup — never inline in inference code
- All external data (Hailo results, VLAN poller outputs, MQTT payloads) treated as untrusted — schema validation before any AI decision path
- LLM output sanitisation layer required between Hermes response and Fleet Manager UI render

### Integration Requirements

| System | Integration type | Status | Notes |
|---|---|---|---|
| ÖBB SSO (IdP) | Identity provider for ECM sign-off `technician_id` binding | Required for ECM compliance | Must be IdP-bound, not local session |
| MQTT broker | IntentPacket transport, QoS 1, retain=True | Spec frozen | `oebb/{train_id}/intent` |
| Hailo-8 (vision) | Local M.2 inference — detection, tracking, occupancy | Phase 1 built | Always-on |
| Hailo-10H (LLM) | Local M.2 inference — safety queries, briefings | Phase 2 greenfield | Bursty; sub-5s target pending spike |
| Integration with Stadler Diagnostics systems for alarms. 
| VLAN sources | CCTV (5), APC (8), SNMP (7), PIS/FIS (3), Reservations (6), Energy (12), ZFR (2) | Phase 1 pollers built | ZFR read-only |
| Rail MCP Server | HTTP MCP endpoint — 9 tools for Hermes | Spec frozen | New subpackage `rail-mcp-server/` |
| Hermes (BYOK) | LLM orchestration agent runtime | Designed, not built | Nous Research, MIT licence |
| Frontend wrapper for Hermes for the Fleet manager/dispatcher and anyone using the landside portal
| BOOM (work orders) | Work order integration for depot | **Descoped for PoC** | Structured export/copy for manual entry |
| pgvector | Semantic search over `incident_summary` embeddings | Requires migration spike + P99 SLA confirmation | Fallback: keyword ranking |
| ÖBB HAFAS | Timetable data via `get_hafas_timetable` Rail MCP tool | Required for Fleet Manager | 48h horizon |

### Risk Mitigations

| Risk | Mitigation |
|---|---|
| Safety sign-off not obtained before pilot | Gate: written acknowledgement from ÖBB safety authority required before Conductor App advisory mode is enabled. PoC runs in confirmatory-only mode without sign-off. |
| BYOK prompt injection | Output sanitisation layer between Hermes and UI render — validated schema, reject unexpected fields, log anomalies |
| pgvector migration blocks reads | Migration with `IF NOT EXISTS` guard; confirm PostgreSQL user has extension creation rights; rollback: `DROP EXTENSION vector CASCADE` drops all vector columns — document data loss risk |
| ECM sign-off record loss (connectivity gap) | Queue locally at depot, sync on reconnect; extended loss flagged in audit as gap with last-known timestamp |
| Hailo-10H load test not run before sprint | Spike gate: sub-5s latency confirmed before any Conductor App LLM story goes to dev |
| BOOM API unknown | Scoped as "BOOM-ready" not "BOOM-integrated" — structured export only until API contract confirmed with ÖBB IT |
| NIS2 incident reporting obligation | Security incident response runbook required before fleet rollout; not a PoC blocker but must be documented |
| BYOK data residency not confirmed | BYOK keys and vehicle state snapshots must be designated to EU GDPR processors. **Blocks procurement sign-off for multi-tenant rollout.** PoC unaffected — single ÖBB instance, on-premise SYS2. Must be resolved before commercial expansion. |
| pgvector latency exceeds SLA | Fallback to keyword ranking (WARN logged, UI labelled). Spike story must confirm or revise <500ms P99 target before Fleet Manager stories begin. |

### UX Constraints (Domain-Driven)

**ECM Sign-Off Confirmation Window**
The 5-second confirmation window before `ecm_signoff_events` record commits is irreversible by design (append-only). The UX must account for this explicitly:
- Visible countdown timer displayed during confirmation window
- High-contrast confirmation state — distinct from normal UI chrome
- Post-sign summary view: shows exactly what was committed before window closes
- Amendment path (`supersedes_id` correction flow) surfaced immediately after sign-off — not buried in settings
- Rationale: ECM Managers working under time pressure (pre-arrival brief, 14-minute window) must be able to verify what they signed without hunting for it. Regret in a safety-critical sign-off destroys trust faster than any feature wins it back.

**pgvector Fallback Transparency**
- Fleet Manager always knows which search mode is active — semantic or keyword
- UI label "Keyword search active" shown whenever fallback is in effect
- No silent degradation permitted

---

## 6. Project-Type Specific Requirements

### Project-Type Overview

This platform is a **hybrid saas_b2b / iot_embedded** product. The onboard stack is an edge-deployed IoT system (Hailo-8 + Hailo-10H, Docker, SQLite, MQTT) with hard connectivity and inference constraints. The landside stack is a B2B SaaS surface (Fleet Manager AI Interface, PostgreSQL, Hermes BYOK) with enterprise permission model and regulatory compliance requirements. The Depot Briefing Interface bridges both — mobile PWA operating in a fixed-network depot environment with ECM regulatory obligations.

---

### Technical Architecture Considerations

**Multi-tenancy model**
PoC: single ÖBB tenant, on-premise SYS2. No multi-tenancy in PoC. Fleet rollout: one instance per operator (ÖBB, Westbahn, etc.) — shared codebase, separate deployments. Shared-database multi-tenancy explicitly deferred — BYOK data residency and ECM audit trail integrity require tenant isolation before any multi-tenant architecture is considered.

**Permission model**
Five roles, mapped to platform surfaces:

| Role | Surface | Permissions |
|---|---|---|
| Conductor (Zugbegleiter) | Conductor App | Read own train IntentPackets; dismiss alerts; query Hailo-10H |
| Dispatcher (Fahrdienstleiter) | Fleet Manager | Read-only relay card; forward to conductor |
| Fleet Manager (Disponent) | Fleet Manager | Full triage; redeployment decisions; dismissal signal arbitration; shift handover |
| ECM Manager (ECM 1) | Depot Briefing | Sign-off authority; audit trail read; blocking state control |
| Technician (ECM 4) | Depot Briefing | Role-filtered fault feed; ACK faults in own discipline only |

All identity via ÖBB SSO IdP — no local user table for safety/compliance actions.

**Subscription / licensing**
Not applicable for PoC. Fleet rollout: per-operator licence. BYOK model means Hermes API costs are operator-owned — not in platform COGS. **Pre-fleet gate:** BYOK cost simulator required before fleet sales conversations — operators budget in Q+1, per-inference cost surprises kill procurement sign-off.

**SSO per-operator**
ÖBB SSO works for PoC. Fleet rollout: each operator brings their own IdP. Per-operator SSO customisation will be requested — budget 2–3 week integration tax per operator or build a standardised SAML bridge. This is a fleet-rollout architecture decision, not a PoC concern.

---

### IoT / Edge Requirements

**Hardware constraints**
- Hailo-8: always-on, 26 TOPS, vision pipeline — cannot be interrupted for LLM workloads
- Hailo-10H: bursty, 40 TOPS INT4, LLM inference — sub-5s P99 target under concurrent vision load (spike gate before Conductor App LLM stories)
- Thermal envelope: R5001C SYS2 rack mount — simultaneous dual-chip load profile must be confirmed under load test; no battery constraint
- Storage: SQLite onboard, keep last 3 journeys as debug buffer, sync-then-truncate cursor

**Hailo-10H priority arbitration**
When inference queue backs up, safety queries from the Conductor App take precedence over briefing generation (Depot Briefing). Head-of-line blocking for safety-critical inference is not acceptable. Priority rules:
1. Conductor safety query — highest priority, preempts queued briefing generation
2. Depot briefing generation — standard priority, queued
3. Background index updates — lowest priority, deferrable

Queue behaviour on sustained backup: Conductor App surfaces "Inference delayed — response pending" label (not silence, not error). Briefing generation falls back to last cached brief with staleness label.

**Connectivity protocol**
- Onboard: MQTT QoS 1, retain=True, `oebb/{train_id}/intent` — no internet dependency
- Edge → landside: REST + SSE from cloud-backend, `packet_id` idempotency key
- Conductor App: IndexedDB + Workbox service worker — offline-capable PWA

**State sync under partition**
MQTT QoS 1 + sync-then-truncate requires explicit partition handling. If edge loses MQTT connectivity mid-sync, truncation must not occur before landside receipt is confirmed. Required pattern:
- Write-ahead log (WAL) before any truncation — record `last_synced_packet_id` durably
- Truncate only after confirmed ACK from landside sync endpoint
- On MQTT reconnect: replay from `last_synced_packet_id` — no gap, no duplicate (idempotent insert via `packet_id`)
- Extended partition (>72h): flagged in audit as gap with last-known timestamp; no silent data loss

**OTA / update strategy**
- Edge containers updated via Docker pull during depot dwell (fixed network)
- IntentPacket schema frozen at v1.0.0 for PoC
- Post-PoC schema evolution: requires explicit architecture decision — options are (a) one-shot depot-dwell deploys with no rollback window, (b) schema version field with forward/backward adapters, or (c) hot-standby edge for A/B rollover. Decision deferred; schema freeze holds until this is resolved.
- Rolling deploy during PoC: same schema version on all nodes — no mixed-version edge+landside in PoC

**Security model**
- VLAN isolation enforced — credentials never pass through inference code paths
- All external data (Hailo results, VLAN outputs, MQTT payloads) schema-validated before AI decision path
- BYOK keys: env vars (PoC), Docker secrets (fleet) — never in source control

---

### Integration List

| System | Integration type | PoC status | Fleet gate |
|---|---|---|---|
| ÖBB SSO (IdP) | Identity for ECM sign-off + role-based access | Required for ECM compliance | Per-operator SAML bridge — architecture decision pre-fleet | Consider using Keycloak
| MQTT broker | IntentPacket transport, QoS 1, retain=True | Onboard only | **Blocking pre-fleet:** broker identity, failover, and multi-train topic isolation must be specified |
| Docker registry | Edge OTA — container pull during depot dwell | PoC: manual or shared | **Blocking pre-fleet:** private registry, credential rotation ownership, and access model must be in contract template |
| Rail MCP Server | HTTP MCP endpoint — 9 tools for Hermes | Spec frozen, new subpackage | Required for Fleet Manager |
| Hermes (BYOK) | LLM orchestration, operator-owned API keys | Designed, not built | BYOK cost simulator required before fleet sales |
| BOOM (work orders) | Depot work order integration | Descoped — structured JSON export only | API contract required from ÖBB IT before integration stories |
| pgvector | Semantic search over incident_summary embeddings | Spike + P99 confirmation required | Fallback: keyword ranking |
| Stadler Diagnostic and IM system on train
| ÖBB HAFAS | Timetable data via Rail MCP tool | Required for Fleet Manager | 48h horizon | - Require a spike after integration with Stadler Diagnostic and IM system integration is done, to see if all required info  can be retrieved onboard.
| Hailo-8 (vision) | Local M.2 — detection, tracking, occupancy | Phase 1 built | Always-on |
| Hailo-10H (LLM) | Local M.2 — safety queries, briefings | Phase 2 greenfield | Sub-5s spike gate |

---

## 7. Innovation Focus

### What Is Novel Here

This platform is not an analytics dashboard with AI bolted on. It is a structured evidence infrastructure built around a single architecture invariant: every safety-relevant observation, recommendation, and human decision is captured as a versioned, immutable event — onboard, in depot, and landside.

Five claims. Each is testable. None is marketing.

---

**Claim 1 — Offline LLM inference eliminates cloud dependency for safety-adjacent decisions**

Every conductor safety query is answered by Hailo-10H running on the train. No internet, no cloud round-trip, no latency cliff when the train enters a tunnel or a dead zone. The inference chain is: Hailo-8 detects → IntentPacket emitted → Hailo-10H answers → conductor sees recommendation. All within the onboard L2 network.

Commercial implication: Roland can approve budget on this single claim. Cloud-dependent competitors lose coverage on ~15% of rail journeys. This product does not.

*Validation: sub-5s inference P99 under concurrent vision workload — spike gate before Conductor App LLM stories.*

---

**Claim 2 — Trust-curve onboarding earns conductor adoption without change management overhead**

The first 10 shifts run in confirmatory mode only — the platform shows conductors things they can already verify (occupancy they can see, doors they can check). No advisory, no recommendations. The conductor learns to trust the system by watching it be right about things they know.

Advisory mode is unlocked per conductor per alert class after demonstrated accuracy. Trust is earned, not assumed.

Commercial implication: no existing digital tools in the Zugbegleiter role means no workflow replacement friction — clean union conversation, clean adoption story.

*Validation: per-conductor trust-curve state machine logged; advisory unlock event recorded with alert class and accuracy history.*

---

**Claim 3 — Human-in-the-loop model calibration via Fleet Manager dismissal review**

Conductor alert dismissals are logged as candidate feedback signals — no friction, no confirmation gate onboard. Landside, the Fleet Manager sees a queue of all dismissals: conductor ID, alert class, timestamp, operational context. They decide which dismissals represent genuine signal (conductor consistently dismisses low-severity door alerts on a specific route segment) and which are noise (dismissal during a known high-activity boarding period).

Promoted dismissals feed the alert threshold tuning loop. The Fleet Manager is the signal arbiter — they have the operational context to know the difference.

*Validation: dismissal queue visible in Fleet Manager triage view; promotion/discard action logged; threshold adjustment traceable to Fleet Manager decision.*

---

**Claim 4 — Regulatory compliance as a primary feature, not an afterthought**

The ECM audit trail is not a log — it is the product. Every sign-off is an append-only record with IdP-bound identity, vehicle state snapshot at moment of sign-off, EU 2019/779 directive reference, and HMAC tamper evidence. Corrections are new rows with `supersedes_id` — the original is never touched.

This is the evidence chain Dr. Fischer needs for a regulatory inspection. It is also the evidence chain that makes the platform a reference customer for ECM compliance digitisation.

*Validation: ECM audit export matches internal ÖBB documentation requirements — confirmed by ECM-accountable stakeholder (Dr. Fischer / equivalent) during pilot week 3.*

---

**Claim 5 — Safe schema evolution across edge and cloud during live operation**

IntentPacket v1.0.0 is frozen. When v1.1.0 ships, edge containers (onboard) and landside containers may be running different versions simultaneously during a rolling deploy. The platform must handle this without dropping safety-critical alert types or requiring a maintenance window.

The real innovation is not versioning — it is rollback semantics for safety-critical types. If a new alert class introduced in v1.1.0 cannot be interpreted by a v1.0.0 consumer, the platform must degrade gracefully: surface the raw event, not silently discard it.

*Validation: v1.x compatibility test harness before any IntentPacket schema change ships to production edge hardware.*

---

### Market Context

| Competitor approach | This platform |
|---|---|
| Cloud-dependent AI inference | Fully onboard — no internet dependency |
| Generic analytics dashboards | Role-differentiated interfaces per ÖBB accountability model |
| Manual ECM documentation | Append-only digital audit trail under EU 2019/779 |
| Flat alert feeds | Trust-curve onboarding + Fleet Manager signal arbitration |
| Schema versioning as an IT concern | Schema evolution with rollback semantics for safety-critical types |

---

### Validation Approach

| Claim | How validated | Gate |
|---|---|---|
| Offline inference | Hailo-10H load spike — sub-5s P99 under concurrent vision | Pre-Conductor App LLM stories |
| Trust-curve onboarding | Per-conductor advisory unlock event log — accuracy history visible | First 10-shift pilot cohort |
| Dismissal signal loop | Fleet Manager dismissal queue + promotion audit log | Post-pilot analysis |
| ECM compliance | Dr. Fischer (or equivalent) sign-off that export maps to internal requirements | Pilot week 3 |
| Schema evolution | v1.x compatibility test harness — no dropped alert types on mixed-version deploy | Before any schema change ships |

---

### Innovation Risks

| Risk | Mitigation |
|---|---|
| Trust-curve design language has no existing templates | Flag as primary UX design risk — Freya to prototype confirmatory → advisory transition states before stories begin |
| Dismissal signal arbitration adds Fleet Manager workload | Queue must be low-noise by design — only dismissals with pattern (≥3 in same alert class, same route segment) surface for review. Single dismissals silently discarded. |
| Schema evolution complexity underestimated | Freeze IntentPacket at v1.0.0 for PoC. Schema evolution spec is a post-PoC architecture decision. |
| Offline inference SLA not confirmed | Spike gate is hard — no Conductor App LLM stories until sub-5s P99 confirmed under load. |

---

## 8. Project Scoping & Feature Prioritisation

### Strategy & Philosophy

**Approach:** Compliance-first platform PoC — single release covering all three interfaces. The minimum that gives ÖBB a defensible ECM audit trail, one validated conductor safety query loop, and one Fleet Manager triage + redeployment decision. Commercial pilot narrative leads with ECM proof point (Dr. Fischer sign-off), Conductor second, Hermes third — deal velocity, not technical sequence.

**Resource Requirements:** One full-stack dev (React + Python/FastAPI), one AI/ML engineer (Hailo-10H, Hermes integration), one DevOps/infra (Docker, MQTT, PostgreSQL). ÖBB safety authority sign-off required before Conductor App advisory mode goes live — gate, not risk.

---

### Complete Feature Set

**Core User Journeys:** All 10 journeys in scope (conductor safety query, ECM pre-arrival brief, Fleet Manager redeployment, shift handover, stale packet degradation, infrastructure partition, ECM commercial confidence, novel fault, ROI conversation, damaged trust recovery).

**Must-Have Capabilities:**

| Capability | Rationale |
|---|---|
| IntentPacket working schema (MQTT pipeline, Hailo-8 → landside) | Foundation — nothing works without this; v1.0.0 contract formalisation is post-PoC |
| Conductor App — confirmatory mode, alert feed, stale packet muting | Trust-curve onboarding; advisory locked behind safety sign-off gate |
| Conductor App — departure clearance checklist | Procedural safety flow; highest-friction conductor workflow |
| Conductor App — shift-aware density reduction (7h threshold) | Disponent KPI driver; drop only as last resort |
| Depot Briefing — ECM Manager pre-arrival brief | Primary commercial proof point for Dr. Fischer |
| Depot Briefing — ECM 1 blocking state + append-only audit trail | Regulatory compliance feature; closes the compliance sale |
| Depot Briefing — role-filtered technician feed (electrical/mechanical/IT) | Operational value; required for depot pilot |
| Fleet Manager — triage + redeployment + cost-impact one-liner | Disponent KPI alignment; prevented delay-minutes counterfactual |
| Fleet Manager — Hermes BYOK + Rail MCP Server (9 tools) | Natural language query layer; differentiates from dashboard competitors |
| Fleet Manager — shift handover flow | Required for multi-shift pilot |
| pgvector semantic search + keyword fallback | Spike gate: P99 <500ms under realistic incident volume confirmed before stories |
| ECM audit trail export (JSON) | Dr. Fischer's sign-off requirement |
| WAL-before-truncate sync pattern | Data integrity — no silent loss under partition |
| Hailo-10H priority arbitration | Safety query preempts briefing generation |
| Hailo-8 graceful degrade | Vision-dark fallback: last-known state held, conductor notified, no silent failure |

**Nice-to-Have Capabilities (in scope, lower priority):**

| Capability | Notes |
|---|---|
| Fleet Manager — 48–72h forward planning view | Drop first if resource-constrained — forward planning is nice-for-nice |
| Structured JSON export formatting refinements | BOOM-ready but not BOOM-integrated |
| Fleet Manager — dismissal signal arbitration queue | Requires spike (decision tree, queue persistence, failure modes) before buildable; deferred to post-PoC if spike unresolved |

**Spike Gates (must resolve before dependent stories begin):**

| Spike | Gate | Outcome if missed |
|---|---|---|
| Hailo-10H load test — sub-5s P99 under concurrent vision | Before Conductor App LLM stories | Fallback to heuristic; advisory mode deferred |
| pgvector P99 <500ms under realistic incident volume | Before Fleet Manager search stories | Keyword fallback only; semantic search deferred |
| Fleet Manager dismissal signal arbitration | Before dismissal queue stories — spike must define decision tree, queue persistence, failure modes | Feature deferred to post-PoC |
| HAFAS integration required or can we retrieve all the info from the onboard Stadler IM
---

### Risk Mitigation Strategy

**Technical risks:**
- Hailo-10H sub-5s P99 under concurrent vision load — hard spike gate; fallback to heuristic if missed
- pgvector P99 <500ms under realistic incident volume — spike story with keyword fallback confirmed before Fleet Manager search stories
- WAL-before-truncate — explicit acceptance criterion on edge sync stories; no truncate without confirmed ACK
- Hailo-8 vision failure — graceful degrade to last-known state; conductor notified; no silent failure

**Market risks:**
- Safety authority sign-off not obtained — Conductor App runs confirmatory-only without it; PoC value demonstrated via ECM and Fleet Manager proof points
- Dr. Fischer unavailable at pilot week 3 — audit trail still ships; sign-off milestone deferred

**Resource risks:**
- If team is constrained: drop 48–72h forward planning view first; all compliance, triage, and conductor capabilities protected
- Dismissal signal arbitration moves to post-PoC if spike is unresolved — not a PoC blocker

---

## 9. Open Architecture Decisions

The following decisions are deferred to the architecture phase. They are not PoC blockers but must be resolved before the first story in their respective epic begins. The architect owns these inputs.

| Decision | Context | Gate |
|---|---|---|
| Onboard edge RTO | What is the recovery time objective if SYS2 hardware fails mid-service? Disaster Recovery covers software restart (<30s Docker), not hardware replacement. Architecture must define the hardware failure posture. | Before any onboard edge story |
| Hermes intent disambiguation rules | FR31 requires the query system to identify user intent and select the appropriate Rail MCP tool. The disambiguation logic (rules-based, embedding-based, or LLM-classified) is an architecture decision — the PRD specifies the outcome, not the mechanism. | Before Fleet Manager Hermes stories |
| MQTT broker identity and failover | Broker host, auth model, and multi-train topic isolation strategy for fleet deployment. Flagged in §6 as blocking pre-fleet. Architecture must define the broker topology and failover behaviour before multi-train stories begin. | Pre-fleet gate; confirm identity for PoC before first MQTT story |
| Schema evolution strategy | IntentPacket frozen at v1.0.0 for PoC. Post-PoC options: one-shot depot-dwell deploys, schema version field with forward/backward adapters, or hot-standby edge for A/B rollover. Architecture decision required before any schema change ships. | Post-PoC; must not block PoC |
| Conductor App native wrapper | PWA via onboard AP for PoC. If native device features (persistent background sync, native push, haptic APIs) are required post-pilot, Capacitor wrapper evaluated. Architecture decision pending pilot feedback. | Post-pilot |
| Fleet Manager AI interface App frontend wrapper reequired over Hermes agent backend.

---

## 10. Non-Functional Requirements

### Performance

| Requirement | Target | Context |
|---|---|---|
| Conductor LLM safety query response | P99 < 5s | Local Hailo-10H inference only — no cloud dependency; connectivity gaps handled by graceful degrade (FR5/FR40), not by relaxing this target. Spike gate before Conductor App LLM stories. |
| pgvector semantic search | P99 < 500ms | Under realistic incident volume — spike gate before Fleet Manager search stories |
| IntentPacket MQTT publish-to-consume latency | < 2s end-to-end | Onboard L2 network; excludes landside sync |
| Fleet Manager triage feed load | < 3s initial render | PostgreSQL query under PoC fleet size (~10–20 vehicles) |
| ECM sign-off commit | < 1s write acknowledgement | Landside PostgreSQL direct write; not through edge event store |
| Conductor App first meaningful paint | < 2s cold start | IndexedDB cache warm; Workbox service worker active |
| Rail MCP Server tool call timeout | 8s hard timeout | Hermes surfaces structured error; Fleet Manager decision logged with data-availability flag |

### Reliability

| Requirement | Target | Context |
|---|---|---|
| Onboard container restart recovery | Automatic, < 30s | Docker restart policies mandatory; all onboard services self-recover without operator intervention |
| Edge event data durability | Zero loss under MQTT partition | Outcome requirement — implementation pattern (WAL-before-truncate) specified in architecture doc |
| ECM audit trail integrity | Append-only, tamper-evident | No UPDATE or DELETE permitted; HMAC on every row; row-level trigger raises exception on modification attempt |
| Conductor App offline operation | Full read access to last cached IntentPacket | IndexedDB via Workbox; expired packets muted not deleted |
| Landside event delivery | No duplicate delivery after service restart | Outcome requirement — idempotency implementation specified in architecture doc |
| Alembic migration write-lock ceiling | < 30s | Any migration exceeding this threshold requires an offline maintenance window; rollback path documented before merge |

### Security

| Requirement | Target | Context |
|---|---|---|
| ECM sign-off identity | IdP-bound only | ÖBB SSO — no local session token accepted for compliance actions |
| BYOK API key storage | Never in source control, logs, or inference paths | Env vars (PoC), Docker secrets (fleet) |
| LLM output sanitisation | Validated schema before UI render | Between Hermes response and Fleet Manager render layer — prompt injection mitigation |
| VLAN credential isolation | Credentials never in inference code paths | Resource initialisation at container startup only |
| External data validation | Schema validation before any AI decision path | All Hailo results, VLAN outputs, MQTT payloads treated as untrusted |
| Raw video containment | Never leaves the train | Hailo-8 processes onboard; only anonymised event metadata transmitted landside |

### Scalability

| Requirement | Target | Context |
|---|---|---|
| PoC fleet size | 1–5 trains, 10–20 vehicles | Single ÖBB tenant, on-premise SYS2 |
| Fleet rollout architecture | Per-operator deployment | Separate instances per operator — no shared-database multi-tenancy in initial fleet architecture |
| Hermes API quota | Operator-owned | BYOK model — operator owns API quota; platform does not throttle |
| PostgreSQL read concurrency | Migrations safe under concurrent reads | No table locks on reads during pilot operations |

### Accessibility

| Requirement | Target | Context |
|---|---|---|
| Conductor App tap targets | Minimum 64dp | One-handed operation while moving through coaches; conductor bracing against coach sway demands larger targets than stationary-use 56dp baseline |
| Contrast ratio | WCAG AA minimum; tested under night shift and glare conditions | Shift-aware density mode increases contrast ratios after threshold; full-screen alerts always maximum contrast |
| Severity-1 alert takeover | Full-screen, layered with explicit dismiss affordance | Must not suspend keyboard nav or screen reader flow — takeover layers over UI, not replaces it |
| Critical action step ceiling | Maximum 2 deliberate steps (open flow + confirm) | Emergency and safety-adjacent flows: open + confirm only. Standard flows: maximum 3 steps (FR43) |
| Haptic feedback | Required for all confirmation events on Conductor App | Train vibration masks visual-only cues; haptic pulse on hold-to-confirm and checklist completion |
| Audio alerts | Depot PWA audio alert spec required | Depot ambient noise floors polite notification sounds; alert audio must meet minimum dB threshold above depot floor |

### Infrastructure Readiness (Pre-Pilot Gate)

The following must be confirmed before pilot go-live. Martin Lerch owns sign-off.

| Gate | Requirement | Status |
|---|---|---|
| SYS2 thermal envelope | Dual-chip concurrent load (Hailo-8 always-on + Hailo-10H bursty) must not exceed thermal limits under sustained operation — confirmed via load test | **Open — spike required** |
| Concurrent VLAN headroom | SYS2 must sustain simultaneous polling of VLAN 5 (CCTV), VLAN 7 (SNMP), VLAN 8 (APC), VLAN 3 (PIS/FIS) without packet loss during peak inference | **Open — to be measured** |
| Integration with Stadler IM and retrieving all required information.
| MQTT broker identity | Broker host, auth model, and failover behaviour confirmed for multi-train deployment | **Open — pre-fleet gate** |
| Docker registry | Private registry access confirmed; credential rotation ownership agreed | **Open — pre-fleet gate** |

### Maintainability

| Requirement | Target | Context |
|---|---|---|
| IntentPacket schema | Frozen at v1.0.0 for PoC | Schema evolution spec deferred post-PoC; no mixed-version edge+landside in PoC |
| Edge container updates | Via Docker pull during depot dwell | Fixed network at depot; no OTA over mobile |
| Alembic migrations | Reversible, rollback path documented before merge | See reliability: write-lock ceiling 30s; offline window required if exceeded |

---

## 10. Functional Requirements

### Onboard Alert & Safety Query

- FR1: Conductor can view the current active alert feed for their train, filtered to their journey context
- FR2: Conductor can query the onboard LLM with a natural language safety question and receive a structured response
- FR3: Conductor can dismiss an alert, with the dismissal logged as a candidate feedback signal
- FR4: Conductor can view a stale-packet indicator when the most recent IntentPacket has expired, showing time since last update
- FR5: Conductor can see the system's last-known state when onboard inference is delayed or unavailable, with no silent failure
- FR6: System can mute (not delete) an expired IntentPacket alert and surface a staleness label with elapsed time
- FR7: System can degrade gracefully when Hailo-8 vision is unavailable, holding last-known state and notifying the conductor

### Conductor Departure & Shift Management

- FR8: Conductor can complete a structured departure clearance checklist before train departure, with auto-populated steps where sensor data is available
- FR9: Conductor can confirm the final departure clearance step via a deliberate hold-to-confirm gesture, active only when all checklist steps are complete
- FR10: System can adapt UI density automatically after a configurable shift duration threshold, reducing ambient data without conductor action

### Depot Briefing & ECM Compliance

- FR11: ECM Manager can view a pre-arrival briefing for an inbound vehicle, generated from the vehicle's IntentPacket history
- FR12: ECM Manager can see all open faults on a vehicle, organised by technician discipline (electrical, mechanical, IT)
- FR13: ECM Manager can view a vehicle's blocking state when all technician faults are resolved but ECM 1 authorisation is pending
- FR14: ECM Manager can review a full fault resolution summary and sign off on a vehicle's return to service via a deliberate confirmation action
- FR15: ECM Manager can view a post-sign summary of exactly what was committed immediately after sign-off
- FR16: ECM Manager can correct a sign-off record by creating a superseding record, without modifying or deleting the original
- FR17: ECM Manager can export the full audit trail for a vehicle as structured JSON
- FR18: Technician can view a role-filtered fault feed showing only faults within their discipline
- FR19: Technician can acknowledge a fault with a structured ACK record
- FR20: System can maintain an append-only sign-off event log with IdP-bound identity, vehicle state snapshot, directive reference, and tamper-evidence signature

### Fleet Manager Triage & Redeployment

- FR21: Fleet Manager can view a real-time triage feed of active fleet disruptions, with cost-impact one-liner per disruption
- FR22: Fleet Manager can assign a replacement vehicle to a disrupted service
- FR23: Fleet Manager can view a staleness indicator on any fleet card where IntentPacket data is older than a threshold
- FR24: Fleet Manager can hand over their shift to an incoming Fleet Manager, with a snapshot of all open incidents and optional notes
- FR25: Fleet Manager can view an incoming shift handover summary at the top of the triage view on login
- FR26: Fleet Manager can review the conductor dismissal queue and promote or discard individual dismissals as alert threshold tuning signals

### Fleet Manager AI Query (Hermes)

- FR27: Fleet Manager can query the fleet via natural language and receive a structured response grounded in live fleet data
- FR28: Fleet Manager can query predicted maintenance windows and fleet availability over a 48–72h horizon
- FR29: Fleet Manager can search past incidents using semantic similarity and receive ranked results
- FR30: Fleet Manager can see which search mode is active (semantic or keyword fallback) for any query result
- FR31: Query system identifies user intent and selects the appropriate data source to ground the response
- FR32: System can surface a structured error when a Rail MCP Server tool call times out, with the Fleet Manager's decision logged alongside the data-availability flag

### IntentPacket & Event Pipeline

- FR33: System can generate an IntentPacket from Hailo-8 vision output and publish it via MQTT to the correct train topic
- FR34: System can ingest an IntentPacket landside with idempotent insert using packet_id as the deduplication key
- FR35: System can sync edge events to landside using a WAL-before-truncate pattern, ensuring no data loss under MQTT partition
- FR36: System can replay edge events from last confirmed sync point on MQTT reconnect

### Identity, Access & Audit

- FR37: All safety-critical and compliance actions can be bound to an ÖBB SSO IdP identity, not a local session
- FR38: Each platform surface can enforce role-based access so users see only the capabilities and data relevant to their role
- FR39: System can record every acknowledgement, sign-off, dismissal, and status change as a server-side write to PostgreSQL, not component state

### Resilience & Consistency

- FR40: System provides a defined fallback state when the vision pipeline is unavailable — last-known values displayed with explicit unavailability label; interface remains navigable
- FR41: System maintains write consistency for cross-boundary operations — partial failures are logged and surfaced for reconciliation, never silently dropped
- FR42: Fleet Manager can view delay-impact attribution for redeployment decisions — actual vs. projected delay-minutes traceable to the Fleet Manager action
- FR43: Conductor-facing confirmation flows complete within a defined maximum interaction count — no safety-adjacent action requires more than 3 deliberate steps

---

*This is the binding capability contract — 43 FRs across 8 capability areas. Any feature not listed here will not exist in the final product unless explicitly added.*
