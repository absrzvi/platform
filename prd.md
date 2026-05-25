---
stepsCompleted: [step-01-init, step-02-discovery, step-02b-vision, step-02c-executive-summary, step-03-success, step-03-dr, step-04-journeys, step-04-journeys-p2, step-05-domain, step-06-innovation, step-07-project-type, step-08-scoping, step-09-functional, step-10-nonfunctional, step-11-polish]
releaseMode: single-release
inputDocuments:
  - platform/_archive/party-mode-critical-assessment-2026-05-23.md
  - platform/oebb-role-research-2026-05-23.md
  - platform/ux-decisions-2026-05-23.md
  - platform/ux-decisions-update-1-2026-05-23.md
  - platform/rail-mcp-server-spec-2026-05-23.md
  - platform/ÖBB Telematics Project Management UX - Google Gemini.pdf
  - platform/intent-packet-schema-v1.0.0.md (reference — schema owned by oebb-brain)
workflowType: 'prd'
splitFrom: prd-unified-2026-05-23 (see git history for combined version)
classification:
  projectType: Landside fleet operations & telematics platform (web dashboard for situational awareness and planning; AI agent interface for fleet analytics; mobile depot briefing PWA for ECM compliance)
  domain: Rail operations — regulated landside (EU ECM Directive 2019/779, NIS2, GDPR). Safety-adjacent advisory only; no automated control commands.
  complexity: High — multi-system integration (HAFAS, vendor diagnostic portals) + Hermes BYOK agent runtime + pgvector semantic search + append-only ECM audit
  projectContext: Greenfield landside platform; receives IntentPacket stream from oebb-brain (onboard) and from vendor telemetry sources. SAP/ServiceNow/BOOM integrations are post-MVP.
  deploymentModel: Single ÖBB instance (PoC); per-operator deployment at fleet rollout
  aiPosture: Advisory only — Hermes suggests via structured Action Intent Objects, human approves via one-click interlock, platform records to append-only audit trail
---

# Product Requirements Document — ÖBB Landside Platform

**Author:** AbbasRizvi
**Date:** 2026-05-23 (split into landside-only on 2026-05-25)
**Companion document:** `oebb-brain/prd.md` — onboard brain PRD (Conductor App, Hailo inference, IntentPacket schema)

---

## Scope Boundary

This PRD covers the **landside platform** — the web and mobile surfaces used by ÖBB's fleet operations, technical services, and telematics project management teams. The onboard "brain" (Hailo-10H inference, Conductor App PWA, IntentPacket schema, edge resilience) is a separate product documented in `oebb-brain/prd.md`.

The landside platform **consumes** IntentPackets from the brain over MQTT and persists them. The schema is owned by the brain project. Landside is a downstream subscriber, alongside HAFAS timetable data and Stadler vendor portal feeds. SAP PM, ServiceNow, and BOOM are post-MVP integrations.

---

## Executive Summary

ÖBB's landside operations — Disponent (Fleet Manager), ECM Manager (regulatory authorisation), Technicians (ECM 4), and Teil-Projektleitung Telematik (Telematics Sub-Project Manager) — currently make high-stakes decisions across fragmented systems: vendor diagnostic portals, Power BI dashboards, ServiceNow tickets, vendor work orders, Microsoft Project schedules, and Excel reconciliation files. Each switch costs cognitive load; each gap between systems is a place a fault goes unattributed or a compliance record goes missing.

The platform unifies these surfaces around three roles:

**Live decision support — Fleet Manager** *(primary persona — Roland Ruisz, ÖBB operations)* — the Disponent receives a real-time triage feed of fleet disruptions, each tagged with a cost-impact one-liner. Hermes (operator-owned BYOK LLM) bundles complex multi-system actions (HAFAS rotation update + work order creation) into structured Action Intent Objects rendered as one-click approval cards. The Disponent approves; the platform writes immutable audit log entries and dispatches programmatic commands to backend systems.

**Pre-arrival compliance — ECM Manager** — the ECM 1 Manager receives a structured fault brief generated from the vehicle's IntentPacket history before docking. They sign off with an IdP-bound identity, vehicle state snapshot, and EU 2019/779 directive reference written to an append-only ledger. Corrections create new rows referencing the original; the original is never touched.

**Commissioning & lifecycle — Teil-Projektleitung Telematik** *(primary persona — Roland Ruisz, ÖBB operations; same person as Fleet Manager)* — the Telematics Sub-Project Manager monitors trial runs (Probebetrieb) for newly delivered fleets, validates configuration alignment across the Stadler delivery programme (DOSTO Neu pilot, Cityjet Enzo on rollout expansion), monitors phased OTA software rollouts and configuration/firmware drift, and triages root-cause warranty disputes against vendors using end-to-end network path evidence. This persona was identified in the Gemini research pack (May 2026) and is the daily user of the landside platform's macro situational awareness zone.

Every decision is made by a human. The platform records it as immutable evidence.

### What Makes This Special

**Prevented delay-minutes counterfactual** — measurable, ÖBB-validated KPI the Disponent is already accountable for. The platform makes this number visible. Cloud-dependent competitors cannot produce this because they have no edge inference layer producing structured event evidence.

**ECM audit trail** — not a log, but the regulatory compliance record under EU 2019/779. Append-only, IdP-bound, HMAC-signed, with `supersedes_id` for corrections. Built to survive regulatory inspection.

**Hermes Action Intent Objects** — the landside differentiator. Where competitor dashboards show data and require humans to translate it into actions across 4–7 systems, this platform's Hermes co-pilot bundles multi-system actions into a single authorisation card. The human-in-the-loop interlock is preserved (SSO check + immutable log entry + dispatch) while reducing the per-decision system-switching cost.

**Vendor warranty leverage** — the platform's end-to-end network path visibility resolves "is it our landside or the manufacturer's hardware" disputes during the Probebetrieb commissioning phase, where a single telematics breakdown can freeze formal sign-off. This is a non-obvious commercial wedge for the Telematics Sub-Project Manager persona.

### Pilot Success Criteria

| Criterion | Target | Evidence method |
|---|---|---|
| ECM pre-arrival review | >80% of arrivals | IntentPacket generated time vs HAFAS scheduled arrival time vs ECM open time. **Pre-arrival trigger = HAFAS scheduled arrival (MVP default — IntentPacket schema v1.0.0 carries no GPS/position field). Brain workspace to confirm if a position field is planned for a future schema version.** |
| Disponent action latency reduction | Measurable reduction vs baseline | Instrument time-to-decision for redeployment calls; baseline early pilot mandays, measured later pilot mandays |
| Dispatcher coordination | Deferred — Dispatcher persona removed from MVP scope |
| Action Intent approval rate | >70% of Hermes-generated cards approved (vs modified or rejected) within first 30 pilot mandays | Approval action log; rejections feed prompt tuning |
| Rollout recommendation | Within 6 months of pilot start | Composite pilot health score updated weekly with visible threshold line |

### Project Classification

- **Project Type:** Landside fleet operations and telematics platform — web dashboard, mobile PWA, AI agent interface
- **Domain:** Rail operations — regulated landside (EU ECM Directive 2019/779, NIS2, GDPR)
- **Complexity:** High — multi-system integration + Hermes BYOK + pgvector + append-only audit
- **Project Context:** Greenfield landside; consumes IntentPackets from oebb-brain
- **Deployment Model:** Single ÖBB instance (PoC); per-operator deployment at fleet rollout
- **AI Posture:** Advisory only — Hermes suggests via structured Action Intent Objects, human approves via one-click interlock, platform records

---

## Success Criteria

### User Success

| Persona | Success moment | Measurable signal |
|---|---|---|
| Fleet Manager (Disponent) | Makes a redeployment decision with cost consequence visible before confirming | Time-to-decision for redeployment calls, measured before/after |
| Dispatcher (Fahrdienstleiter) | Post-MVP — persona deferred from MVP scope | — |
| ECM Manager | Arrives at depot having already reviewed the fault brief before docking | >80% of arrivals with IntentPacket opened before vehicle arrives |
| Technician (ECM 4) | Logs a resolution ACK that appears in Klaus's pre-arrival brief before the vehicle docks — no phone call, no paper handover | Fault reassignment rate per role per vehicle; % of faults with ACK logged before vehicle arrival (same signal as ECM pre-arrival review rate, from the technician side) |
| Teil-Projektleitung Telematik (Roland Ruisz) | Resolves a vendor warranty dispute using platform evidence in a single session | Time-from-dispute-raised to evidence-packaged; reduction in vendor escalation cycles |

### Business Success

- **Prevented delay-minutes:** Faults detected before service disruption, cross-referenced against historical delay data for same fault class on unmonitored units. Target: produce counterfactual evidence for at least 7 incidents in pilot period.
- **ECM audit trail completeness:** Sign-off timestamp, briefing-read timestamp, and fault resolution summary present for >95% of vehicle returns to service during pilot.
- **Fleet Manager decision latency reduction:** Measurable reduction in time-to-decision for redeployment calls compared to baseline (pre-platform). Baseline measured during early pilot mandays, post-platform measured in later pilot mandays (see Pilot calendar anchor).
- **Rollout recommendation:** Composite pilot health score crosses threshold within 6 months. Presented to Roland Ruisz, Martin Lerch, and ECM-accountable role (Head of Vehicle Maintenance or CSO).

### Technical Success

- pgvector semantic search P99 < 500ms under realistic incident volume — spike gate before Fleet Manager search stories
- ECM sign-off commit < 1s write acknowledgement; zero UPDATE/DELETE events on `ecm_signoff_events`
- Fleet Manager triage feed < 3s initial render under PoC fleet size (10–20 vehicles)
- Rail MCP Server tool call timeout 8s hard; structured error surfaced with data-availability flag
- IntentPacket landside ingest: idempotent insert on `packet_id`; no duplicate delivery after service restart

### Pilot-Kill Triggers

The following conditions pause the pilot pending investigation. Roland Ruisz (operations) and Martin Lerch (Technical Services) must be notified within 24h of any trigger firing.

| Trigger | Threshold | Action |
|---|---|---|
| ECM sign-off write failure | Unresolved >24h | Halt depot sign-off via platform; revert to paper process |
| pgvector semantic search unavailable | Keyword fallback active >48h continuous | Flag to Martin Lerch; infrastructure review required |
| Hermes BYOK API unavailable | >24h sustained 429 / 5xx | Disable Action Intent Object generation; manual triage only |
| Rail MCP Server tool failure | Any single tool unavailable >12h | Surface "Data source unavailable" to Fleet Manager; review by Nomad ops |
| Landside ingestion gap from brain | >72h without IntentPacket sync on any active train | Coordinate with brain team — root cause may be onboard or transport |

### Measurable Outcomes (Pilot Evidence Targets)

| Evidence | Method | Target |
|---|---|---|
| ECM trail completeness | Log IntentPacket generated time vs HAFAS scheduled arrival time vs ECM open time vs sign-off time. Pre-arrival = ECM brief opened before HAFAS scheduled arrival. | >80% pre-arrival review rate |
| Fleet Manager decision latency | Instrument time-to-decision for redeployment calls before/after | Measurable reduction |
| Audit trail usage correlation | Sign-off timestamp vs briefing-read timestamp — did reading the brief change the outcome? | Qualitative + quantitative |
| Action Intent Object approval rate | Approval action log per intent_id, segmented by `action_type` | >70% approval, rejection reasons catalogued |

**Pilot calendar anchor:** Pilot schedule is tracked in **mandays**, not calendar days, because test bench availability depends on hardware procurement lead time (Hailo-8 and Hailo-10 units must be ordered before bench testing begins — this is the gating dependency). First deployment targeting pilot manday 1 (Type Approval achieved April 21, 2026; actual start date TBD pending hardware delivery). "Baseline mandays" = first ~10 mandays of active platform use; "measurement mandays" = mandays 20–30. Roland Ruisz sign-off meeting targeted around manday 15. Rollout recommendation within 6 months of pilot manday 1. **Pre-pilot gate:** confirm Hailo-8 and Hailo-10 order placed and delivery date confirmed before pilot manday 1 is set. **Probebetrieb dependency (Journey 6 risk):** Journey 6 (Vendor Warranty Triage) requires the platform to be active during an active Probebetrieb commissioning cycle. Pre-pilot gate: confirm which Probebetrieb cycles are active during the pilot window with Lukas's programme team. If no cycle overlaps, Journey 6 is a post-pilot story and the pilot evidence dashboard drops the warranty triage claim.

**Baseline methodology (prevented delay-minutes) — approximation approach:** A precise historical baseline from ÖBB's Power BI Diagnoseplattform or equivalent is not expected to be available for the pilot. The prevented delay-minutes figure will be calculated as an approximation: fault detected by platform → no service disruption → cross-reference against industry-standard or ÖBB-acknowledged mean delay for the same fault class. Roland Ruisz to agree on the approximation methodology and the fault-class reference values before pilot manday 1. This is a best-effort evidence narrative for the ROI conversation, not a statistically rigorous baseline — the goal is a credible order-of-magnitude figure, not audit-grade precision.

**ECM completeness gap (80% → 95%):** The 80% pre-arrival review target is platform-delivered. The 95% business success target requires process change (ECM Manager opens brief before leaving for depot, not on arrival). Martin Lerch owns the process workstream.

---

## Product Scope

### MVP — Minimum Viable Product

Three landside surfaces, delivered in priority order:

1. **Depot Briefing Interface** — PWA, mobile-first. Role-filtered fault feed (ECM Manager / Electrical / Mechanical / IT), ECM 1 blocking authorisation state, append-only sign-off with IdP-bound identity, pre-arrival brief generated from IntentPackets. BOOM integration descoped — structured export/copy for manual BOOM entry.

2. **Fleet Manager AI Interface** — The primary landside control surface. Hermes BYOK, Rail MCP Server (9 tools), three entry states (triage 70/30, crisis full-width, trend 30/70), pgvector semantic search over incident summaries, cost-impact one-liner per triage card, shift handover summary, relay card for Dispatcher forwarding, Hermes-generated Action Intent Object approval cards.

3. **Telematics Project Management surfaces** *(from Gemini research)* — the **ÖBB Unified Telematics Control Panel**: a single workspace presenting three linked zones to the Teil-Projektleitung Telematik persona. Zone 1 (Macro Situational Awareness): fleet compatibility matrix + geospatial telemetry overlay. Zone 2 (Micro Visual Train Topology): SVG train anatomy + inline SAP-sourced metadata drawers per hardware node. Zone 3 (Hermes AI Co-Pilot Console): chat + Proactive Solution Cards in Action Intent Object format. Zones are bound by interaction: selecting a vehicle in Zone 1 opens its topology in Zone 2; a fault detected in Zone 1 or 2 generates a Proactive Solution Card in Zone 3. Use cases covered: trial run validation (Probebetrieb), cross-fleet modernization monitoring (DOSTO Neu pilot; Cityjet Enzo onboarded on rollout success — both are new Stadler deliveries, not retrofits), OTA software rollout with canary + drift detection, root-cause warranty triage, digital twin / cross-entity validation after workshop hardware swaps, multi-vehicle procurement spec export (Lastenheft), closed-loop trust recovery with per-alert-type accuracy metrics, ROI analysis. **Scope locked as MVP** — FR36–FR50 below are the binding contract for this surface.

**Infrastructure MVP:**
- `rail-mcp-server/` — 9-tool HTTP MCP endpoint (spec frozen — see `rail-mcp-server-spec-2026-05-23.md`)
- `ecm_signoff_events` + `shift_handovers` tables in PostgreSQL
- 4 Hermes spikes run and findings documented (pre-story gate)
- pgvector fallback ranking strategy defined and spiked before Fleet Manager stories
- IntentPacket landside ingest endpoint — REST + SSE from oebb-brain MQTT broker, idempotent on `packet_id`

### Growth Features (Post-MVP)

- Fleet Manager 48–72h forward planning view alongside real-time triage
- Shift handover cross-session AI conversation context
- BOOM API integration when API contract confirmed with ÖBB IT
- ServiceNow ITSM bidirectional integration (ticket creation from Fleet Manager triage)
- SAP PM deeper integration — automatic G-Check work order creation from Action Intent Objects
- Multi-tenant deployment — credential isolation, per-operator Rail MCP Server instances
- Dismissal signal arbitration queue (conductor dismissals from brain feed into Fleet Manager threshold tuning loop)

### Vision (Future)

- Hermes agents that proactively surface fleet patterns without query — push-based fleet intelligence
- ECM audit trail integration with ÖBB's internal compliance documentation system
- Lastenheft procurement requirements export — clauses generated from cumulative operational evidence (descoped from MVP; format, consumer, and failure threshold to be defined post-pilot)
- Multi-operator rail network deployment (non-ÖBB carriers)
- Configuration drift auto-remediation — Hermes proposes patch deployment to fleet trains showing drift

---

## Disaster Recovery (Landside)

| Failure | Behaviour | Recovery |
|---|---|---|
| PostgreSQL unavailable | ECM sign-off writes queue locally at depot, sync on reconnect. Fleet Manager AI Interface shows stale data with timestamp. Rail MCP Server returns 503 with `recoverable: true`. | Depot assumed to have fixed network — extended outage triggers ops alert. ECM sign-off writes resume on reconnect; append-only guarantee preserved. |
| pgvector index failure | Fleet Manager semantic search degrades to keyword fallback ranking. No silent failure — UI labels results "Keyword search active". | pgvector restart clears index; re-index from `incident_summary` embeddings. Spike required pre-Fleet Manager stories to define fallback behaviour and re-index SLA. |
| Hermes BYOK API unavailable (429 / 5xx) | Fleet Manager AI chat shows "Fleet intelligence temporarily unavailable". Triage and crisis views remain fully functional — they do not depend on Hermes. Retry with exponential backoff + budget cap. | BYOK 429 handling is Hermes spike 4. |
| Rail MCP Server tool failure | Hermes receives structured error `{error, detail, recoverable}`. Recoverable errors retried once. Non-recoverable surfaced to Fleet Manager as "Data source unavailable for [tool name]". | Each tool is independently recoverable. |
| Brain → landside ingest gap | IntentPacket sync paused. MQTT retain=True surfaces only the last-known packet on reconnect — gap-period packets are not on the MQTT broker. Fleet Manager and Telematics surfaces show staleness indicator on affected vehicles based on sequence-number or heartbeat detection (FR45). Audit trail gap flagged with last-known timestamp. | Brain replays unacknowledged packets via its dedicated replay channel on reconnect; landside acknowledges per `packet_id` (idempotent insert). Replay-channel spec is brain-owned. Until replay completes, affected surfaces remain in staleness state — no false-fresh data. |
| Action Intent Object dispatch partial failure | If any dispatched operation fails (e.g. HAFAS update succeeds but a subsequent operation fails), platform writes `intent_id` to reconciliation queue. Fleet Manager surfaces "Partial execution — review" card. | Manual reconciliation by Disponent; subsequent dispatch attempts use idempotency key. |

### ECM Audit Trail Integrity

- `ecm_signoff_events` is append-only enforced at database level (row-level trigger raises exception on UPDATE/DELETE attempt)
- Corrections are new rows with `supersedes_id` pointing to corrected record — no in-place mutation ever
- `signature` field (HMAC) provides tamper evidence for each row
- Extended connectivity loss: sign-off writes queue locally at depot, sync to landside PostgreSQL on reconnect. Queue is durable — depot device loss before sync is an unrecoverable gap, flagged in audit as missing record with last-known timestamp
- Sign-off gesture (Option C): 3-second hold-to-initiate → 5-second countdown → single atomic write at expiry. No server-side pending state — the write fires exactly once, at countdown expiry. Cancellation at any point during countdown prevents write. After write, record is permanent — no undo. Idempotency key generated at countdown initiation; retries with the same key are safe (409 Conflict treated as success). See `design-artifacts/ecm-signoff-interaction-spec.md` for full state machine

---

## User Journeys

### Journey 1 — ECM Manager: "The Pre-Arrival Brief" (Primary — Success Path)

**Meet Klaus.** Klaus is an ECM 1 Manager at Wien Westbahnhof depot. He is accountable under EU Directive 2019/779 for every vehicle that returns to service. He arrives cold — he was not on shift when the train came in last night. Unit 4722 is in Bay 3.

**Before the platform.** Klaus walked to Bay 3, asked the technician what happened, read a paper fault log, and made a judgment call. If he missed something and the vehicle failed in service, the liability was his and the paper trail was thin.

**Now.** At 05:50, fourteen minutes before Unit 4722 docks, the landside platform generates a depot briefing from the journey's IntentPackets (received from oebb-brain over MQTT). Klaus opens the Depot Briefing Interface on his phone. Role: ECM Manager. He sees: two faults sorted by severity. CRITICAL: Door motor fault, Car 4 Door 2L — resolved by conductor at 07:14, replacement part fitted at 03:22 by Electrical team. WARNING: SNMP camera firmware version mismatch on VLAN 5 — acknowledged by IT technician, deferred to next maintenance window with reason logged.

**The climax.** Klaus taps "Review & Authorise" on the door motor fault. He reads the full resolution chain — detection → conductor ACK → technician assignment → part fitted → test result. He holds to confirm. The `ecm_signoff_events` record writes: his IdP identity, UTC timestamp, full vehicle state snapshot at sign-off, directive reference EU 2019/779.

**The new reality.** Unit 4722 clears to service at 06:08. Klaus has a legally defensible audit trail. The platform did not authorise the vehicle — Klaus did. The platform made his decision traceable.

*Capabilities revealed: Pre-arrival brief generation, role-filtered fault feed, ECM 1 blocking state, hold-to-authorise gesture, append-only sign-off, full resolution chain view.*

---

### Journey 2 — Fleet Manager: "The Crisis Redeployment" (Operations — Success Path)

**Meet Stefan.** Stefan is a Fleet Manager (Disponent) at ÖBB Wien operations. He is responsible for cost management through fleet deployment across a 12-hour shift. It is 07:23. He opens the Fleet Manager AI Interface.

**The modal fires.** "Unit 4722 — Door Motor Fault, Car 4. Active unresolved incident. View?" He taps yes. Full-width crisis view. AI chat pre-seeded: "Unit 4722 door motor fault detected at 06:14. Conductor resolved manually at 07:14. Vehicle cleared by ECM at 06:08. Replacement unit available: Unit 4819."

**Stefan queries.** "What does deploying 4819 cost versus keeping 4722 on route?" Hermes calls `get_maintenance_cost_estimate(unit_id="4722", replacement_unit_id="4819")`. Response in 2.1 seconds: "Replacing with Unit 4819 adds ~€2,400 estimated maintenance overhead. Confidence: medium. Rationale: 4819 last serviced 34 days ago, approaching mileage threshold." The cost one-liner was already in the triage card before he typed the question.

**Action Intent Object.** Stefan asks: "Redeploy 4819 to take 4722's slot on the next rotation." Hermes generates an Action Intent Object: `{action_type: "FLEET_REDEPLOYMENT", operations: [HAFAS rotation update at 22:15]}`. Rendered as a one-click approval card in the Hermes console zone. *(SAP work order routing is post-MVP — MVP dispatches to HAFAS only.)*

**The decision.** Stefan reviews the card. He taps "Approve Redeployment." Platform verifies his active IdP session (Keycloak in MVP), writes an immutable log entry to `action_intent_audit`, and dispatches the HAFAS rotation update. Stefan sees "Executed at 07:24:11."

**The new reality.** Stefan made a €2,400 decision in 90 seconds with evidence, and the rotation update was applied without him opening HAFAS manually. Prevented delay-minutes: 14-minute delay contained at conductor level; the redeployment is for the *next* rotation, not this one. The counterfactual is logged.

*Capabilities revealed: Crisis entry state, incident modal, pre-seeded AI chat, Rail MCP tool calls, cost-impact one-liner, Action Intent Object generation, one-click approval interlock, multi-system dispatch with audit log.*

---

### Journey 3 — Dispatcher: "The Relay" *(Post-MVP — Dispatcher persona deferred from MVP scope)*

**Meet Ana.** Ana is a Fahrdienstleiter at Salzburg control room. She monitors signal and switch systems across her section. At 14:32 an IntentPacket arrives: Train 4712, 14-minute delay, signal fault at Salzburg Hbf.

**What Ana sees.** In the Fleet Manager AI Interface, alongside the triage alert, a relay card appears: "Train 4712 is delayed 14 minutes due to signal fault at Salzburg Hbf. Expected recovery: on departure." Plain language. Passenger-safe. No fault codes.

**The action.** Ana taps "Forward to Conductor." The relay card is pushed to the conductor's PWA notification (via brain). Ana taps "Copy" and pastes it into the PA system announcement. She did not rewrite anything. She did not call the conductor. Thirty seconds from alert to passenger announcement.

**The new reality.** The relay card was generated by Hermes from the IntentPacket `incident_summary`. Ana's job is coordination, not translation. The platform handled the translation.

*Capabilities revealed: Relay card component, LLM-generated passenger-safe language, copy-to-clipboard, forward-to-conductor push notification, Dispatcher coordination loop.*

---

### Journey 4 — ECM Lead: "Reviews the Platform" (Commercial Confidence Journey)

**Meet Dr. Renate Fischer.** Dr. Fischer is Head of Vehicle Maintenance at ÖBB Technische Services — the ECM-accountable role. She was not in the pilot planning meeting. Martin Lerch brings her in at approximately pilot manday 15. Her question is not "does it work?" — it is "does our audit trail map to our internal ECM documentation requirements, and would it survive a regulatory inspection?"

**What she reviews.** She opens the ECM audit trail export for Unit 4722. She sees: IntentPacket generated at 05:50 (14 minutes pre-arrival), opened by Klaus at 05:53, fault resolution chain — detection → conductor ACK → technician assignment → part fitted → test result, sign-off at 06:08 by Klaus (IdP identity, not a local username), vehicle state snapshot at sign-off, directive reference EU 2019/779, HMAC signature on each row.

**Her first question.** "Can I export this in a format our documentation system accepts?" PoC answer: structured JSON export, copy-paste ready. BOOM integration deferred — noted, not a blocker.

**Her second question.** "What happens if Klaus signs off incorrectly — can we correct it without deleting the record?" She is shown the `supersedes_id` pattern — a new row points to the original, original is never touched. She is satisfied.

**The outcome.** Dr. Fischer does not object. She requests to be a reference customer for the compliance output. Roland gets his ECM sign-off for the rollout recommendation. Late-stage objection risk eliminated.

*Capabilities revealed: ECM audit trail export, full resolution chain view, supersedes_id correction flow, IdP-bound identity display, HMAC tamper evidence, structured JSON export.*

---

### Journey 5 — The ROI Conversation (Commercial — Pilot Week 3)

**Meet Roland and Martin.** It is approximately pilot manday 15. Roland Ruisz (operations) and Martin Lerch (Technical Services) sit down with the pilot evidence dashboard. They are not assessing features — they are assessing whether to sign a rollout recommendation.

**What they see.** The pilot health score composite: ECM pre-arrival review rate 83% (target >80% ✓), dispatcher adoption 54% (target >50% ✓), Disponent decision latency reduced by 38% vs baseline, Action Intent approval rate 76%. Below: prevented delay-minutes. Seven faults detected pre-disruption (sourced from brain telemetry). Historical baseline for same fault classes on unmonitored units: 23-minute mean delay. Estimated delay-minutes prevented: 161.

**Martin's question.** "What does this cost us to run per train per month?" The platform surfaces infrastructure cost estimate. Roland cross-references against 161 delay-minutes prevented — at ÖBB's internal cost-per-delay-minute figure, the platform pays for itself in under two months of full fleet deployment.

**The outcome.** Roland does not need to argue the case to his CFO. The platform produced the evidence. Martin does not need to justify IT infrastructure spend — the ROI is in the dashboard. The rollout recommendation is signed at week 6, not week 24.

*Capabilities revealed: Pilot evidence dashboard — composite health score with threshold line, prevented delay-minutes counterfactual panel, per-criterion trend view, infrastructure cost estimate input, export for executive presentation.*

---

### Journey 6 — Teil-Projektleitung Telematik: "Vendor Warranty Triage" (Gemini-derived) ⚠️ *Scope assessment required*

> **[NEEDS DECISION]** Journey 6 depends on: (1) an active Probebetrieb commissioning cycle overlapping the pilot window — confirm with Lukas's programme team; (2) SAP asset registry integration (post-MVP in current scope — metadata drawers will use local registry in MVP). If neither dependency is met during the pilot, this journey is a post-pilot story. The warranty evidence packaging capability (FR41) is still in MVP scope, but cannot be evidenced during the pilot without an active Probebetrieb cycle.

**Meet Lukas.** Lukas is Teil-Projektleitung Telematik for the Cityjet DOSTO Neu programme. Seat reservation screens have gone black on three of the new Stadler double-decker units over the last week. Stadler's first response: "Check your landside reservation database — it's probably timing out our bus controller."

**Before the platform.** Lukas would have spent two days pulling logs from the vendor diagnostic portal, cross-referencing against ÖBB's PostgreSQL reservation service uptime in Power BI, exporting timestamps to Excel, and writing a Change Request rebuttal email. Likely outcome: Stadler holds the line, and the dispute escalates to procurement.

**Now.** Lukas opens the Telematics control board. He filters the fleet compatibility matrix to the affected units. He clicks one unit, opens the SVG train topology, and selects the reservation screen node. Inline drawer surfaces: MAC address, firmware version, last 24h packet trace from VLAN 6 (reservations), and the corresponding landside reservation service response latencies during the failure window. Hermes is already on the case in zone 3: "Reservation service response P99 was 187ms during all three failure windows. Bus controller memory utilisation crossed 94% in the same window on all three units. Pattern consistent with onboard memory leak, not landside latency."

**The action.** Lukas approves an Action Intent Object: "Package evidence dump for warranty claim — Stadler ticket #SR-2026-0517." Hermes assembles the timestamp-correlated logs, generates a structured warranty claim document, and exports to PDF. Lukas attaches it to the Stadler ticket. Two days later, Stadler authorises the bus controller swap under warranty.

**The new reality.** What used to be a multi-day forensic exercise across four systems is now a 20-minute investigation with platform-generated evidence. The vendor lock-in dynamic shifts because ÖBB now has independent diagnostic visibility.

*Capabilities revealed: Macro fleet compatibility matrix, micro SVG train topology with metadata drawers (IntentPacket + local registry in MVP), Hermes correlation across landside + onboard telemetry, warranty evidence packaging Action Intent Object, vendor-facing structured export.*

---

### Journey 7 — Teil-Projektleitung Telematik: "OTA Rollout Monitoring" (Gemini-derived)

**Meet Lukas again.** A critical security patch for the passenger information system has been deployed across the DOSTO Neu pilot fleet by Nomad Rail operations. Last time this happened without visibility, an inconsistent patch level reached three workshops (Simmering, Floridsdorf, Linz) and Lukas spent a week reconciling Excel matrices to find the drifted unit.

**The flow.** Lukas opens the OTA rollout monitoring surface in the Telematics control board. The platform shows the current firmware and software version reported by each unit in the fleet — sourced from the most recent IntentPacket received from the brain for each train. The brain retrieves version state from the onboard Stadler diagnostic system and includes it in the IntentPacket payload. Lukas can see at a glance which units have confirmed the new patch level and which are still on the prior version or have not reported in.

**The validation.** The canary unit reported the new patch version at 06:14. Its last IntentPacket shows uptime nominal, link stability nominal. At hour 24 of monitoring, all but one unit in the fleet have reported the updated version. The platform surfaces the outlier: "Unit DOSTO-Neu-4218: last reported firmware version 2.3.1 (expected 2.4.0). Last IntentPacket received 4h ago." Lukas flags it for manual depot check.

**Configuration drift detection.** The platform compares the reported hardware revision in each unit's latest IntentPacket against the local asset registry. One unit reports hardware revision B+ against an expected revision B baseline. The platform surfaces: "Hardware revision mismatch — manual review required before next patch cycle." Without the platform, this would have been a Simmering workshop discovery two weeks later, with one drifted unit in service in the meantime.

*Capabilities revealed: Fleet-wide firmware/software version monitoring from IntentPacket data, canary unit status tracking, per-unit last-reported-version display with staleness indicator, configuration drift detection (reported revision vs local registry), outlier surfacing for manual follow-up.*

---

### Journey 8 — System Degraded: Fleet Manager Under Stale Data (Edge Case)

**The scenario.** 09:14. Onboard brain inference stalls on a specific train — the brain's IntentPacket emission has paused for 80 seconds. The landside platform's MQTT consumer keeps the last retained packet but no new ones arrive.

**What Stefan sees.** In the Fleet Manager triage view, the fleet card for Train 4712 shows a staleness indicator — last updated timestamp, amber border. He queries Hermes: "What is the current status of Train 4712?" Hermes calls `get_fleet_summary`. The Rail MCP Server responds with `data_freshness_ts` showing 94 seconds ago. Hermes surfaces this: "Last known status 94 seconds ago — onboard inference delayed. Data may not reflect current state." Stefan waits. He does not act on stale data.

**MCP timeout.** Stefan asks for the cost estimate on a redeployment. The Rail MCP Server call to `get_maintenance_cost_estimate` times out after 8 seconds. The triage card shows: "Cost estimate unavailable — data source timeout." Stefan makes the redeployment decision without the cost figure. The platform records his decision with a flag: cost data unavailable at decision time.

**Recovery.** Brain inference resumes at 09:17. Fresh IntentPacket arrives. Stefan's fleet card updates. No data is lost. No decision is silently affected.

*Capabilities revealed: Landside staleness indicator on stale brain data, MCP timeout handling — structured error surfaced to UI, decision logging with data-availability flag, automatic recovery on brain ingest resume.*

---

### Journey 9 — ECM 4 Technician: "The Fault ACK" (Primary — Operational Path)

**Meet Marco.** Marco is an ECM 4 Electrical Technician at Wien Westbahnhof depot. He covers the overnight maintenance window. He is not accountable under 2019/779 — that is Klaus's responsibility — but nothing moves on Klaus's screen until Marco logs his work.

**Before the platform.** Marco read fault codes off the diagnostic terminal, wrote the resolution on a paper sheet, handed it to the shift supervisor, who transcribed it to the system. If he used the wrong fault code, the ECM sign-off referred to a resolution that didn't match the physical repair.

**Now.** At 03:15, Marco opens the Depot Briefing Interface on his tablet. He taps "Electrical" — his discipline. He sees one fault in his queue: CRITICAL, Car 4 Door 2L motor fault. He opens it. The full IntentPacket chain is visible: conductor ACK at 01:20, fault isolated to door motor assembly, part requisitioned.

**The ACK.** Marco fits the replacement part at 03:22. He taps "Acknowledge — Resolved." The structured ACK record writes: his IdP identity, UTC timestamp, resolution action ("Replacement part fitted — Door Motor Assembly Car 4 Door 2L"), and the fault ID it references. No free text. The fault moves from OPEN to RESOLVED in Klaus's pre-arrival brief.

**What he cannot do.** Marco cannot sign off the vehicle — that requires ECM 1 authority. He cannot see faults outside Electrical — Mechanical and IT faults are not in his feed. He cannot modify or delete his ACK after it is written.

**The new reality.** Klaus sees Marco's ACK in the resolution chain at 05:50. The platform told Klaus who fixed it, when, and what they did — before Klaus left for the depot. The handover happened asynchronously.

*Capabilities revealed: Role-filtered fault feed (Electrical discipline only), structured ACK with IdP-bound identity, append-only ACK record, fault state transitions (OPEN → RESOLVED), ACK visible in downstream ECM brief.*

---

### Journey Requirements Summary

| Capability area | Revealed by |
|---|---|
| Pre-arrival depot brief generation | Journey 1 |
| Role-filtered fault feed (ECM/Electrical/Mechanical/IT) | Journey 1 |
| ECM 1 blocking authorisation + hold-to-confirm | Journey 1 |
| Append-only sign-off with full resolution chain | Journeys 1, 4 |
| Crisis entry state + incident modal | Journey 2 |
| Rail MCP tool call (cost estimate, fleet summary, telemetry) | Journeys 2, 6, 7, 8 |
| Cost-impact one-liner per triage card | Journey 2 |
| Pre-seeded AI chat with incident context | Journey 2 |
| Action Intent Object generation + one-click approval interlock | Journeys 2, 6, 7 |
| Multi-system dispatch with audit log | Journey 2 |
| Relay card + LLM passenger-safe language | Journey 3 *(post-MVP — Dispatcher persona deferred)* |
| Forward-to-conductor push notification (to brain) | Journey 3 *(post-MVP — Dispatcher persona deferred)* |
| ECM audit trail export + supersedes_id correction | Journey 4 |
| Pilot evidence dashboard + prevented delay-minutes panel | Journey 5 |
| Fleet compatibility matrix + geospatial overlay | Journey 6 |
| SVG train topology + inline metadata drawers (IntentPacket + local registry; SAP post-MVP) | Journey 6 *(scope assessment required — see Journey 6 note)* |
| Hermes correlation across landside + onboard telemetry | Journey 6 |
| Warranty evidence packaging | Journey 6 |
| OTA rollout monitoring — per-unit firmware/version from IntentPackets, outlier surfacing, staleness indicator | Journey 7 |
| Configuration drift detection (local registry in MVP; SAP asset registry post-MVP) | Journey 7 |
| Stale data indicator on brain ingest gap | Journey 8 |
| MCP timeout — structured error to UI, decision flag | Journey 8 |
| Role-filtered fault ACK (Electrical discipline) | Journey 9 |
| Structured ACK with IdP-bound identity, append-only | Journey 9 |
| Fault state transition (OPEN → RESOLVED) visible in ECM brief | Journey 9 |
| Unified Telematics Control Panel — 3-zone cross-zone interaction binding | FR50, FR51 |
| Multi-programme compatibility matrix (DOSTO Neu pilot → Enzo expansion) | FR36 |
| Geographic vs hardware staleness disambiguation | FR48 |
| Digital Twin validation after workshop hardware swap | FR49 |
| Topology node graceful degradation under partial failure | FR54 |
| Predictive edge alert → workshop reservation | FR53 |
| Probebetrieb end-to-end network path validation | FR52 |
| Lastenheft procurement clause export | *(Descoped — moved to Vision)* |
| Per-alert-type accuracy metric + Dismiss tuning signal | FR56 |

---

## Domain-Specific Requirements

### Compliance & Regulatory

**EU ECM Directive 2019/779**
- Every vehicle returning to service must have a documented ECM sign-off by an authorised ECM 1 entity
- Sign-off record must identify the authorising individual by their regulatory identity (ÖBB SSO IdP — not a local session username)
- Audit trail must be tamper-evident and append-only — no UPDATE or DELETE permitted on `ecm_signoff_events`
- Corrections are new rows with `supersedes_id` referencing the corrected record
- Vehicle state at moment of sign-off must be captured as a JSONB snapshot
- `directive_ref` field (`EU 2019/779`) present on every sign-off record

**GDPR**
- Raw video never leaves the train (handled by brain — landside never receives video). Only anonymised event metadata reaches landside.
- No personally identifiable passenger data in any IntentPacket or event envelope reaching landside
- Events tagged for deletion scope per GDPR Article 17 — retention policy required before fleet rollout

**NIS2**
- Platform classified as critical infrastructure operator — ÖBB's NIS2 classification applies
- Incident reporting obligations: material cybersecurity incidents must be reported to national authority within 24h. **Operational owner: ÖBB IT Security (ÖBB PV and BCC departments** — not Martin Lerch's org; Martin Lerch is ÖBB Technische Services). The platform's obligation is to emit structured security incident events (authentication anomaly, unauthorised sign-off attempt, BYOK key exposure, LLM injection detection) to ÖBB's security monitoring pipeline — not to perform the regulatory filing itself. FR58 covers this. **Becomes a PoC blocker when:** (a) platform handles live operational data for trains in revenue service, or (b) PoC scope expands beyond 5 trains. Whichever comes first. Before that threshold is crossed, the security incident runbook must be signed off by ÖBB PV/BCC security and Nomad Rail legal.
- BYOK API keys for Hermes must never appear in source control, logs, or inference code paths
- Prompt injection threat model: all LLM output sanitised before reaching UI render layer

### Technical Constraints

**Landside Infrastructure**
- PostgreSQL for all operational state — no React component state or Hermes conversation memory for acknowledged alerts or status changes
- pgvector extension required — `CREATE EXTENSION vector` needs superuser or `pg_extension_owner` role; migration must include `IF NOT EXISTS` guard
- **pgvector P99 latency SLA: <500ms** — if exceeded, fallback to keyword ranking (`search_fleet_events` full-text), logged as WARN; UI labels results "Keyword search active". Formal acceptance criterion for the pgvector spike story.
- Alembic migrations safe under concurrent reads — no table locks during pilot operations
- All API responses snake_case JSON; React converts to camelCase at API client layer only

**Brain → Landside Ingest**
- MQTT subscriber consumes `oebb/{train_id}/intent` topic. QoS 1 + retain=True provides *last-known-state* on subscriber reconnect — it does **not** replay packets emitted during the disconnect window. Gap-period replay is the responsibility of the brain's WAL-before-truncate sync (see brain disaster-recovery story); landside is a pull-eligible target for that replay, not a passive recoverer
- Gap-recovery contract: brain replays unacknowledged packets to landside via a dedicated replay channel after reconnect; landside acknowledges per `packet_id`. The replay channel is brain-owned spec — landside subscribes
- REST + SSE bridge from MQTT to PostgreSQL; `packet_id` is idempotency key on insert (handles both live and replayed packets)
- Schema contract is brain-owned (v1.0.0 frozen); landside validates against locally-cached schema definition
- Landside detects ingest gaps by monitoring per-train packet sequence numbers (when present) or heartbeat intervals; gaps are surfaced via FR34/FR45 staleness indicators independently of replay completion
- **No landside-initiated diagnostic commands to brain in MVP.** The brain emits on every state transition within 2s — there is no polling cadence to bypass. On-demand diagnostic tools (e.g. `trigger_edge_diagnostics`) are dropped from MVP scope; see §9 decision log.

**Security**
- BYOK API keys stored in environment variables (PoC) or Docker secrets (fleet) — never in source control
- All external data (IntentPackets, MCP tool inputs, vendor diagnostic feeds) treated as untrusted — schema validation before any AI decision path
- LLM output sanitisation layer required between Hermes response and Fleet Manager UI render
- Action Intent Object dispatch requires re-verified SSO session at moment of approval (not just at login)

### Integration Requirements

| System | Integration type | Status | Notes |
|---|---|---|---|
| Identity provider (IdP) | Identity for ECM sign-off + Action Intent approval | **MVP: Keycloak with manually created users.** ÖBB IdP integration is a post-MVP change request, costed separately. | Must be IdP-bound, not local session |
| MQTT broker | IntentPacket consumption from oebb-brain | Owned by brain — landside is subscriber | `oebb/{train_id}/intent` |
| Rail MCP Server | HTTP MCP endpoint — spec frozen. 9 tools: `get_fleet_events`, `get_fleet_summary`, `get_hafas_timetable`, `search_fleet_events`, `get_maintenance_cost_estimate`, `get_shift_handover`, `get_ecm_audit_trail`, `get_intent_packet`, `get_forward_planning`. `trigger_edge_diagnostics` removed 2026-05-25 (see §9 ADR). Spec authority: `rail-mcp-server-spec-2026-05-23.md` | Spec frozen | New subpackage `rail-mcp-server/` |
| Hermes (BYOK) | LLM orchestration agent runtime | Designed, not built | Nous Research, MIT licence |
| HAFAS | Timetable data + rotation assignment updates via Action Intent Objects | Required for Fleet Manager + Telematics | 48h horizon for planning; write-back for redeployment |
| SAP PM/EAM | Asset registry + work order creation | **Post-MVP** — no SAP integration in PoC. Topology metadata drawers and work order write-back require API contract with ÖBB TS. | Read for asset metadata; write for work order routing |
| ServiceNow ITSM | Ticket creation from Fleet Manager triage | **Post-MVP** | Bidirectional sync — incidents flow both ways |
| BOOM | Depot work order integration | **Descoped for PoC** | Structured JSON export/copy for manual entry |
| pgvector | Semantic search over `incident_summary` embeddings | Requires migration spike + P99 SLA confirmation | Fallback: keyword ranking |
| Vendor diagnostic portals (Stadler, Siemens) | Read-only — evidence for warranty triage | Post-MVP | Authentication via vendor-issued credentials; scoped read |
| Power BI Diagnoseplattform | Coexistence — high-level fleet uptime visualisation | No integration in MVP | Platform does not replace; Telematics persona uses both |

### Risk Mitigations

| Risk | Mitigation |
|---|---|
| BYOK prompt injection | Output sanitisation layer between Hermes and UI render — validated schema, reject unexpected fields, log anomalies |
| pgvector migration blocks reads | Migration with `IF NOT EXISTS` guard; confirm PostgreSQL user has extension creation rights; document `DROP EXTENSION vector CASCADE` rollback risk |
| ECM sign-off record loss (connectivity gap) | Queue locally at depot, sync on reconnect; extended loss flagged in audit as gap with last-known timestamp |
| BOOM API unknown | Scoped as "BOOM-ready" not "BOOM-integrated" — structured export only until API contract confirmed with ÖBB IT |
| NIS2 incident reporting obligation | Platform emits structured security incident events (FR58); ÖBB IT Security (ÖBB PV and BCC departments) owns the 24h regulatory filing. Runbook must be signed off before platform handles live revenue-service data or exceeds 5-train scope — whichever comes first. Owner: ÖBB PV/BCC security + Nomad Rail legal. |
| BYOK data residency not confirmed | BYOK keys and vehicle state snapshots must be designated to EU GDPR processors. **Blocks procurement sign-off for multi-tenant rollout.** PoC unaffected. Must be resolved before commercial expansion. |
| pgvector latency exceeds SLA | Fallback to keyword ranking (WARN logged, UI labelled). Spike story must confirm or revise <500ms P99 target before Fleet Manager stories begin. |
| Action Intent Object partial dispatch | Reconciliation queue with idempotency keys; Fleet Manager surfaces partial-execution cards explicitly |
| Hermes data residency vs vendor portal credentials | Vendor portal credentials must not pass through inference code path; resource initialisation pattern enforced |

### UX Constraints (Domain-Driven)

**ECM Sign-Off Confirmation — Option C (resolved 2026-05-25)**

The sign-off interaction is a two-phase gesture: hold-to-initiate (3s) → 5-second visible countdown → single atomic write at expiry. Full interaction arc specified in `design-artifacts/ecm-signoff-interaction-spec.md`.

Six screen states:
- **STATE 0 — Idle:** Vehicle ID, location, timestamp, open-defect count, hold button (disabled if defects > 0), legal notice
- **STATE 1 — Hold Active (3s):** Progress bar fills in button via `requestAnimationFrame`; haptic 40ms pulse per second; early release returns to STATE 0 with no write
- **STATE 2 — Countdown (5s):** Idempotency key generated at STATE 1→2 transition (`crypto.randomUUID()`). 5 dots + numeric countdown. Haptic 80ms pulse per second. Full-width cancel button. **No server request yet.** Countdown pauses on `visibilitychange` (radio call, incoming notification) — resumes on focus return. Cancel returns to STATE 0, no log entry
- **STATE 3 — Writing:** Single atomic POST with `X-Idempotency-Key` header. Cancel button gone. `409 Conflict` on retry treated as success → STATE 4
- **STATE 4 — Confirmed:** Protokoll-ID (server record ID), vehicle, location, timestamp, technician, verdict displayed at `text-lg` minimum. Klaus reads this aloud to dispatcher. "Dispatcher bestätigt" writes a lightweight ack row (not a second commit). Haptic double-pulse `[100, 50, 100]` on entry. Non-blocking toast if card closed in < 15s
- **STATE 5 — Error:** "Die Freigabe wurde NICHT gespeichert." in `text-3xl text-red-600`. Retry uses same idempotency key — safe against lost-response double-commit

Key constraints:
- Hold threshold: **3 seconds** (not 2 — accidental-trigger range in gloves at 2s)
- Client-side timer only — no server-side pending state (eliminates accidental commit under network jitter)
- Idempotency key generated at countdown initiation, sent as `X-Idempotency-Key` header — safe retries
- Countdown pauses on focus loss — Klaus is not committed while on a radio call
- Protokoll-ID is mandatory in STATE 4 — chain of custody requires Klaus to be able to read it aloud
- Green only appears in STATE 4 — not in the button, not in progress indicators

**Open gate (pre-go-live, not pre-dev):** One walkthrough with an ECM 4 Technician timing the 8-second total gesture (3s hold + 5s countdown) against a real 14-minute depot window. If 8s is unacceptable, countdown compresses to 3s. Two-phase structure is non-negotiable regardless.

**pgvector Fallback Transparency**
- Fleet Manager always knows which search mode is active — semantic or keyword
- UI label "Keyword search active" shown whenever fallback is in effect
- No silent degradation permitted

**Action Intent Object Approval**
- The approval button is a hard authentication gate — re-verifies SSO session at moment of click, not just at app load
- Pre-dispatch summary: explicit list of every system being touched (HAFAS in MVP; SAP post-MVP) with the exact payload
- Approval action is logged with `intent_id`, approver identity, timestamp, dispatch outcome per operation
- Partial-execution scenarios are surfaced as distinct UI state — not buried in error logs

---

## 6. Project-Type Specific Requirements

### Project-Type Overview

This platform is a **landside B2B SaaS surface** with multi-system integration and an embedded AI agent (Hermes BYOK). It serves three landside user surfaces — Depot Briefing PWA (mobile-first), Fleet Manager AI Interface (web), and Telematics Project Management surfaces (web, 3-zone control board).

---

### Technical Architecture Considerations

**Multi-tenancy model**
PoC: single ÖBB tenant, on-premise. No multi-tenancy. Fleet rollout: one instance per operator (ÖBB, Westbahn, etc.) — shared codebase, separate deployments. Shared-database multi-tenancy explicitly deferred — BYOK data residency and ECM audit trail integrity require tenant isolation.

**Permission model**
Five landside roles, mapped to surfaces:

| Role | Surface | Permissions |
|---|---|---|
| Fleet Manager (Disponent) | Fleet Manager | Full triage; redeployment Action Intent approval; shift handover |
| ECM Manager (ECM 1) | Depot Briefing | Sign-off authority; audit trail read; blocking state control |
| Technician (ECM 4) | Depot Briefing | Role-filtered fault feed; ACK faults in own discipline only |
| Teil-Projektleitung Telematik | Unified Telematics Control Panel (primary); Fleet Manager (full — same user as Disponent) | Full fleet matrix + topology + OTA monitoring + Digital Twin validation + warranty Action Intent approval on Telematics surface. As Roland Ruisz holds both the Fleet Manager and Teil-Projektleitung Telematik roles, he has full Fleet Manager approval authority across both surfaces. |

All identity via IdP-bound session — no local user table for safety/compliance actions. **MVP uses Keycloak with manually created users.** ÖBB SSO IdP integration is a post-MVP change request, costed separately.

**IdP per-operator**
MVP uses Keycloak with manually created users — no ÖBB SSO integration. ÖBB SSO IdP integration is a post-MVP change request, costed separately. Fleet rollout: each operator brings their own IdP. Per-operator SSO customisation will be requested — budget 2–3 week integration tax per operator or build a standardised SAML bridge.

---

### Integration List

| System | Integration type | PoC status | Fleet gate |
|---|---|---|---|
| Identity provider (IdP) | Identity for ECM sign-off + Action Intent approval | **MVP: Keycloak with manually created users.** ÖBB SSO IdP integration is post-MVP — scoped as a change request. | Per-operator SAML bridge at fleet rollout |
| MQTT broker | IntentPacket consumption from oebb-brain | Onboard subscriber only | **Blocking pre-fleet:** broker identity, failover, and multi-train topic isolation specified by brain team |
| Rail MCP Server | HTTP MCP endpoint — 9 tools for Hermes | Spec frozen | Required for Fleet Manager + Telematics |
| Hermes (BYOK) | LLM orchestration, operator-owned API keys | Designed, not built | BYOK cost simulator required before fleet sales |
| HAFAS | Timetable + rotation write-back | Required for Fleet Manager | 48h horizon |
| SAP PM/EAM | Asset metadata + work order write-back | **Post-MVP** — API contract required with ÖBB TS before integration stories | Read for asset metadata; write for work order routing |
| ServiceNow ITSM | Ticket creation from triage | Post-MVP | API contract required |
| BOOM | Depot work order integration | Descoped — structured JSON export only | API contract required from ÖBB IT before integration stories |
| pgvector | Semantic search over incident_summary embeddings | Spike + P99 confirmation required | Fallback: keyword ranking |
| Vendor diagnostic portals | Read-only warranty evidence | Post-MVP | Vendor credential model required |

---

## 7. Innovation Focus

### What Is Novel Here (Landside)

This platform reframes landside fleet operations around **structured evidence infrastructure** — every operationally relevant observation, AI recommendation, and human decision is captured as a versioned, immutable event. The novel claims:

---

**Claim 1 — Hermes Action Intent Objects bridge AI advice and multi-system dispatch under a single approval gate**

The Disponent currently switches between 4–7 systems to execute one redeployment decision. The platform's Action Intent Objects bundle multi-system operations (HAFAS rotation update + SAP work order creation) into a single structured object, rendered as a one-click approval card. The human-in-the-loop interlock is preserved (SSO re-verification at moment of approval, immutable audit log entry, partial-execution reconciliation) — the cost reduction is in system-switching, not in human authority.

*Validation: Action Intent approval rate >70% in first 6 weeks of pilot. Rejection reasons catalogued — UX design feedback loop.*

---

**Claim 2 — Vendor warranty leverage from end-to-end network path visibility**

The Teil-Projektleitung Telematik persona currently fights vendor lock-in by manually correlating logs across vendor portals, Power BI, and Excel. The platform's correlation across onboard IntentPackets + landside service telemetry + local asset metadata produces a structured warranty evidence packet in 20 minutes vs the current 2-day process. *(SAP asset metadata cross-reference is post-MVP — MVP uses local registry.)* This shifts the vendor-customer power dynamic during the high-pressure Probebetrieb commissioning phase.

*Validation: Time-from-dispute to evidence-package, measured on at least 3 warranty triage cycles during pilot.*

---

**Claim 3 — Regulatory compliance as a primary feature**

The ECM audit trail is not a log — it is the product. Every sign-off is an append-only record with IdP-bound identity, vehicle state snapshot, EU 2019/779 directive reference, and HMAC tamper evidence. Corrections are new rows with `supersedes_id`. This is the evidence chain Dr. Fischer needs for a regulatory inspection.

*Validation: ECM audit export matches internal ÖBB documentation requirements — confirmed by ECM-accountable stakeholder during pilot manday ~15.*

---

**Claim 4 — OTA rollout monitoring with configuration drift detection**

Configuration drift across workshops (Simmering, Floridsdorf, Linz) is the Teil-Projektleitung's recurring pain. The platform gives ÖBB visibility over OTA rollouts executed by the train manufacturer or Nomad Rail — Teil-Projektleitung monitors rollout progress and firmware/configuration drift across the fleet; ÖBB does not execute the rollouts. The platform detects hardware revision mismatches and surfaces incompatible units for manual review — eliminating the "discover the drift two weeks later in a workshop" failure mode.

*Validation: At least one configuration drift instance detected and surfaced during a pilot OTA rollout monitoring cycle.*

---

### Innovation Risks

| Risk | Mitigation |
|---|---|
| Action Intent Object UX has no template | Freya prototypes the approval card pattern before stories begin; specifically the partial-execution and re-verification flows |
| Hermes BYOK + multi-system dispatch security | Hermes spike 4 (BYOK 429 + credential handling) must complete before Action Intent stories begin; output sanitisation between Hermes and dispatch layer is mandatory |
| Vendor portal integration legal posture | Read-only access to vendor portals may be restricted by vendor contracts — confirm with ÖBB legal before Telematics warranty stories |
| Telematics surfaces under-spec at UX level | Scope is locked (§8) but UX detailing is in flight; Freya owns the Unified Telematics Control Panel wireframes and interaction spec before stories begin. PRD scope does not change; UX delivery sequencing does. |

---

## 8. Project Scoping & Feature Prioritisation

### Strategy & Philosophy

**Approach:** Compliance-first landside platform PoC — single release covering Depot Briefing + Fleet Manager + Telematics control board surfaces. Commercial pilot narrative leads with ECM proof point (Dr. Fischer sign-off), Fleet Manager second (Disponent decision latency), Telematics third (warranty triage on first Probebetrieb cycle).

**Resource Requirements:** One full-stack dev (React + Python/FastAPI), one AI/ML engineer (Hermes integration, prompt engineering, output sanitisation), one DevOps/infra (Docker, MQTT subscriber, PostgreSQL, pgvector).

---

### Complete Feature Set

**Must-Have Capabilities:**

| Capability | Rationale |
|---|---|
| MQTT subscriber for IntentPacket ingest | Foundation — nothing works without consuming brain output |
| Depot Briefing — ECM Manager pre-arrival brief | Primary commercial proof point for Dr. Fischer |
| Depot Briefing — ECM 1 blocking state + append-only audit trail | Regulatory compliance feature; closes the compliance sale |
| Depot Briefing — role-filtered technician feed (electrical/mechanical/IT) | Operational value; required for depot pilot |
| Fleet Manager — triage + cost-impact one-liner | Disponent KPI alignment; prevented delay-minutes counterfactual |
| Fleet Manager — Hermes BYOK + Rail MCP Server (9 tools) | Natural language query layer; differentiates from dashboard competitors |
| Fleet Manager — Action Intent Objects with one-click approval interlock | Core landside innovation — multi-system dispatch under single approval gate |
| Fleet Manager — shift handover flow | Required for multi-shift pilot |
| pgvector semantic search + keyword fallback | Spike gate: P99 <500ms under realistic incident volume before stories |
| ECM audit trail export (JSON) | Dr. Fischer's sign-off requirement |
| Multi-system dispatch with reconciliation queue | Action Intent Object partial-execution recovery |

**Telematics Unified Control Panel (MVP — locked scope):**

| Capability | Rationale |
|---|---|
| Zone 1: Macro situational awareness — fleet compatibility matrix (DOSTO Neu + Enzo — both new Stadler deliveries; pilot on DOSTO Neu, Enzo onboarded on rollout success) + geospatial telemetry overlay with dead-zone detection | Daily-use surface for Teil-Projektleitung; foundation for warranty + OTA evidence |
| Zone 2: Micro visual train topology — SVG anatomy + inline metadata drawers per hardware node (MAC, serial, firmware; sourced from IntentPackets + local registry in MVP; SAP asset registry post-MVP) | Required for warranty triage and digital-twin validation flows |
| Zone 3: Hermes AI Co-Pilot Console — chat + Proactive Solution Cards in Action Intent Object format | The action surface; the same Action Intent Object pattern as Fleet Manager |
| Cross-zone interaction binding (Zone 1 vehicle select → Zone 2 topology; Zone 1/2 fault → Zone 3 card) | Without this, the three zones are three disconnected pages |
| OTA rollout monitoring: canary uptime tracking + firmware/configuration drift detection (local registry in MVP; SAP asset registry post-MVP) | Use Case 4 — ÖBB monitors and approves rollouts; platform surfaces drift, not executes rollout |
| Warranty evidence packaging Action Intent Object | Use Case 5 — vendor leverage during Probebetrieb |
| Trial run validation (Probebetrieb): end-to-end network path display proving fault is landside vs onboard | Use Case 1 — commissioning sign-off accelerator |
| Digital Twin cross-entity validation after workshop hardware swap | Use Case 7 — prevents post-workshop misconfiguration reaching service |
| Lastenheft procurement clause export | *(Descoped — format, consumer, and failure threshold undefined. Moved to Vision for post-pilot consideration.)* |
| Per-alert-type accuracy metric display + Dismiss tuning loop (closed-loop trust recovery) | Use Case 10 — survives the first false-positive event; otherwise adoption collapses |
| Predictive edge alert → pre-emptive workshop reservation flow (via `reserve_maintenance_slot` MCP tool) | Use Case 3 — defining pre-emptive maintenance value proposition |

**Nice-to-Have Capabilities (in scope, lower priority):**

| Capability | Notes |
|---|---|
| Fleet Manager — 48–72h forward planning view | Drop first if resource-constrained |
| Structured JSON export formatting refinements | BOOM-ready but not BOOM-integrated |
| Dismissal signal arbitration queue | Conductor dismissals from brain feed into Fleet Manager threshold tuning; requires brain coordination |

**Spike Gates (must resolve before dependent stories begin):**

| Spike | Gate | Outcome if missed |
|---|---|---|
| pgvector P99 <500ms under realistic incident volume | Before Fleet Manager search stories | Keyword fallback only; semantic search deferred |
| Hermes BYOK 429 handling + credential isolation | Before Action Intent Object stories | Action Intent feature deferred to post-PoC |
| Action Intent Object dispatch reconciliation semantics | Before multi-system dispatch stories | Reduce scope to single-system dispatch only |
| SAP PM API contract for write-back | Post-MVP gate — SAP not in MVP scope | MVP ships HAFAS dispatch only; SAP work order write-back requires API contract with ÖBB TS |

---

### Risk Mitigation Strategy

**Technical risks:**
- pgvector P99 <500ms — spike story with keyword fallback confirmed before Fleet Manager search stories
- Hermes BYOK 429 + credential isolation — spike before any Action Intent dispatch story
- Action Intent dispatch atomicity — explicit acceptance criteria; reconciliation queue with idempotency keys
- Brain ingest gap — coordinate with brain team on WAL-before-truncate sync; landside surfaces staleness explicitly

**Market risks:**
- Dr. Fischer unavailable at pilot manday ~15 — audit trail still ships; sign-off milestone deferred
- Vendor portal access blocked — Telematics warranty story deferred or scoped to landside-only evidence

**Resource risks:**
- If team is constrained: defer Telematics surfaces (Gemini-derived); ship Depot Briefing + Fleet Manager core
- 48–72h forward planning view is nice-to-have, drop first

---

## 9. Open Architecture Decisions

The following decisions are deferred to the architecture phase. They are not PoC blockers but must be resolved before the first story in their respective epic begins. The architect owns these inputs.

| Decision | Context | Gate |
|---|---|---|
| Hermes intent disambiguation rules | FR identifying user intent and selecting Rail MCP tool. Rules-based, embedding-based, or LLM-classified is an architecture decision. | Before Fleet Manager Hermes stories |
| Action Intent Object dispatch reconciliation semantics | What constitutes "approved" vs "executed"; how partial failures are surfaced; idempotency key strategy across HAFAS (SAP post-MVP) | Before multi-system dispatch stories |
| MQTT broker identity (consumer side) | Subscriber host, auth model, multi-train topic subscription strategy | Owned by brain team — landside consumes spec |
| Schema evolution strategy (landside consumer) | When brain bumps IntentPacket schema, how landside handles forward/backward compatibility for in-flight packets | Post-PoC; must not block PoC |
| Telematics surface scope | ~~Which Gemini-derived surfaces are MVP vs post-MVP~~ — **resolved 2026-05-25:** full Unified Telematics Control Panel locked into MVP. See §8 capability table. | n/a — closed |
| Landside→brain command channel (`trigger_edge_diagnostics`) | **Resolved 2026-05-25:** Dropped from MVP. The brain emits IntentPackets on every state transition within 2s — there is no polling cadence to bypass. Landside staleness is caused by hardware/network gaps that a diagnostic ping cannot resolve, or by the brain not detecting a state change yet, which a ping also cannot resolve. Warranty triage (Journey 6) requires immutable timestamped chain-of-custody evidence — on-demand diagnostics at investigation time would weaken, not strengthen, vendor disputes. Landside remains a pure IntentPacket subscriber for v1. **Revisit trigger:** post-pilot telemetry shows >X% of Telematics or Fleet Manager decisions made on IntentPackets older than the brain's 5-minute pilot-kill-trigger threshold — specifics named after pilot baseline established. Any future landside-initiated tool requires a joint brain+landside ADR signed by both tech leads before scoping begins. | n/a — closed |
| Fleet Manager AI interface frontend wrapper | Browser app shell architecture (React layout, state management, routing) over Hermes agent backend | Before Fleet Manager UI stories |

---

## 10. Non-Functional Requirements

### Performance

| Requirement | Target | Context |
|---|---|---|
| pgvector semantic search | P99 < 500ms | Under realistic incident volume — spike gate before Fleet Manager search stories |
| IntentPacket landside ingest latency | < 2s from MQTT receipt to PostgreSQL commit | Excludes brain-side emission latency |
| Fleet Manager triage feed load | < 3s initial render | PostgreSQL query under PoC fleet size (~10–20 vehicles) |
| ECM sign-off commit | < 1s write acknowledgement | Landside PostgreSQL direct write |
| Rail MCP Server tool call timeout | 8s hard timeout | Hermes surfaces structured error; Fleet Manager decision logged with data-availability flag |
| Action Intent Object dispatch | < 5s per system; total < 10s for multi-system dispatch | Surfaced as in-flight state until all operations confirmed |
| Hermes response latency for query (BYOK) | P95 < 8s | Operator-owned LLM; depends on BYOK provider |

### Reliability

| Requirement | Target | Context |
|---|---|---|
| ECM audit trail integrity | Append-only, tamper-evident | No UPDATE or DELETE permitted; HMAC on every row; row-level trigger raises exception on modification attempt |
| Landside event delivery | No duplicate delivery after service restart | Idempotency on `packet_id` |
| Alembic migration write-lock ceiling | < 30s | Any migration exceeding this threshold requires an offline maintenance window; rollback path documented before merge |
| Action Intent dispatch reconciliation | All partial failures surfaced within 30s | Reconciliation queue polled; UI surfaces partial-execution state |
| Hermes BYOK retry policy | Exponential backoff, budget-capped | Configurable per operator; default 3 retries with 1s/4s/15s delays |

### Security

| Requirement | Target | Context |
|---|---|---|
| ECM sign-off identity | IdP-bound only | ÖBB SSO — no local session token accepted for compliance actions |
| Action Intent approval identity | IdP session re-verified at moment of approval | Not just at app load — fresh check before dispatch |
| BYOK API key storage | Never in source control, logs, or inference paths | Env vars (PoC), Docker secrets (fleet) |
| LLM output sanitisation | Validated schema before UI render | Between Hermes response and Fleet Manager render layer — prompt injection mitigation |
| Vendor portal credential isolation | Resource initialisation only — never inline in inference paths | Per-vendor credentials scoped to read-only operations |
| External data validation | Schema validation before any AI decision path | All IntentPackets, MCP tool inputs, vendor diagnostic feeds treated as untrusted |

### Scalability

| Requirement | Target | Context |
|---|---|---|
| PoC fleet size | 1–5 trains, 10–20 vehicles | Single ÖBB tenant, on-premise |
| Fleet rollout architecture | Per-operator deployment | Separate instances per operator — no shared-database multi-tenancy in initial fleet architecture |
| Hermes API quota | Operator-owned | BYOK model — operator owns API quota; platform does not throttle |
| PostgreSQL read concurrency | Migrations safe under concurrent reads | No table locks on reads during pilot operations |
| Telematics fleet matrix render | Smooth at 109 vehicles (full Cityjet DOSTO Neu fleet) | UI virtualization required at fleet rollout scale |

### Accessibility

| Requirement | Target | Context |
|---|---|---|
| Depot Briefing tap targets | Minimum 56dp | Mobile PWA — depot personnel may be wearing gloves |
| Fleet Manager keyboard navigation | Full keyboard access — no mouse-only interactions | Disponent works long shifts; keyboard primary |
| Contrast ratio | WCAG AA minimum across all surfaces | Depot lighting varies; Fleet Manager desk may be 12h shift |
| Critical action step ceiling | Maximum 2 deliberate steps (open flow + confirm) | ECM sign-off, Action Intent approval. Standard flows: max 3 steps. |
| Audio alerts (depot PWA) | Spec required — minimum dB threshold above depot ambient floor | Depot ambient noise floors polite notification sounds |

### Maintainability

| Requirement | Target | Context |
|---|---|---|
| IntentPacket schema (landside consumer) | Tracks brain-owned v1.0.0 | Schema evolution coordinated with brain team |
| Alembic migrations | Reversible, rollback path documented before merge | See reliability: write-lock ceiling 30s |
| Action Intent Object schema | **Schema of record** (2026-05-25): `{ intent_id: uuid, action_type: str, justification: str [mandatory], operations: [{ sequence: int, system: str, method: str, payload: {} }] }`. Versioned — `intent_id` includes schema version prefix. `justification` is mandatory; Hermes must not generate an object without it. `operations[]` are ordered by `sequence`; partial failure semantics respect ordering (earlier operations are not retried if a later one fails unless the reconciliation queue explicitly re-sequences). Schema evolution requires a joint landside ADR before any field addition or removal. | Schema evolution does not break in-flight intents |

---

## 11. Functional Requirements

### Depot Briefing & ECM Compliance

- FR1: ECM Manager can view a pre-arrival briefing for an inbound vehicle, generated from the vehicle's IntentPacket history. Pre-arrival window is triggered by HAFAS scheduled arrival time (via `get_hafas_timetable`) — the IntentPacket schema v1.0.0 carries no position field. Brief is considered "pre-arrival" when opened before the HAFAS scheduled arrival timestamp.
- FR2: ECM Manager can see all open faults on a vehicle, organised by technician discipline (electrical, mechanical, IT)
- FR3: ECM Manager can view a vehicle's blocking state when all technician faults are resolved but ECM 1 authorisation is pending
- FR4: ECM Manager can review a full fault resolution summary and sign off on a vehicle's return to service via a deliberate confirmation action
- FR5: ECM Manager can view a post-sign summary of exactly what was committed immediately after sign-off
- FR6: ECM Manager can correct a sign-off record by creating a superseding record, without modifying or deleting the original
- FR7: ECM Manager can export the full audit trail for a vehicle as structured JSON
- FR8: Technician can view a role-filtered fault feed showing only faults within their discipline
- FR9: Technician can acknowledge a fault with a structured ACK record. **[NEEDS DECISION before story begins: ACK schema fields, fault code vocabulary (fixed list vs free-entry), whether free-text notes are permitted, and whether a secondary confirmation step is required. Requires input from an ECM Manager or ECM 4 Technician — Abbas to arrange.]**
- FR10: System maintains an append-only sign-off event log with IdP-bound identity, vehicle state snapshot, directive reference, and tamper-evidence signature

### Fleet Manager Triage & Redeployment

- FR11: Fleet Manager can view a real-time triage feed of active fleet disruptions, with cost-impact one-liner per disruption
- FR12: Fleet Manager can assign a replacement vehicle to a disrupted service via an Action Intent Object
- FR13: Fleet Manager can view a staleness indicator on any fleet card where IntentPacket data is older than a threshold
- FR14: Fleet Manager can hand over their shift to an incoming Fleet Manager, with a snapshot of all open incidents and optional notes
- FR15: Fleet Manager can view an incoming shift handover summary at the top of the triage view on login
- FR16: Fleet Manager can review the conductor dismissal queue (from brain feed) and promote or discard individual dismissals as alert threshold tuning signals

### Fleet Manager AI Query (Hermes)

- FR17: Fleet Manager can query the fleet via natural language and receive a structured response grounded in live fleet data
- FR18: Fleet Manager can query predicted maintenance windows and fleet availability over a 48–72h horizon
- FR19: Fleet Manager can search past incidents using semantic similarity and receive ranked results
- FR20: Fleet Manager can see which search mode is active (semantic or keyword fallback) for any query result
- FR21: Query system identifies user intent and selects the appropriate data source to ground the response
- FR22: System surfaces a structured error when a Rail MCP Server tool call times out, with the Fleet Manager's decision logged alongside the data-availability flag

### Action Intent Objects

- FR23: Hermes can generate an Action Intent Object conforming to the schema of record (`intent_id`, `action_type`, `justification` [mandatory], `operations[]` with `sequence`/`system`/`method`/`payload`), bundling operations across MVP-integrated systems (HAFAS only in MVP). `justification` must be non-empty — schema validator rejects objects without it. SAP PM and ServiceNow operations are post-MVP — the schema validator rejects them with a clear "system not integrated in this release" error
- FR24: Fleet Manager can review the full operations list of an Action Intent Object before approval — every system touched and every payload visible
- FR25: Approval action re-verifies the user's SSO session at the moment of click — not just at app load
- FR26: Approved Action Intent Object is logged to `action_intent_audit` with `intent_id`, approver identity, timestamp, operations, dispatch outcome per operation
- FR27: Partial dispatch failures are surfaced as distinct "Partial execution — review" UI state with reconciliation guidance
- FR28: System dispatches Action Intent operations using idempotency keys to prevent duplicate write on retry

### Dispatcher Coordination *(Post-MVP — Dispatcher persona deferred from MVP scope)*

- FR29: *(Post-MVP)* Dispatcher can view a relay card with LLM-generated passenger-safe language summarising an incident
- FR30: *(Post-MVP)* Dispatcher can copy the relay card content to clipboard for PA system entry
- FR31: *(Post-MVP)* Dispatcher can forward the relay card to the conductor (push notification dispatched via brain)

### IntentPacket Ingest (Landside Consumer)

- FR32: System consumes IntentPackets from the brain's MQTT broker on `oebb/{train_id}/intent` topic
- FR33: System persists ingested packets to PostgreSQL with idempotent insert on `packet_id`
- FR34: System surfaces ingest gap explicitly — Fleet Manager and Depot Briefing surfaces show staleness when no fresh packet has arrived
- FR35: System handles unknown `agent_state` values (from future brain schema versions) as raw events rather than silently discarding

### Telematics Project Management — Unified Control Panel (MVP)

**Zone 1 — Macro Situational Awareness**
- FR36: Teil-Projektleitung Telematik can view a fleet compatibility matrix mapping software versions against physical vehicle IDs (Triebfahrzeugnummern). MVP pilot covers DOSTO Neu; the matrix is designed to onboard Cityjet Enzo (a separate, newer Stadler programme) on rollout success without schema changes. Matrix entries distinguish "active — telemetry live", "active — telemetry gap" (hardware or connectivity fault), and "not yet onboarded" (Enzo fleet prior to programme rollout). Acceptance criterion: a reviewer can distinguish a telemetry gap from an un-onboarded vehicle without additional lookup
- FR57: When Cityjet Enzo units are onboarded in a subsequent rollout phase, the system ingests their IntentPackets via the same ingest path as DOSTO Neu. The compatibility matrix validates that Enzo units on the same software version produce equivalent diagnostic coverage to DOSTO Neu; any coverage differences (due to programme-specific sensor configurations) are surfaced as "coverage delta" metadata on affected matrix entries, not as data errors. This FR is post-pilot; it is listed here to ensure the DOSTO Neu matrix schema does not require breaking changes at Enzo onboarding
- FR37: Teil-Projektleitung Telematik can view a geospatial telemetry overlay highlighting recurring connectivity dropouts by location; when multiple trains drop at the same coordinates, the overlay classifies the cause as a regional cellular dead zone rather than a per-vehicle hardware fault
- FR48: System distinguishes stale-data-due-to-geographic-blackout from stale-data-due-to-hardware-fault by correlating the staleness event against the train's GPS position against the dead-zone overlay; staleness indicators are labelled with the disambiguated cause

**Zone 2 — Micro Visual Train Topology**
- FR38: Teil-Projektleitung Telematik can drill into an SVG train topology for a vehicle, with inline metadata drawers per hardware node (MAC address, serial number, firmware version, switch port mapping). **MVP: metadata sourced from IntentPackets and local platform registry. SAP asset registry integration is post-MVP.**
- FR49: Teil-Projektleitung Telematik can trigger an on-demand Digital Twin validation after a workshop hardware swap: the brain retrieves current hardware state from the onboard Stadler diagnostic system and sends it to landside as an IntentPacket; landside compares the received hardware state against the vehicle's expected schematic (local platform registry) and surfaces configuration discrepancies in the topology view before the vehicle is cleared to leave the workshop.
- FR54: Topology nodes degrade gracefully under partial node failure: affected nodes grey out with a "Stale Data" badge showing elapsed time since last successful handshake; the topology view does not display blank state or trigger false red-alerts

**Zone 3 — Hermes Console + Cross-Zone Interaction**
- FR50: Selecting a vehicle in Zone 1 (matrix or geospatial overlay) opens that vehicle's topology in Zone 2; a fault detected in Zone 1 or Zone 2 generates a Proactive Solution Card in Zone 3 in Action Intent Object format
- FR51: Teil-Projektleitung Telematik can view Zone 1, Zone 2, and Zone 3 in a single workspace (the Unified Telematics Control Panel) — no full-screen mode switching required to retain context across zones

**Telematics action flows**
- FR39: Teil-Projektleitung Telematik can monitor OTA rollout progress across the fleet: the platform displays the firmware and software version last reported by each unit, sourced from the most recent IntentPacket received from the brain (which retrieves version state from the onboard Stadler diagnostic system). Lukas can see at a glance which units have confirmed the new patch level, which are on a prior version, and which have not reported in (with last-seen timestamp). ÖBB monitors; rollout execution is performed by Nomad Rail operations or the manufacturer.
- FR40: System detects configuration drift by cross-referencing reported firmware/hardware revisions against the platform's local asset registry; incompatible or drifted units are surfaced for manual review. **MVP: local registry, not SAP asset registry (SAP integration post-MVP).**
- FR41: Teil-Projektleitung Telematik can generate a warranty evidence package as an Action Intent Object — Hermes correlates onboard IntentPacket history + landside service telemetry + local asset registry metadata into a structured PDF export attachable to vendor tickets. *(SAP asset metadata cross-reference is post-MVP.)*
- FR52: Teil-Projektleitung Telematik can view a Probebetrieb (trial run) validation surface during the 2-week passenger trial of a newly delivered train: system displays end-to-end network path for each detected fault, proving whether the issue is inside an ÖBB landside server or the manufacturer's onboard hardware
- FR53: Teil-Projektleitung Telematik can convert a predictive edge alert into a pre-emptive workshop reservation via the `reserve_maintenance_slot` MCP tool — the system books the next compatible workshop slot before the vehicle reaches a failure state
- FR55: *(Removed — Lastenheft procurement clause export descoped. Format, consumer, and failure threshold were undefined and the feature cannot be evidenced during the pilot. Moved to Vision if reinstated post-pilot.)*
- FR56: Teil-Projektleitung Telematik can view a per-alert-type accuracy metric on every alert ("This alert type has X% accuracy on this route, Y false positives in Z days"); a high-visibility neutral "Dismiss" action converts user dismissals into tuning signals for the system's threshold model, with the dismissal recorded server-side per FR44

### Identity, Access & Audit

- FR42: All safety-critical and compliance actions can be bound to an ÖBB SSO IdP identity, not a local session
- FR43: Each platform surface enforces role-based access. For controls a user's role cannot execute: the control is **rendered but visually disabled** with a tooltip explaining the permission gap (e.g. "Approval requires Fleet Manager role") — it is not hidden. Rationale: in safety-critical ops, operators need to know the full action space to make correct escalation decisions. Server-side authorization on the approval endpoint is the binding enforcement layer; the disabled UI state is defense-in-depth, not the security boundary.
- FR44: System records every acknowledgement, sign-off, dismissal, Action Intent approval, and status change as a server-side write to PostgreSQL, not component state
- FR58: System emits a structured security incident event to ÖBB's security monitoring pipeline for each of the following: authentication anomaly (failed IdP verification on a safety-critical action), unauthorised sign-off attempt (role mismatch on ECM or Action Intent approval), BYOK key exposure detection (key appears in any logged field), LLM injection detection (output sanitisation layer rejects a Hermes response). Event payload includes: event type, surface, user identity (if known), timestamp, severity. This is the platform's NIS2 contribution — the regulatory 24h filing is ÖBB IT Security's responsibility, not the platform's

### Resilience & Consistency

- FR45: System provides a defined fallback state when the brain ingest pipeline is unavailable — last-known values displayed with explicit unavailability label
- FR46: System maintains write consistency for cross-boundary operations — partial failures are logged and surfaced for reconciliation, never silently dropped
- FR47: Fleet Manager can view delay-impact attribution for redeployment decisions — actual vs. projected delay-minutes traceable to the Fleet Manager action

---

*This is the binding capability contract — 58 FRs across 8 capability areas. Any feature not listed here will not exist in the final product unless explicitly added.*

*Onboard brain capabilities (Conductor App, Hailo inference, IntentPacket emission, departure clearance, shift-aware density, edge resilience) live in `oebb-brain/prd.md` — see `oebb-brain/` directory for the companion PRD.*
