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
  complexity: High — multi-system integration (HAFAS, vendor diagnostic portals) + Kondukt BYOK agent runtime + pgvector semantic search + append-only ECM audit. MVP is advisory-only — no automated dispatch. Action execution remains with the operator's existing systems (HAFAS, paper sign-off, radio). Phase 2 (post-pilot, gated on trust criteria) re-introduces dispatch.
  projectContext: Greenfield landside platform; receives IntentPacket stream from oebb-brain (onboard) and from vendor telemetry sources. SAP/ServiceNow/BOOM integrations are post-MVP.
  deploymentModel: Single ÖBB instance (PoC); per-operator deployment at fleet rollout
  aiPosture: Advisory-only in MVP — Kondukt produces structured Recommended Action Objects; human reads and acts via existing operational systems; platform records the decision as a shadow audit trail for trust-building. Phase 2 (post-pilot) introduces platform-mediated dispatch with retained authoritative audit recording, gated on Phase 1 trust criteria.
  mvpPhase: advisory-only
  phase2Trigger: trust-criteria-met-at-pilot-end
---

# Product Requirements Document — ÖBB Landside Platform

**Author:** AbbasRizvi
**Date:** 2026-05-23 (split into landside-only on 2026-05-25)
**Companion document:** `oebb-brain/prd.md` — onboard brain PRD (Conductor App, Hailo inference, IntentPacket schema)

---

## Scope Boundary

This PRD covers the **landside platform** — the web and mobile surfaces used by ÖBB's fleet operations, technical services, and telematics project management teams. The onboard "brain" (Hailo-10H inference, Conductor App PWA, IntentPacket schema, edge resilience) is a separate product documented in `oebb-brain/prd.md`.

The landside platform **consumes** IntentPackets from the brain over MQTT and persists them. The schema is owned by the brain project. Landside is a downstream subscriber, alongside HAFAS timetable data and Stadler vendor portal feeds. SAP PM, ServiceNow, and BOOM are post-MVP integrations.

**MVP execution model.** The landside platform is advisory-only throughout the 6-month pilot. It produces recommendations and prepares operators for decisions; it does not dispatch commands to operational systems (HAFAS, SAP, BOOM) or record authoritative ECM sign-offs. Operators continue to act via their existing systems and processes (paper sign-off, HAFAS direct, radio confirmation) and the platform shadows those actions as an audit-trail capability demonstration. Phase 2 — platform-mediated dispatch and authoritative recording — is gated on pilot-end trust criteria and is a separate commercial release.

---

## Executive Summary

ÖBB's landside operations — Disponent (Fleet Manager), ECM Manager (regulatory authorisation), Technicians (ECM 4), and Teil-Projektleitung Telematik (Telematics Sub-Project Manager) — currently make high-stakes decisions across fragmented systems: vendor diagnostic portals, Power BI dashboards, ServiceNow tickets, vendor work orders, Microsoft Project schedules, and Excel reconciliation files. Each switch costs cognitive load; each gap is a place a fault goes unattributed. The platform's MVP closes the *information* gap — it does not yet close the *execution* gap. That comes in Phase 2, after trust is established.

The platform unifies these surfaces around three roles:

**Live decision support — Fleet Manager** *(primary persona — Roland Ruisz, ÖBB operations)* — the Disponent receives a real-time triage feed of fleet disruptions, each tagged with a cost-impact one-liner. Kondukt (operator-owned BYOK LLM) generates Recommended Action Objects (RAOs) — structured proposals describing what should happen across multiple systems (HAFAS rotation update + work order creation), rendered as readable cards with step-by-step execution guidance. The Disponent executes via their existing systems (HAFAS direct, SAP direct); the platform records the action they took and the outcome as a shadow audit entry.

**Pre-arrival compliance — ECM Manager** — the ECM 1 Manager receives a structured fault brief generated from the vehicle's IntentPacket history before docking. They sign off **on paper** per existing ÖBB process; the platform records their decision alongside the paper trail as a shadow audit entry — building the evidence that the platform's audit-trail capability is sound before Phase 2 promotes it to the authoritative record.

**Commissioning & lifecycle — Teil-Projektleitung Telematik** *(primary persona — Roland Ruisz, ÖBB operations; same person as Fleet Manager)* — the Telematics Sub-Project Manager monitors trial runs (Probebetrieb) for newly delivered fleets, validates configuration alignment across the Stadler delivery programme (DOSTO Neu pilot, Cityjet Enzo on rollout expansion), monitors phased OTA software rollouts and configuration/firmware drift, and triages root-cause warranty disputes against vendors using end-to-end network path evidence. Warranty evidence packaging produces a structured export the Telematics Sub-PM submits manually to vendor ticketing systems; the platform does not file warranty claims automatically. This persona was identified in the Gemini research pack (May 2026) and is the daily user of the landside platform's macro situational awareness zone.

Every decision is made by a human and executed via the operator's existing systems. The platform records the decision and its outcome as shadow audit evidence — proving the platform's capability to be the authoritative record once trust is established.

### What Makes This Special

**Prevented delay-minutes counterfactual** — measurable, ÖBB-validated KPI the Disponent is already accountable for. The platform makes this number visible. Cloud-dependent competitors cannot produce this because they have no edge inference layer producing structured event evidence.

**ECM audit trail** — not a log, but the regulatory compliance record under EU 2019/779. Append-only, IdP-bound, HMAC-signed, hash-chained, with `supersedes_id` for corrections. In MVP, the trail shadows the paper sign-off process; in Phase 2, it becomes the authoritative regulatory compliance record under EU 2019/779.

**Kondukt Recommended Action Objects (RAOs)** — the landside differentiator. Where competitor dashboards show data and require humans to translate it into actions across 4–7 systems, this platform's Kondukt co-pilot produces structured multi-system action proposals as readable recommendation cards with step-by-step execution guidance. The Disponent reviews the recommendation, executes via existing systems, and logs the action taken back into the platform — building the recommendation→action→outcome dataset that gates Phase 2 dispatch capability.

**Vendor warranty leverage** — the platform's end-to-end network path visibility resolves "is it our landside or the manufacturer's hardware" disputes during the Probebetrieb commissioning phase, where a single telematics breakdown can freeze formal sign-off. This is a non-obvious commercial wedge for the Telematics Sub-Project Manager persona.

### Pilot Success Criteria

| Criterion | Target | Evidence method |
|---|---|---|
| ECM pre-arrival review | >80% of arrivals | IntentPacket generated time vs HAFAS scheduled arrival time vs ECM brief opened time. Sign-off itself is recorded on paper; the platform's shadow record is verified against the paper trail post-shift. **Pre-arrival trigger = HAFAS scheduled arrival (MVP default — IntentPacket schema v1.0.0 carries no GPS/position field). Brain workspace to confirm if a position field is planned for a future schema version.** |
| Disponent action latency reduction | Measurable reduction vs baseline | Instrument time-to-decision for redeployment calls; baseline early pilot mandays, measured later pilot mandays |
| Dispatcher coordination | Deferred — Dispatcher persona removed from MVP scope |
| Recommendation acceptance rate | ≥ Roland-confirmed threshold (working assumption ≥70%) within first 30 pilot mandays | Acceptance + modification + rejection rates catalogued per RAO; modification rate signals recommendations close-but-not-yet-trusted-to-dispatch |
| Shadow audit-trail match rate | 100% of paper sign-offs during pilot have matching platform shadow record | Field-for-field reconciliation within 24h of paper signature; divergences flagged and tracked |
| Phase 2 trust threshold met | Composite of acceptance rate + 100% shadow-match + Renate Fischer sign-off + outside-auditor tabletop verdict + zero unresolved pilot-kill triggers in final 30 days | All five required for full Phase 2; partial satisfaction triggers narrower Phase 2 scope |
| Phase 2 readiness recommendation | Within 6 months of pilot start, with Phase 2 readiness assessment generated automatically from the trust-criteria composite | Composite pilot health score AND Phase 2 trust-criteria status updated weekly with visible threshold lines for both |

### Project Classification

- **Project Type:** Landside fleet operations and telematics platform — web dashboard, mobile PWA, AI agent interface
- **Domain:** Rail operations — regulated landside (EU ECM Directive 2019/779, NIS2, GDPR)
- **Complexity:** High — multi-system integration + Kondukt BYOK + pgvector + append-only audit
- **Project Context:** Greenfield landside; consumes IntentPackets from oebb-brain
- **Deployment Model:** Single ÖBB instance (PoC); per-operator deployment at fleet rollout
- **AI Posture:** Advisory-only in MVP — Kondukt produces structured Recommended Action Objects; human reads and acts via existing operational systems; platform records the decision as a shadow audit trail for trust-building. Phase 2 (post-pilot) introduces platform-mediated dispatch with retained authoritative audit recording, gated on Phase 1 trust criteria.

---

## Success Criteria

### User Success

| Persona | Success moment | Measurable signal |
|---|---|---|
| Fleet Manager (Disponent) | Makes a redeployment decision with cost consequence visible before confirming | Time-to-decision for redeployment recommendations, measured before/after. Includes time-from-platform-recommendation to operator-action-logged-in-platform, not platform-dispatch time. |
| Dispatcher (Fahrdienstleiter) | Post-MVP — persona deferred from MVP scope | — |
| ECM Manager | Arrives at depot having already reviewed the fault brief; signs on paper per existing process; verifies that the platform's shadow record matches the paper sign-off before leaving the bay | >80% of arrivals with IntentPacket opened before vehicle arrives; 100% shadow-trail match against paper |
| Technician (ECM 4) | Logs a resolution ACK that appears in Klaus's pre-arrival brief before the vehicle docks — no phone call (paper handover still required per existing depot process; platform record runs alongside) | Fault reassignment rate per role per vehicle; % of faults with ACK logged in platform before vehicle arrival (shadow signal, not authoritative); % of platform shadow ACKs matching corresponding paper records |
| Teil-Projektleitung Telematik (Roland Ruisz) | Resolves a vendor warranty dispute using platform evidence in a single session | Time-from-dispute-raised to evidence-packaged; reduction in vendor escalation cycles |

### Business Success

- **Prevented delay-minutes:** Faults detected before service disruption, cross-referenced against historical delay data for same fault class on unmonitored units. Target: produce counterfactual evidence for at least 7 incidents in pilot period.
- **ECM audit trail completeness:** Sign-off timestamp, briefing-read timestamp, and fault resolution summary present for >95% of vehicle returns to service during pilot.
- **Shadow audit-trail completeness:** 100% of paper ECM sign-offs during the pilot have a corresponding platform shadow record. Mismatches surface within 24h of the paper sign-off and are reconciled before pilot manday close.
- **Fleet Manager decision latency reduction:** Measurable reduction in time-to-decision for redeployment calls compared to baseline (pre-platform). Latency measured from disruption-detected to operator-action-logged. The platform does not itself execute the action. Baseline measured during early pilot mandays, post-platform measured in later pilot mandays (see Pilot calendar anchor).
- **Phase 2 readiness recommendation:** Composite trust-criteria score crosses threshold within 6 months. Threshold composition: recommendation acceptance rate ≥X% AND shadow audit-trail match rate = 100% AND Renate-signed audit-export quality assessment AND outside-auditor tabletop verdict. Presented to Roland Ruisz, Martin Lerch, and Renate Fischer.

### Technical Success

- pgvector semantic search P99 < 500ms under realistic incident volume (internal target <250ms at 50K vectors with ef_search=64) — spike gate before Fleet Manager search stories
- Shadow audit-trail write < 1s acknowledgement; zero UPDATE/DELETE events on `ecm_signoff_events`; reconciliation against paper trail completes within the same shift
- Fleet Manager triage feed < 3s initial render under PoC fleet size (10–20 vehicles)
- Rail MCP Server tool call timeout 8s hard; structured error surfaced with data-availability flag
- IntentPacket landside ingest: idempotent insert on `packet_id`; no duplicate delivery after service restart

### Pilot-Kill Triggers

The following conditions pause the pilot pending investigation. Roland Ruisz (operations) and Martin Lerch (Technical Services) must be notified within 24h of any trigger firing.

| Trigger | Threshold | Action |
|---|---|---|
| ECM shadow audit write failure | Unresolved >24h | Halt platform-side shadow recording; ECM paper process continues unaffected; flag to Martin Lerch for infrastructure review |
| Shadow audit-trail divergence | >2 mismatches per pilot week between paper and platform records | Platform-side root-cause investigation; if divergence persists, Phase 2 readiness assessment downgrades |
| pgvector semantic search unavailable | Keyword fallback active >48h continuous | Flag to Martin Lerch; infrastructure review required |
| Kondukt BYOK API unavailable | >24h sustained 429 / 5xx | Disable Recommended Action Object generation; manual triage only |
| Rail MCP Server tool failure | Any single tool unavailable >12h | Surface "Data source unavailable" to Fleet Manager; review by Nomad ops |
| Landside ingestion gap from brain | >72h without IntentPacket sync on any active train | Coordinate with brain team — root cause may be onboard or transport |

### Measurable Outcomes (Pilot Evidence Targets)

| Evidence | Method | Target |
|---|---|---|
| ECM trail completeness | Log IntentPacket generated time vs HAFAS scheduled arrival time vs ECM open time vs sign-off time. Pre-arrival = ECM brief opened before HAFAS scheduled arrival. | >80% pre-arrival review rate |
| Fleet Manager decision latency | Instrument time-to-decision for redeployment calls before/after | Measurable reduction |
| Audit trail usage correlation | Sign-off timestamp vs briefing-read timestamp — did reading the brief change the outcome? | Qualitative + quantitative |
| Recommended Action Object acceptance rate | Acceptance action log per intent_id, segmented by `action_type`; modification rate also tracked — a RAO that is partially followed signals the recommendation engine is close but not yet trusted to dispatch | >70% acceptance, rejection reasons catalogued |
| Recommendation→action→outcome chain | Capture every RAO with: generated_at, displayed_to_user, user_action_taken (executed/modified/rejected/ignored), action_logged_at, outcome_observed_at | This is the dataset Phase 2 trust evaluation depends on |

**Pilot calendar anchor:** Pilot manday 1: **end of August 2026 (target) or September 2026 (slip-safe target).** Hailo hardware delivery is the gating dependency. Pilot schedule is tracked in **mandays**, not calendar days. Type Approval achieved April 21, 2026; pre-pilot critical path (Apr–Aug 2026) includes whole-application build and Nomad Rail sole-proprietary company formation. "Baseline mandays" = first ~10 mandays of active platform use; "measurement mandays" = mandays 20–30. Roland Ruisz sign-off meeting targeted around pilot manday 15 (mid-Sept / mid-Oct 2026). Phase 2 readiness recommendation within 6 months of pilot manday 1 (Feb/Mar 2027). **Pre-pilot gate:** confirm Hailo-8 and Hailo-10 order placed and delivery date confirmed before pilot manday 1 is set. **Probebetrieb dependency (Journey 6 risk):** Journey 6 (Vendor Warranty Triage) requires the platform to be active during an active Probebetrieb commissioning cycle. Pre-pilot gate: confirm which Probebetrieb cycles are active during the pilot window with Lukas's programme team. If no cycle overlaps, Journey 6 is a post-pilot story and the pilot evidence dashboard drops the warranty triage claim.

**Baseline methodology (prevented delay-minutes) — approximation approach:** A precise historical baseline from ÖBB's Power BI Diagnoseplattform or equivalent is not expected to be available for the pilot. The prevented delay-minutes figure will be calculated as an approximation: fault detected by platform → no service disruption → cross-reference against industry-standard or ÖBB-acknowledged mean delay for the same fault class. Roland Ruisz to agree on the approximation methodology and the fault-class reference values before pilot manday 1. This is a best-effort evidence narrative for the ROI conversation, not a statistically rigorous baseline — the goal is a credible order-of-magnitude figure, not audit-grade precision.

**ECM completeness gap (80% → 95%):** The 80% pre-arrival review target is platform-delivered. The 95% business success target requires process change (ECM Manager opens brief before leaving for depot, not on arrival). Martin Lerch owns the process workstream. **Phase 1 does not change the paper sign-off process; Phase 2 replaces it.**

---

## Product Scope

### MVP — Minimum Viable Product

Three landside surfaces, delivered in priority order:

1. **Depot Briefing Interface** — PWA, mobile-first. Role-filtered fault feed (ECM Manager / Electrical / Mechanical / IT), ECM 1 blocking authorisation state, append-only sign-off with IdP-bound identity, pre-arrival brief generated from IntentPackets. BOOM integration descoped — structured export/copy for manual BOOM entry.

2. **Fleet Manager AI Interface** — The primary landside control surface. Kondukt BYOK, Rail MCP Server (9 tools), three entry states (triage 70/30, crisis full-width, trend 30/70), pgvector semantic search over incident summaries, cost-impact one-liner per triage card, shift handover summary, relay card for Dispatcher forwarding, Kondukt-generated Recommended Action Object cards with three-tier confidence visual grammar.

3. **Telematics Project Management surfaces** *(from Gemini research)* — the **ÖBB Unified Telematics Control Panel**: a single workspace presenting three linked zones to the Teil-Projektleitung Telematik persona. Zone 1 (Macro Situational Awareness): fleet compatibility matrix + geospatial telemetry overlay. Zone 2 (Micro Visual Train Topology): SVG train anatomy + inline metadata drawers per hardware node. Zone 3 (Kondukt AI Co-Pilot Console): chat + Proactive Recommendation Cards in Recommended Action Object format. Zones are bound by interaction: selecting a vehicle in Zone 1 opens its topology in Zone 2; a fault detected in Zone 1 or 2 generates a Proactive Recommendation Card in Zone 3. Use cases covered: trial run validation (Probebetrieb), cross-fleet modernization monitoring (DOSTO Neu pilot; Cityjet Enzo onboarded on rollout success — both are new Stadler deliveries, not retrofits), OTA software rollout with canary + drift detection, root-cause warranty triage, digital twin / cross-entity validation after workshop hardware swaps, multi-vehicle procurement spec export (Lastenheft), closed-loop trust recovery with per-alert-type accuracy metrics, ROI analysis. **Scope locked as MVP** — FR36–FR50 below are the binding contract for this surface.

**Infrastructure MVP:**
- `rail-mcp-server/` — 9-tool HTTP MCP endpoint (spec frozen — see `rail-mcp-server-spec-2026-05-23.md`)
- `ecm_signoff_events` + `shift_handovers` tables in PostgreSQL
- 4 Kondukt spikes run and findings documented (pre-story gate)
- pgvector fallback ranking strategy defined and spiked before Fleet Manager stories
- IntentPacket landside ingest endpoint — REST + SSE from oebb-brain MQTT broker, idempotent on `packet_id`

### Growth Features (Post-MVP)

- Fleet Manager 48–72h forward planning view alongside real-time triage
- Shift handover cross-session AI conversation context
- BOOM API integration when API contract confirmed with ÖBB IT
- ServiceNow ITSM bidirectional integration (ticket creation from Fleet Manager triage)
- SAP PM deeper integration — automatic G-Check work order creation from Recommended Action Objects (Phase 2 expansion)
- Multi-tenant deployment — credential isolation, per-operator Rail MCP Server instances
- Dismissal signal arbitration queue (conductor dismissals from brain feed into Fleet Manager threshold tuning loop)

### Vision (Future)

- Kondukt agents that proactively surface fleet patterns without query — push-based fleet intelligence
- ECM audit trail integration with ÖBB's internal compliance documentation system
- Lastenheft procurement requirements export — clauses generated from cumulative operational evidence (descoped from MVP; format, consumer, and failure threshold to be defined post-pilot)
- Multi-operator rail network deployment (non-ÖBB carriers)
- Configuration drift auto-remediation — Kondukt proposes patch deployment to fleet trains showing drift

---

## Disaster Recovery (Landside)

| Failure | Behaviour | Recovery |
|---|---|---|
| PostgreSQL unavailable | Shadow audit-trail writes queue locally at depot, sync on reconnect. Paper sign-off process continues normally — platform unavailability does not block ECM clearance. Fleet Manager AI Interface shows stale data with timestamp. Rail MCP Server returns 503 with `recoverable: true`. | Depot assumed to have fixed network — extended outage triggers ops alert. Shadow audit-trail writes resume on reconnect; append-only guarantee preserved. |
| pgvector index failure | Fleet Manager semantic search degrades to keyword fallback ranking. No silent failure — UI labels results "Keyword search active". | pgvector restart clears index; re-index from `incident_summary` embeddings. Spike required pre-Fleet Manager stories to define fallback behaviour and re-index SLA. |
| Kondukt BYOK API unavailable (429 / 5xx) | Fleet Manager AI chat shows "Fleet intelligence temporarily unavailable". Triage and crisis views remain fully functional — they do not depend on Kondukt. Retry with exponential backoff + budget cap. | BYOK 429 handling is Kondukt spike 4. |
| Rail MCP Server tool failure | Kondukt receives structured error `{error, detail, recoverable}`. Recoverable errors retried once. Non-recoverable surfaced to Fleet Manager as "Data source unavailable for [tool name]". | Each tool is independently recoverable. |
| Brain → landside ingest gap | IntentPacket sync paused. MQTT retain=True surfaces only the last-known packet on reconnect — gap-period packets are not on the MQTT broker. Fleet Manager and Telematics surfaces show staleness indicator on affected vehicles based on sequence-number or heartbeat detection (FR45). Audit trail gap flagged with last-known timestamp. | Brain replays unacknowledged packets via its dedicated replay channel on reconnect; landside acknowledges per `packet_id` (idempotent insert). Replay-channel spec is brain-owned. Until replay completes, affected surfaces remain in staleness state — no false-fresh data. |
| Recommended Action Object — outcome reconciliation gap | If the operator logs that they executed an RAO but the platform cannot verify the outcome via downstream system polling (HAFAS-state check, SAP-state check), platform marks the RAO outcome as 'self-reported unverified' in the shadow trail. Disponent surfaces this and provides clarification. | Subsequent polling cycles attempt reconciliation; persistent unverified outcomes flagged for Phase 2 readiness review. |

### Shadow Audit Trail Integrity (MVP) / ECM Audit Trail Integrity (Phase 2)

- `ecm_signoff_events` is the platform's shadow record of paper sign-offs in MVP; it becomes the authoritative ECM record in Phase 2. Append-only enforcement is identical in both phases — enforced at database level (row-level trigger raises exception on UPDATE/DELETE attempt)
- Corrections are new rows with `supersedes_id` pointing to corrected record — no in-place mutation ever
- `signature` field (HMAC) provides tamper evidence for each row; hash chain links each row to previous (`sha256(prev_row_hash || row_hmac)`) proving non-modification AND non-deletion across the trail
- Extended connectivity loss: shadow audit writes queue locally at depot, sync to landside PostgreSQL on reconnect. Paper sign-off proceeds unaffected. Queue is durable — depot device loss before sync is an unrecoverable shadow-trail gap, flagged as missing record with last-known timestamp; paper trail remains the source of truth in MVP regardless
- **Sign-off gesture in MVP** — the gesture described in `design-artifacts/ecm-signoff-interaction-spec.md` records the *platform's shadow entry* matching the operator's just-completed paper sign-off. 3-second hold-to-initiate → 5-second countdown → single atomic write at expiry. No server-side pending state — the write fires exactly once, at countdown expiry. Cancellation at any point during countdown prevents write. After write, record is permanent — no undo. Idempotency key generated at countdown initiation; retries with the same key are safe (409 Conflict treated as success). The 3-second-hold + 5-second-countdown gesture is preserved as the trust-building muscle memory for Phase 2 — operators learn the gesture under low-stakes conditions where the paper trail is the safety net. In Phase 2 the same gesture writes the authoritative record.

---

## User Journeys

### Journey 1 — ECM Manager: "The Pre-Arrival Brief" (Primary — Success Path)

**Meet Klaus.** Klaus is an ECM 1 Manager at Wien Westbahnhof depot. He is accountable under EU Directive 2019/779 for every vehicle that returns to service. He arrives cold — he was not on shift when the train came in last night. Unit 4722 is in Bay 3.

**Before the platform.** Klaus walked to Bay 3, asked the technician what happened, read a paper fault log, and made a judgment call. If he missed something and the vehicle failed in service, the liability was his and the paper trail was thin.

**Now.** At 05:50, fourteen minutes before Unit 4722 docks, the landside platform generates a depot briefing from the journey's IntentPackets (received from oebb-brain over MQTT). Klaus opens the Depot Briefing Interface on his phone. Role: ECM Manager. He sees: two faults sorted by severity. CRITICAL: Door motor fault, Car 4 Door 2L — resolved by conductor at 07:14, replacement part fitted at 03:22 by Electrical team. WARNING: SNMP camera firmware version mismatch on VLAN 5 — acknowledged by IT technician, deferred to next maintenance window with reason logged.

**The climax.** Klaus reviews the resolution chain on the platform — detection → conductor ACK → technician assignment → part fitted → test result. He walks to Bay 3 and confirms what he has read with the technician on site. He signs off **on paper** per ÖBB's existing ECM 1 process. Then he taps "Record sign-off" in the platform — the same 3-second-hold gesture, the same 5-second countdown — and the `ecm_signoff_events` shadow record writes: his IdP identity, UTC timestamp, full vehicle state snapshot at sign-off, directive reference EU 2019/779. The paper is the sign-off; the platform's record is the shadow.

**The new reality.** Unit 4722 clears to service at 06:08. Klaus has the paper signed and a digital shadow that proves — if the paper were ever lost or disputed — what he actually authorised. Over the 6-month pilot Klaus and his peers build a dataset of paper-and-shadow pairs that proves the platform's audit-trail capability is sound. That is the evidence Phase 2 trust criteria rests on.

*Capabilities revealed: Pre-arrival brief generation, role-filtered fault feed, ECM 1 blocking state, hold-to-record gesture, append-only shadow sign-off, full resolution chain view, paper-trail reconciliation.*

---

### Journey 2 — Fleet Manager: "The Crisis Redeployment" (Operations — Success Path)

**Meet Stefan.** Stefan is a Fleet Manager (Disponent) at ÖBB Wien operations. He is responsible for cost management through fleet deployment across a 12-hour shift. It is 07:23. He opens the Fleet Manager AI Interface.

**The modal fires.** "Unit 4722 — Door Motor Fault, Car 4. Active unresolved incident. View?" He taps yes. Full-width crisis view. AI chat pre-seeded: "Unit 4722 door motor fault detected at 06:14. Conductor resolved manually at 07:14. Vehicle cleared by ECM at 06:08. Replacement unit available: Unit 4819."

**Stefan queries.** "What does deploying 4819 cost versus keeping 4722 on route?" Kondukt calls `get_maintenance_cost_estimate(unit_id="4722", replacement_unit_id="4819")`. Response in 2.1 seconds: "Replacing with Unit 4819 adds ~€2,400 estimated maintenance overhead. Confidence: medium. Rationale: 4819 last serviced 34 days ago, approaching mileage threshold." The cost one-liner was already in the triage card before he typed the question.

**Recommended Action Object.** Stefan asks: "What does it take to redeploy 4819 to take 4722's slot on the next rotation?" Kondukt generates a Recommended Action Object: `{action_type: "FLEET_REDEPLOYMENT", operations: [{system: HAFAS, method: rotation_update, payload: {rotation_id: ..., effective_at: '22:15', replacement_unit: '4819'}}], justification: "..."}`. Rendered as a readable recommendation card with the exact HAFAS payload Stefan needs to enter, a step-by-step procedure ("1. Open HAFAS, 2. Navigate to Rotation 22:15, 3. Replace 4722 with 4819, 4. Confirm with conductor by radio"), and a "Mark executed" button. *(SAP work order routing is post-MVP. In MVP the platform does not dispatch — Stefan executes via HAFAS himself.)*

**The decision.** Stefan reviews the recommendation. The HAFAS payload matches his judgment. He opens HAFAS in his other tab, executes the rotation update himself, confirms with the conductor by radio, then returns to the platform and taps "Mark executed — confirmed via HAFAS at 07:24". The platform records: the RAO that was generated, Stefan's review action, his execution confirmation, and (via background polling against HAFAS read endpoints) the HAFAS-state change that verifies the rotation went through. All as shadow audit entries — none of them authoritative on their own, but together a complete chain of recommendation→decision→action→outcome.

**The new reality.** Stefan made a €2,400 decision in 90 seconds with evidence the platform packaged for him, and the rotation update was applied because Stefan applied it — not because the platform applied it for him. Prevented delay-minutes: 14-minute delay contained at conductor level; the redeployment is for the *next* rotation. The counterfactual is logged. **Over 6 months, the dataset of recommendation-to-action chains demonstrates whether the platform should be trusted to apply the next one itself.**

*Capabilities revealed: Crisis entry state, incident modal, pre-seeded AI chat, Rail MCP tool calls, cost-impact one-liner, Recommended Action Object generation, step-by-step execution guidance, operator-action logging, downstream-state polling for outcome verification, complete recommendation→action→outcome shadow audit chain.*

---

### Journey 3 — Dispatcher: "The Relay" *(Post-MVP — Dispatcher persona deferred from MVP scope)*

**Meet Ana.** Ana is a Fahrdienstleiter at Salzburg control room. She monitors signal and switch systems across her section. At 14:32 an IntentPacket arrives: Train 4712, 14-minute delay, signal fault at Salzburg Hbf.

**What Ana sees.** In the Fleet Manager AI Interface, alongside the triage alert, a relay card appears: "Train 4712 is delayed 14 minutes due to signal fault at Salzburg Hbf. Expected recovery: on departure." Plain language. Passenger-safe. No fault codes.

**The action.** Ana taps "Forward to Conductor." The relay card is pushed to the conductor's PWA notification (via brain). Ana taps "Copy" and pastes it into the PA system announcement. She did not rewrite anything. She did not call the conductor. Thirty seconds from alert to passenger announcement.

**The new reality.** The relay card was generated by Kondukt from the IntentPacket `incident_summary`. Ana's job is coordination, not translation. The platform handled the translation.

*Capabilities revealed: Relay card component, LLM-generated passenger-safe language, copy-to-clipboard, forward-to-conductor push notification, Dispatcher coordination loop.*

---

### Journey 4 — ECM Lead: "Reviews the Platform" (Commercial Confidence Journey)

**Meet Dr. Renate Fischer.** Dr. Fischer is Head of Vehicle Maintenance at ÖBB Technische Services — the ECM-accountable role. She was not in the pilot planning meeting. Martin Lerch brings her in at approximately pilot manday 15. Her question is not "does it work?" — it is "is this platform's audit-trail capability sound enough that I would be willing, after pilot end, to make it the authoritative ECM record?"

**What she reviews.** She opens the ECM shadow audit-trail export for Unit 4722 alongside the original paper sign-off scan. She compares: IntentPacket generated at 05:50 (14 minutes pre-arrival), platform shadow record opened by Klaus at 05:53, fault resolution chain identical to the paper — detection → conductor ACK → technician assignment → part fitted → test result, shadow sign-off at 06:08 (Klaus's IdP identity, not a local username), vehicle state snapshot at sign-off, directive reference EU 2019/779, HMAC + hash-chain tamper evidence on each row. The shadow record and the paper match field-for-field.

**Her first question.** "Can I export this in a format our documentation system accepts?" PoC answer: structured JSON export, copy-paste ready. BOOM integration deferred — noted, not a blocker.

**Her second question.** "What happens if Klaus signs off incorrectly — can we correct it without deleting the record?" She is shown the `supersedes_id` pattern — a new row points to the original, original is never touched. She also asks the new question that advisory-only invites: "And what happens if the paper says one thing and your shadow says another?" Shown the reconciliation queue and the divergence flag — divergences are surfaced within 24h, never silently ignored, and persistent divergence downgrades the Phase 2 readiness assessment. She is satisfied.

**The outcome.** She does not object. She requests to be a reference customer for the audit-trail capability — and she conditions her support for Phase 2 on the shadow trail's match rate against paper holding at 100% across the full pilot. Roland gets his Phase 2 readiness signal. Late-stage objection risk eliminated.

*Capabilities revealed: ECM shadow audit-trail export, full resolution chain view, supersedes_id correction flow, IdP-bound identity display, HMAC + hash-chain tamper evidence, paper-trail reconciliation surface, divergence flagging, structured JSON export.*

---

### Journey 5 — The ROI Conversation (Commercial — Pilot Week 3)

**Meet Roland and Martin.** It is approximately pilot manday 15. Roland Ruisz (operations) and Martin Lerch (Technical Services) sit down with the pilot evidence dashboard. They are not assessing features — they are assessing whether to sign a rollout recommendation.

**What they see.** The pilot health score composite: ECM pre-arrival review rate 83% (target >80% ✓), Disponent decision latency reduced by 38% vs baseline, **Recommendation acceptance rate 76%**, **Shadow audit-trail match rate 100% across N sign-offs**, **Phase 2 readiness composite: AT THRESHOLD**. Below: prevented delay-minutes. Seven faults detected pre-disruption (sourced from brain telemetry). Historical baseline for same fault classes on unmonitored units: 23-minute mean delay. Estimated delay-minutes prevented: 161. **The counterfactual is honest about what the platform did and did not do.** The platform recommended; Stefan acted; the rotation went through; the delay was prevented. The platform's contribution was the speed of the recommendation, not the act itself. Phase 2 closes the remaining latency by removing the human-in-the-middle step.

**Martin's question.** "What does this cost us to run per train per month?" The platform surfaces infrastructure cost estimate. Roland cross-references against 161 delay-minutes prevented — at ÖBB's internal cost-per-delay-minute figure, the platform pays for itself in under two months of full fleet deployment — **and that's the Phase 1 ROI**. Phase 2 ROI is materially higher because the human-in-the-middle latency drops out.

**The outcome.** Roland does not need to argue the case to his CFO. The platform produced the evidence for Phase 1 ROI *and* the trust-criteria evidence that gates Phase 2. The Phase 2 commercial proposal is a contract renewal with a pre-mapped path, not a fresh procurement fight.

*Capabilities revealed: Pilot evidence dashboard — composite Phase 1 health score with threshold line, **Phase 2 trust-criteria composite with separate threshold line**, prevented delay-minutes counterfactual panel, recommendation→action→outcome chain visualization, infrastructure cost estimate input, export for executive presentation **including Phase 2 readiness verdict**.*

---

### Journey 6 — Teil-Projektleitung Telematik: "Vendor Warranty Triage" (Gemini-derived) ⚠️ *Scope assessment required*

> **[NEEDS DECISION]** Journey 6 depends on: (1) an active Probebetrieb commissioning cycle overlapping the pilot window — confirm with Lukas's programme team; (2) SAP asset registry integration (post-MVP in current scope — metadata drawers will use local registry in MVP). If neither dependency is met during the pilot, this journey is a post-pilot story. The warranty evidence packaging capability (FR41) is still in MVP scope, but cannot be evidenced during the pilot without an active Probebetrieb cycle.

**Meet Lukas.** Lukas is Teil-Projektleitung Telematik for the Cityjet DOSTO Neu programme. Seat reservation screens have gone black on three of the new Stadler double-decker units over the last week. Stadler's first response: "Check your landside reservation database — it's probably timing out our bus controller."

**Before the platform.** Lukas would have spent two days pulling logs from the vendor diagnostic portal, cross-referencing against ÖBB's PostgreSQL reservation service uptime in Power BI, exporting timestamps to Excel, and writing a Change Request rebuttal email. Likely outcome: Stadler holds the line, and the dispute escalates to procurement.

**Now.** Lukas opens the Telematics control board. He filters the fleet compatibility matrix to the affected units. He clicks one unit, opens the SVG train topology, and selects the reservation screen node. Inline drawer surfaces: MAC address, firmware version, last 24h packet trace from VLAN 6 (reservations), and the corresponding landside reservation service response latencies during the failure window. Kondukt is already on the case in zone 3: "Reservation service response P99 was 187ms during all three failure windows. Bus controller memory utilisation crossed 94% in the same window on all three units. Pattern consistent with onboard memory leak, not landside latency."

**The action.** Lukas approves a Recommended Action Object: "Package evidence dump for warranty claim — Stadler ticket #SR-2026-0517." Kondukt assembles the timestamp-correlated logs and generates a structured warranty claim document. Lukas reviews, downloads the PDF, and attaches it to the Stadler ticket himself. Two days later, Stadler authorises the bus controller swap under warranty.

**The new reality.** What used to be a multi-day forensic exercise across four systems is now a 20-minute investigation with platform-generated evidence. The vendor lock-in dynamic shifts because ÖBB now has independent diagnostic visibility.

*Capabilities revealed: Macro fleet compatibility matrix, micro SVG train topology with metadata drawers (IntentPacket + local registry in MVP), Kondukt correlation across landside + onboard telemetry, warranty evidence packaging Recommended Action Object, vendor-facing structured export, **operator-submitted to vendor systems**.*

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

**What Stefan sees.** In the Fleet Manager triage view, the fleet card for Train 4712 shows a staleness indicator — last updated timestamp, amber border. He queries Kondukt: "What is the current status of Train 4712?" Kondukt calls `get_fleet_summary`. The Rail MCP Server responds with `data_freshness_ts` showing 94 seconds ago. Kondukt surfaces this: "Last known status 94 seconds ago — onboard inference delayed. Data may not reflect current state." Stefan waits. He does not act on stale data.

**MCP timeout.** Stefan asks for the cost estimate on a redeployment. The Rail MCP Server call to `get_maintenance_cost_estimate` times out after 8 seconds. The triage card shows: "Cost estimate unavailable — data source timeout." Stefan makes the redeployment recommendation review **without the cost figure**, executes the rotation in HAFAS himself, and logs his action. The platform records his decision with a flag: cost data unavailable at decision time. Phase 2 trust evaluation tracks whether decisions made under degraded data conditions diverge in outcome from decisions made with full data.

**Recovery.** Brain inference resumes at 09:17. Fresh IntentPacket arrives. Stefan's fleet card updates. No data is lost. No decision is silently affected.

*Capabilities revealed: Landside staleness indicator on stale brain data, MCP timeout handling — structured error surfaced to UI, decision logging with data-availability flag, automatic recovery on brain ingest resume.*

---

### Journey 9 — ECM 4 Technician: "The Fault ACK" (Primary — Operational Path)

**Meet Marco.** Marco is an ECM 4 Electrical Technician at Wien Westbahnhof depot. He covers the overnight maintenance window. He is not accountable under 2019/779 — that is Klaus's responsibility — but nothing moves on Klaus's screen until Marco logs his work.

**Before the platform.** Marco read fault codes off the diagnostic terminal, wrote the resolution on a paper sheet, handed it to the shift supervisor, who transcribed it to the system. If he used the wrong fault code, the ECM sign-off referred to a resolution that didn't match the physical repair.

**Now.** At 03:15, Marco opens the Depot Briefing Interface on his tablet. He taps "Electrical" — his discipline. He sees one fault in his queue: CRITICAL, Car 4 Door 2L motor fault. He opens it. The full IntentPacket chain is visible: conductor ACK at 01:20, fault isolated to door motor assembly, part requisitioned.

**The ACK.** Marco fits the replacement part at 03:22. He taps "Acknowledge — Resolved" in the platform. **He also signs the existing paper fault log per current depot process.** The platform's shadow ACK record writes: his IdP identity, UTC timestamp, resolution action ("Replacement part fitted — Door Motor Assembly Car 4 Door 2L"), and the fault ID it references. No free text. The fault moves from OPEN to RESOLVED in Klaus's pre-arrival shadow brief.

**What he cannot do.** Marco cannot sign off the vehicle — that requires ECM 1 authority (and remains a paper process in MVP). He cannot see faults outside Electrical — Mechanical and IT faults are not in his feed. He cannot modify or delete his shadow ACK after it is written.

**The new reality.** Klaus sees Marco's shadow ACK in the platform's resolution chain at 05:50, and verifies it against the paper fault log on arrival. Over 6 months, the consistent match between Marco's shadow ACKs and the paper fault log builds the dataset that proves Phase 2 can drop the paper step.

*Capabilities revealed: Role-filtered fault feed (Electrical discipline only), structured shadow ACK with IdP-bound identity, append-only ACK record, fault state transitions (OPEN → RESOLVED), shadow ACK with paper-trail reconciliation, ACK match rate as Phase 2 trust input.*

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
| Recommended Action Object generation + three-tier confidence card review | Journeys 2, 6, 7 |
| RAO execution-guidance + operator-action logging + downstream-state polling | Journey 2 |
| Shadow audit-trail with paper reconciliation | Journeys 1, 4, 9 |
| Phase 2 trust-criteria composite scoring | Journey 5 |
| Relay card + LLM passenger-safe language | Journey 3 *(post-MVP — Dispatcher persona deferred)* |
| Forward-to-conductor push notification (to brain) | Journey 3 *(post-MVP — Dispatcher persona deferred)* |
| ECM audit trail export + supersedes_id correction | Journey 4 |
| Pilot evidence dashboard + prevented delay-minutes panel | Journey 5 |
| Fleet compatibility matrix + geospatial overlay | Journey 6 |
| SVG train topology + inline metadata drawers (IntentPacket + local registry; SAP post-MVP) | Journey 6 *(scope assessment required — see Journey 6 note)* |
| Kondukt correlation across landside + onboard telemetry | Journey 6 |
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

> **MVP posture: the paper sign-off process under existing ÖBB procedure remains the authoritative ECM record throughout the pilot. The platform records a shadow trail. Phase 2 transitions the platform to the authoritative record gated on pilot-end trust criteria.**

- Every vehicle returning to service must have a documented ECM sign-off by an authorised ECM 1 entity (MVP: paper; Phase 2: platform)
- Sign-off record must identify the authorising individual by their regulatory identity (ÖBB SSO IdP — not a local session username) (applies to both paper and shadow record in MVP; to platform record only in Phase 2)
- Audit trail must be tamper-evident and append-only — no UPDATE or DELETE permitted on `ecm_signoff_events`. Append-only enforcement plus **hash-chained row linkage** (each row includes hash of previous row); the chain proves both non-modification of individual rows AND non-deletion of any row across the trail.
- Corrections are new rows with `supersedes_id` referencing the corrected record
- Vehicle state at moment of sign-off must be captured as a JSONB snapshot
- `directive_ref` field (`EU 2019/779`) present on every sign-off record

**GDPR**
- Raw video never leaves the train (handled by brain — landside never receives video). Only anonymised event metadata reaches landside.
- No personally identifiable passenger data in any IntentPacket or event envelope reaching landside
- Events tagged for deletion scope per GDPR Article 17 — retention policy required before fleet rollout

**NIS2**
- Platform classified as critical infrastructure operator — ÖBB's NIS2 classification applies
- Incident reporting obligations: material cybersecurity incidents must be reported following the NIS2 Article 23 timeline — **early warning within 24h** of becoming aware, **incident notification within 72h**, **final report within 1 month**. **Operational owner: ÖBB IT Security (ÖBB PV and BCC departments** — not Martin Lerch's org; Martin Lerch is ÖBB Technische Services). The platform's obligation is to emit structured security-incident events with metadata sufficient to support all three reporting stages (authentication anomaly, unauthorised action attempt, BYOK key exposure, LLM injection detection) to ÖBB's security monitoring pipeline — not to perform the regulatory filing itself. FR58 covers this. **Becomes a PoC blocker when:** (a) platform handles live operational data for trains in revenue service, or (b) PoC scope expands beyond 5 trains. Whichever comes first. Before that threshold is crossed, the security incident runbook must be signed off by ÖBB PV/BCC security and Nomad Rail legal.
- BYOK API keys for Kondukt must never appear in source control, logs, or inference code paths
- Prompt injection threat model: all LLM output sanitised before reaching UI render layer

**EU Cyber Resilience Act (CRA, Regulation 2024/2847)**
- The platform is a "product with digital elements" under CRA when placed on the EU market. CRA reporting obligations apply from **11 September 2026** — before the pilot's planned start (end-Aug / Sept 2026).
- MVP commitments:
  - Structured `IncidentReport` Pydantic model shared with NIS2, extended with optional CRA fields (`product_identifier`, `cve_ids`, `actively_exploited`, `coordinated_disclosure_status`)
  - SBOM generated in build pipeline per release (cyclonedx or syft); attached to release artifacts
  - `SECURITY.md` + `.well-known/security.txt` (RFC 9116) with vulnerability disclosure process and named owner
  - Signed container images (cosign) with named update channel + runbook for on-prem operators
  - Vulnerability handling process document with triage criteria, severity rubric (CVSS), and decision tree from "report received" → "patched and disclosed"
- Live ENISA API integration is post-MVP — the dispatch layer ships stubbed in PoC; ÖBB IT Security retains regulatory filing responsibility.
- **CRA-readiness check: tabletop exercise at pilot manday 20–25 simulates the 24h–72h–30d response cycle end-to-end (see §14 Tabletop Exercise Plan).**

**EU AI Act (Regulation 2024/1689) — Article 14 Human Oversight**
- AI Act becomes enforceable on **2 August 2026**, before pilot start. Railway operations is Annex III critical infrastructure → high-risk AI system classification by default.
- **MVP advisory-only posture materially reduces exposure:** the platform does not output decisions that get automatically executed — it outputs recommendations that a human (Disponent, ECM Manager, Telematics Sub-PM) reviews and acts on via their existing operational systems. Article 14 human-oversight requirements are met by the platform's design — every Kondukt recommendation is reviewable, modifiable, and rejectable; the audit trail captures the human's decision and the action taken; the platform itself never closes the loop.
- The boundary doc (see §13) and the AI Act Article 14 compliance crosswalk annex articulate this.
- Phase 2 reintroduces dispatch and requires a re-assessment of Article 14 obligations as a precondition of release.

### Technical Constraints

**Landside Infrastructure**
- PostgreSQL for all operational state — no React component state or Kondukt conversation memory for acknowledged alerts or status changes
- pgvector extension required — `CREATE EXTENSION vector` needs superuser or `pg_extension_owner` role; migration must include `IF NOT EXISTS` guard
- **pgvector P99 latency SLA: <500ms (contractual ceiling) / <250ms at 50K vectors with ef_search=64 (internal target)** — if 500ms ceiling exceeded, fallback to keyword ranking (`search_fleet_events` full-text), logged as WARN; UI labels results "Keyword search active". Formal acceptance criterion for the pgvector spike story. Do not write the CI latency gate test until corpus exceeds 100K vectors (post-MVP).
- Alembic migrations safe under concurrent reads — no table locks during pilot operations
- All API responses snake_case JSON; React converts to camelCase at API client layer only

**Brain → Landside Ingest**
- MQTT subscriber consumes `oebb/{train_id}/intent` topic. QoS 1 + retain=True provides *last-known-state* on subscriber reconnect — it does **not** replay packets emitted during the disconnect window. Gap-period replay is the responsibility of the brain's WAL-before-truncate sync (see brain disaster-recovery story); landside is a pull-eligible target for that replay, not a passive recoverer
- Gap-recovery contract: brain replays unacknowledged packets to landside via a dedicated replay channel after reconnect; landside acknowledges per `packet_id`. The replay channel is brain-owned spec — landside subscribes
- REST + SSE bridge from MQTT to PostgreSQL; `packet_id` is idempotency key on insert (handles both live and replayed packets)
- Schema contract is brain-owned (v1.0.0 frozen); landside validates against locally-cached schema definition
- Landside detects ingest gaps by monitoring per-train packet sequence numbers (when present) or heartbeat intervals; gaps are surfaced via FR34/FR45 staleness indicators independently of replay completion
- **No landside-initiated diagnostic commands to brain in MVP.** The brain emits on every state transition within 2s — there is no polling cadence to bypass. On-demand diagnostic tools (e.g. `trigger_edge_diagnostics`) are dropped from MVP scope; see §9 decision log.

**Brain Schema Compatibility (Forward-Compat Trip-Wire)**
- Landside maintains `intent_packet_schema_registry` table keyed by `schema_version`. On every inbound IntentPacket, diff observed fields against registry. Unknown field → log structured event `intent_packet.unknown_field` + metric, do NOT drop the packet. Major version mismatch → quarantine + alert. Minor version mismatch → log + accept. This is the cross-workspace contract trip-wire — pairs with the brain workspace's schema-versioning posture.

**Recommended Action Object Outbox Pattern**
- All Recommended Action Objects and their reconciliation state persist in `recommended_action_outbox` (PostgreSQL table) with `status` enum, `idempotency_key UNIQUE`. In-memory queues are not used. Worker polls + `SELECT ... FOR UPDATE SKIP LOCKED`. This survives platform restart without loss — critical for the shadow-audit-trail integrity guarantee.

**Security**
- BYOK API keys stored in environment variables (PoC) or Docker secrets (fleet) — never in source control
- All external data (IntentPackets, MCP tool inputs, vendor diagnostic feeds) treated as untrusted — schema validation before any AI decision path
- LLM output sanitisation layer required between Kondukt response and Fleet Manager UI render
- **Recommended Action Object review and operator-action logging** — the platform does not dispatch in MVP. The 'Mark executed' confirmation is recorded against the operator's IdP-bound session as a shadow audit event. Re-verified IdP session is not required for the confirmation gesture because no operational system is being mutated by the platform.

### Integration Requirements

| System | Integration type | Status | Notes |
|---|---|---|---|
| Identity provider (IdP) | Identity for ECM shadow sign-off + RAO confirmation | **MVP: Keycloak with manually created users.** ÖBB IdP integration is a post-MVP change request, costed separately. | Must be IdP-bound, not local session |
| MQTT broker | IntentPacket consumption from oebb-brain | Owned by brain — landside is subscriber | `oebb/{train_id}/intent` |
| Rail MCP Server | HTTP MCP endpoint — spec frozen. 9 tools: `get_fleet_events`, `get_fleet_summary`, `get_hafas_timetable`, `search_fleet_events`, `get_maintenance_cost_estimate`, `get_shift_handover`, `get_ecm_audit_trail`, `get_intent_packet`, `get_forward_planning`. `trigger_edge_diagnostics` removed 2026-05-25 (see §9 ADR). Spec authority: `rail-mcp-server-spec-2026-05-23.md` | Spec frozen | New subpackage `rail-mcp-server/` |
| Kondukt (BYOK) | LLM orchestration agent runtime | Designed, not built | Nous Research, MIT licence |
| HAFAS | Timetable data (read-only in MVP); rotation write-back deferred to Phase 2 | Required for Fleet Manager + Telematics | 48h horizon for planning; MVP reads only — operator dispatches via HAFAS directly |
| SAP PM/EAM | Asset registry + work order creation | **Post-MVP** — no SAP integration in PoC. Topology metadata drawers and work order write-back require API contract with ÖBB TS. | Read for asset metadata; write for work order routing |
| ServiceNow ITSM | Ticket creation from Fleet Manager triage | **Post-MVP** | Bidirectional sync — incidents flow both ways |
| BOOM | Depot work order integration | **Descoped for PoC** | Structured JSON export/copy for manual entry |
| pgvector | Semantic search over `incident_summary` embeddings | Requires migration spike + P99 SLA confirmation | Fallback: keyword ranking |
| Vendor diagnostic portals (Stadler, Siemens) | Read-only — evidence for warranty triage | Post-MVP | Authentication via vendor-issued credentials; scoped read |
| Power BI Diagnoseplattform | Coexistence — high-level fleet uptime visualisation | No integration in MVP | Platform does not replace; Telematics persona uses both |

### Risk Mitigations

| Risk | Mitigation |
|---|---|
| BYOK prompt injection | Output sanitisation layer between Kondukt and UI render — validated schema, reject unexpected fields, log anomalies |
| pgvector migration blocks reads | Migration with `IF NOT EXISTS` guard; confirm PostgreSQL user has extension creation rights; document `DROP EXTENSION vector CASCADE` rollback risk |
| ECM sign-off record loss (connectivity gap) | Queue locally at depot, sync on reconnect; extended loss flagged in audit as gap with last-known timestamp |
| BOOM API unknown | Scoped as "BOOM-ready" not "BOOM-integrated" — structured export only until API contract confirmed with ÖBB IT |
| NIS2 incident reporting obligation (24-72-30 cadence) | Platform emits structured security incident events (FR58) supporting all three reporting stages (24h early warning, 72h notification, 30d final report); ÖBB IT Security (ÖBB PV and BCC departments) owns the regulatory filings. Runbook must be signed off before platform handles live revenue-service data or exceeds 5-train scope — whichever comes first. Tabletop exercise at manday 20–25 validates end-to-end. Owner: ÖBB PV/BCC security + Nomad Rail legal. |
| CRA reporting readiness | Structured incident endpoint + SBOM + security.txt + vuln-handling process + signed releases in place before pilot manday 1. Tabletop exercise at manday 20–25 validates the end-to-end process. Live ENISA integration deferred to Phase 2. |
| AI Act Article 14 oversight design | Advisory-only MVP design satisfies Article 14 by construction — Kondukt outputs recommendations, never automated decisions. Phase 2 release requires a fresh Article 14 assessment before dispatch capability is enabled. |
| § 1313a ABGB / Erfüllungsgehilfe | Austrian civil code defaults to inclusion of sub-performers in contracting party's responsibility unless explicitly carved out. Boundary doc duties-matrix annex enumerates operator duties *affirmatively* (not by exclusion). Third-party components (Hailo runtime, OS, key-management lib) get explicit treatment — flow-down liability to suppliers OR explicit ÖBB-acceptance documentation. See §13. |
| BYOK data residency not confirmed | BYOK keys and vehicle state snapshots must be designated to EU GDPR processors. **Blocks procurement sign-off for multi-tenant rollout.** PoC unaffected. Must be resolved before commercial expansion. |
| pgvector latency exceeds SLA | Fallback to keyword ranking (WARN logged, UI labelled). Spike story must confirm or revise <500ms P99 target before Fleet Manager stories begin. |
| RAO outcome-verification gap | When the operator marks an RAO executed but the platform cannot verify the outcome via downstream polling, the entry is recorded as 'self-reported unverified'. Persistent unverified outcomes feed into Phase 2 readiness assessment and may downgrade trust criteria. |
| Kondukt data residency vs vendor portal credentials | Vendor portal credentials must not pass through inference code path; resource initialisation pattern enforced |

### UX Constraints (Domain-Driven)

**ECM Shadow Sign-Off Gesture (MVP) / ECM Authoritative Sign-Off Gesture (Phase 2) — Option C (resolved 2026-05-25)**

The sign-off interaction is a two-phase gesture: hold-to-initiate (3s) → 5-second visible countdown → single atomic write at expiry. Full interaction arc specified in `design-artifacts/ecm-signoff-interaction-spec.md`. **In MVP, the gesture records the platform's shadow entry that matches the operator's just-completed paper sign-off.** The gesture is preserved unchanged for Phase 2, when the same gesture writes the authoritative record.

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

**Glove gesture acceptance criteria — pre-pilot gate.** Walkthrough with an ECM 4 Technician timing the 8-second total gesture (3s hold + 5s countdown) against a real 14-minute depot window, **wearing the specific issued ÖBB winter glove (named SKU), across at least two depot lighting conditions, n ≥ 20 sessions. Acceptance bundle:**
- ≥90% first-try success
- median completion ≤9s, p95 ≤14s
- false-abort rate <5%
- subjective trust ≥4/5 on post-shift SUS-style prompt

If acceptable, full gesture ships; if 8s is unacceptable, countdown compresses to 3s. Two-phase structure non-negotiable regardless.

**pgvector Fallback Transparency**
- Fleet Manager always knows which search mode is active — semantic or keyword
- UI label "Keyword search active" shown whenever fallback is in effect
- No silent degradation permitted

**Recommended Action Object Review (MVP)**
- The 'Mark executed' button confirms the operator has acted on the recommendation via their existing system; it does not dispatch
- The pre-action summary lists every system the operator must touch (HAFAS in MVP) with the exact payload they should enter
- Confirmation logs the operator's IdP identity, timestamp, and the system-by-system outcome they self-report
- Platform polls downstream systems where possible to verify the self-reported outcome
- Self-reported-unverified outcomes are surfaced as distinct UI state — not buried in error logs

**Three-Tier Recommendation Card Visual Grammar**

Every RAO presented to the operator is rendered in one of three tiers, with the rule that fired visible on the card.

- **Tier 1 — Confident card (green rail, single-tap accept).** Left edge: 4px solid ÖBB green. Header line: action verb, ETA delta, cost delta. Body: two lines max, source-system badges right-aligned. Primary CTA: `Accept` (full-width footer; Enter to confirm). Keyboard: Enter accepts, D opens details, Esc defers.
- **Tier 2 — Hedged card (amber rail, accept-with-acknowledgment).** Left edge: 4px solid amber. Header includes inline confidence tag (lowercase, no exclamation: `partial data` / `est. cost` / `single-source`). Body: three lines, third line names *what's hedged* in plain language. Primary CTA: `Accept with hedge` with 1-second debounce; button briefly shows "Confirming...". Keyboard: Enter triggers debounce-accept, V jumps to verify field, Esc defers.
- **Tier 3 — Low-confidence card (slate rail, forced two-step).** Left edge: 4px slate-grey (deliberately *not* red — red reserved for safety alarms). Header includes open-circle glyph. Body: four lines, first body line names *reason for low confidence* in plain language. Primary CTA: two-step `Review & Accept` → inline expansion shows three system payloads side-by-side → `Confirm dispatch` from inside the expansion. Secondary: `Reject with reason` (4-5 common reasons). Keyboard: Enter opens expansion, Enter again from inside confirms, R rejects, Esc collapses.

**Tier-selection rule (deterministic, rules-based — calibrated confidence model deferred to Phase 2):**

A card lands in **Tier 3 (slate)** if ANY of:
1. Action touches more than one downstream system (e.g., ECM + Telematics + Crew)
2. Action is irreversible within 15 minutes (e.g., tag-out, brake-release lockout, crew reassignment triggering union timer)
3. Source IntentPacket has `staleness > 10min` OR was emitted from a known-blind segment
4. Estimated cost delta > €5,000 OR ETA delta > 20min
5. RAO `confidence` field present AND < 0.6 (Phase 2-reserved field; null in MVP, rule inactive until populated)

A card lands in **Tier 2 (amber)** if ANY of (and no Tier-3 conditions fire):
1. Any cost or ETA field marked `estimated: true` rather than `measured: true`
2. Source data 3–10 minutes stale
3. MCP tool call chain length ≥ 3
4. Single-system action but touches passenger-facing surface (PIS update, platform change)
5. RAO `confidence` field 0.6–0.85

Otherwise Tier 1.

**Depot Briefing PWA haptic counterpart:** Tier 1 brief → no haptic, single tap to ack. Tier 2 hedged brief → single 50ms haptic on display, ack with 250ms held tap. Tier 3 forced-attention brief → double haptic (50ms-100ms-50ms), ack requires swipe-and-confirm.

**PWA Wake-from-Sleep Behaviour**

Applies to: Depot Briefing PWA, Conductor App PWA (any landside-rendered PWA surface).
Trigger: PWA regains foreground focus after ≥30 seconds of background/locked state.

On every foreground-regain event, the PWA SHALL:
1. Display the last-known-good content immediately (no blocking spinner)
2. Overlay a non-blocking refresh indicator ("Refreshing — last updated HH:MM")
3. Fire a freshness check to the landside session endpoint within 200ms
4. If no changes → indicator turns green-checkmark for 2 seconds, fades
5. If changes available → apply per freshness-class table below

**Stale data classification:**

| Data class | Max staleness | Behavior if exceeded |
|---|---|---|
| Static reference (depot map, consist composition once assigned, route geography) | Session lifetime | Never blocks |
| Personal (Markus's shift assignment, name, certifications) | 8 hours | Re-fetch silently in background |
| Operational briefing (weather, route advisories, schedule) | 5 minutes | Show with "Updated HH:MM" tag; refresh in background; slide-in banner if new content arrives |
| Trust-critical (new ECM advisory issued during sleep, new safety bulletins) | 0 minutes — must refresh before display | Full-screen "Refreshing safety data..." overlay until confirmed current OR offline-fallback fires |

**Auth re-prompt predicates** (re-prompt for PIN/biometric per depot policy only if):
- Session token expired (hard rule, no exceptions)
- OR background duration ≥ 4 hours (cold-start trust threshold)
- OR device moved depot/geofence since lock (anti-handoff)
- OR trust-critical content has changed AND background duration ≥ 30 minutes

Skip re-prompt if: token valid AND background < 4 hours AND same depot geofence AND (no trust-critical change OR background < 30 min).

**Offline fail-state:** if freshness check fails (timeout 3s, no network): indicator turns amber ("Offline — showing brief from HH:MM"). All non-trust-critical content readable. Trust-critical blocks show slate placeholder. PWA retries every 30s. If background ≥ 4 hours AND offline, force re-auth on next online moment.

**Acceptance criteria:** wake-to-readable median <500ms; every content block carries its own `Updated HH:MM` tag; no blocking UI on the happy path; full enumerated state coverage in test matrix.

**Audio-Alert Policy**

Audio is **not** the primary alert channel in any depot or operations surface. Multi-modal alert spec: **haptic ≥200ms pulse + visual ≥500ms full-bleed colour shift + audio as confirmation only** (no dB-threshold requirement above ambient). Rationale: depot ambient noise (78–84 dB at Wien-Matzleinsdorf) makes any reliable audio threshold also hated. Owner: Freya/Sally. Due: end of Phase 1 design (pre-pilot).

**Colour-Blind State Encoding**

Every multi-state UI element (notably the FR36 fleet matrix's three-state encoding active/gap/not-yet-onboarded) must be distinguishable without colour — shape, label, and one-character text token in addition to colour. WCAG 1.4.1 conformance is non-optional on safety surfaces. Deuteranopia rate ~8% in a male-skewed workforce; safety board cannot rely on hue alone.

---

## 6. Project-Type Specific Requirements

### Project-Type Overview

This platform is a **landside B2B SaaS surface** with multi-system integration and an embedded AI agent (Kondukt BYOK). It serves three landside user surfaces — Depot Briefing PWA (mobile-first), Fleet Manager AI Interface (web), and Telematics Project Management surfaces (web, 3-zone control board).

---

### Technical Architecture Considerations

**Multi-tenancy model**
PoC: single ÖBB tenant, on-premise. No multi-tenancy. Fleet rollout: one instance per operator (ÖBB, Westbahn, etc.) — shared codebase, separate deployments. Shared-database multi-tenancy explicitly deferred — BYOK data residency and ECM audit trail integrity require tenant isolation.

**Permission model**
Five landside roles, mapped to surfaces:

| Role | Surface | Permissions |
|---|---|---|
| Fleet Manager (Disponent) | Fleet Manager | Full triage; recommendation review; operator-action logging; shift handover. **Does not dispatch — executes via existing systems.** |
| ECM Manager (ECM 1) | Depot Briefing | Pre-arrival brief review; **paper sign-off (existing process); shadow sign-off via platform gesture**; audit trail read; blocking state control |
| Technician (ECM 4) | Depot Briefing | Role-filtered fault feed; **shadow ACK of faults in own discipline (paper ACK remains existing process)** |
| Teil-Projektleitung Telematik | Unified Telematics Control Panel (primary); Fleet Manager (full — same user as Disponent) | Full fleet matrix + topology + OTA monitoring + Digital Twin validation + **warranty evidence-export review (export submitted manually to vendor systems)**. As Roland Ruisz holds both the Fleet Manager and Teil-Projektleitung Telematik roles, he has full Fleet Manager review authority across both surfaces. |

All identity via IdP-bound session — no local user table for safety/compliance actions. **MVP uses Keycloak with manually created users.** ÖBB SSO IdP integration is a post-MVP change request, costed separately.

**IdP per-operator**
MVP uses Keycloak with manually created users — no ÖBB SSO integration. ÖBB SSO IdP integration is a post-MVP change request, costed separately. Fleet rollout: each operator brings their own IdP. Per-operator SSO customisation will be requested — budget 2–3 week integration tax per operator or build a standardised SAML bridge.

---

### Integration List

| System | Integration type | PoC status | Fleet gate |
|---|---|---|---|
| Identity provider (IdP) | Identity for ECM shadow sign-off + RAO confirmation | **MVP: Keycloak with manually created users.** ÖBB SSO IdP integration is post-MVP — scoped as a change request. | Per-operator SAML bridge at fleet rollout |
| MQTT broker | IntentPacket consumption from oebb-brain | Onboard subscriber only | **Blocking pre-fleet:** broker identity, failover, and multi-train topic isolation specified by brain team |
| Rail MCP Server | HTTP MCP endpoint — 9 tools for Kondukt | Spec frozen | Required for Fleet Manager + Telematics |
| Kondukt (BYOK) | LLM orchestration, operator-owned API keys | Designed, not built | BYOK cost simulator required before fleet sales |
| HAFAS | Timetable + rotation write-back | Required for Fleet Manager | 48h horizon |
| SAP PM/EAM | Asset metadata + work order write-back | **Post-MVP** — API contract required with ÖBB TS before integration stories | Read for asset metadata; write for work order routing |
| ServiceNow ITSM | Ticket creation from triage | Post-MVP | API contract required |
| BOOM | Depot work order integration | Descoped — structured JSON export only | API contract required from ÖBB IT before integration stories |
| pgvector | Semantic search over incident_summary embeddings | Spike + P99 confirmation required | Fallback: keyword ranking |
| Vendor diagnostic portals | Read-only warranty evidence | Post-MVP | Vendor credential model required |

---

## 7. Innovation Focus

### What Is Novel Here (Landside)

This platform reframes landside fleet operations around **structured recommendation infrastructure** — every operationally relevant observation produces a Kondukt-generated recommendation; every operator action against that recommendation is captured as a versioned, immutable shadow audit event; every paper-trail event is reconciled against the shadow. The novel claims, in MVP-applicable form:

---

**Claim 1 — Kondukt Recommended Action Objects bridge AI advice and operator action via a single readable review gate**

The Disponent currently switches between 4–7 systems to execute one redeployment decision and produces no structured record of the decision chain. The platform's Recommended Action Objects bundle multi-system action descriptions (HAFAS rotation update + SAP work order — SAP post-MVP) into a single structured proposal with step-by-step execution guidance. The operator reviews, acts via existing systems, and confirms — the platform shadows the chain. Phase 2 (post-pilot) closes the human-in-the-middle latency once trust criteria are met; the structural innovation in MVP is the *evidence-grade recommendation→action→outcome dataset* that gates Phase 2.

*Validation: RAO acceptance rate >70% in first 6 weeks of pilot; modification + rejection rates catalogued; **shadow audit-trail match against operator self-report and downstream-system polling tracked as Phase 2 trust input.***

---

**Claim 2 — Vendor warranty leverage from end-to-end network path visibility**

The Teil-Projektleitung Telematik persona currently fights vendor lock-in by manually correlating logs across vendor portals, Power BI, and Excel. The platform's correlation across onboard IntentPackets + landside service telemetry + local asset metadata produces a structured warranty evidence packet in 20 minutes vs the current 2-day process **for the Telematics Sub-PM to submit to vendor systems.** *(SAP asset metadata cross-reference is post-MVP — MVP uses local registry.)* This shifts the vendor-customer power dynamic during the high-pressure Probebetrieb commissioning phase.

*Validation: Time-from-dispute to evidence-package, measured on at least 3 warranty triage cycles during pilot.*

---

**Claim 3 — Regulatory compliance as a trust-building primary feature**

The ECM shadow audit trail is not a log — in MVP it is the proof artifact that the platform's audit-trail capability *can be* the regulatory record. Every shadow sign-off is an append-only, hash-chained record with IdP-bound identity, vehicle state snapshot, EU 2019/779 directive reference, and HMAC tamper evidence. Corrections are new rows with `supersedes_id`. The pilot's reconciliation against paper trails — 100% match rate target — is the evidence Dr. Fischer needs to authorise Phase 2.

*Validation: Shadow audit export matches paper sign-off field-for-field in 100% of pilot sign-offs — confirmed by ECM-accountable stakeholder during pilot manday ~15 and reviewed continuously thereafter.*

---

**Claim 4 — OTA rollout monitoring with configuration drift detection**

Configuration drift across workshops (Simmering, Floridsdorf, Linz) is the Teil-Projektleitung's recurring pain. The platform gives ÖBB visibility over OTA rollouts executed by the train manufacturer or Nomad Rail — Teil-Projektleitung monitors rollout progress and firmware/configuration drift across the fleet; ÖBB does not execute the rollouts. The platform detects hardware revision mismatches and surfaces incompatible units for manual review — eliminating the "discover the drift two weeks later in a workshop" failure mode.

*Validation: At least one configuration drift instance detected and surfaced during a pilot OTA rollout monitoring cycle.*

---

**Claim 5 — Advisory-first earns the right to dispatch**

No competitor in this market offers a phased trust progression where Phase 1 is recommendation + audit-capability proof, and Phase 2 unlocks dispatch only after trust criteria are met. This sequencing converts the buyer's risk-committee objection ("we cannot let an AI dispatch on day one") into a contractual phase the buyer's procurement office can sign with confidence. **Operational-sovereignty retention is itself the sword:** the boundary doc's first paragraph claims, in affirmative terms, the differentiator no incumbent can match in six months.

*Validation: Roland's Phase 2 readiness sign-off at manday 15+ on the basis of the trust-criteria composite.*

---

### Innovation Risks

| Risk | Mitigation |
|---|---|
| Recommended Action Object UX template — three-tier confidence card grammar | Freya prototypes the three-tier card pattern (confident/hedged/forced-two-step) per §G26 before stories begin; rules-based tier selector unit-tested before pilot |
| Kondukt BYOK + operator-action logging security | Kondukt spike 4 (BYOK 429 + credential handling) must complete before RAO stories begin; output sanitisation between Kondukt and UI render layer is mandatory |
| Vendor portal integration legal posture | Read-only access to vendor portals may be restricted by vendor contracts — confirm with ÖBB legal before Telematics warranty stories |
| Telematics surfaces under-spec at UX level | Scope is locked (§8) but UX detailing is in flight; Freya owns the Unified Telematics Control Panel wireframes and interaction spec before stories begin. PRD scope does not change; UX delivery sequencing does. |
| Phase 1 trust criteria not met at pilot end | Composite criteria are weighted, not binary — partial trust unlocks a narrower Phase 2 scope (e.g., dispatch on read-only HAFAS rotation updates only, deferring SAP write-back further). Phase 2 release is itself phased. |

---

## 8. Project Scoping & Feature Prioritisation

### Strategy & Philosophy

**Approach:** Compliance-first, advisory-only landside platform PoC — single release covering Depot Briefing + Fleet Manager + Telematics control board surfaces, all in advisory mode. Phase 2 (post-pilot) reintroduces dispatch and authoritative recording, gated on pilot-end trust criteria. Commercial pilot narrative leads with ECM proof point (Dr. Fischer sign-off on shadow-vs-paper match rate), Fleet Manager second (Disponent decision latency on recommendation→action chains), Telematics third (warranty triage on first Probebetrieb cycle).

**Resource Requirements:** One full-stack dev (React + Python/FastAPI), one AI/ML engineer (Kondukt integration, prompt engineering, output sanitisation), one DevOps/infra (Docker, MQTT subscriber, PostgreSQL, pgvector).

---

### Complete Feature Set

**Must-Have Capabilities:**

| Capability | Rationale |
|---|---|
| MQTT subscriber for IntentPacket ingest | Foundation — nothing works without consuming brain output |
| Depot Briefing — ECM Manager pre-arrival brief | Primary commercial proof point for Dr. Fischer |
| Depot Briefing — ECM 1 blocking state + append-only shadow audit trail | Phase 1 records shadow; Phase 2 promotes to authoritative. Regulatory capability proof — closes the compliance sale. |
| Depot Briefing — role-filtered technician feed (electrical/mechanical/IT) | Operational value; required for depot pilot |
| Fleet Manager — triage + cost-impact one-liner | Disponent KPI alignment; prevented delay-minutes counterfactual |
| Fleet Manager — Kondukt BYOK + Rail MCP Server (9 tools) | Natural language query layer; differentiates from dashboard competitors |
| Fleet Manager — Recommended Action Objects with three-tier confidence card review + operator-action logging | Core landside innovation — multi-system recommendation under single review gate. Dispatch is operator-side via existing systems in MVP. |
| Fleet Manager — shift handover flow | Required for multi-shift pilot |
| pgvector semantic search + keyword fallback | Spike gate: P99 <500ms contractual (target <250ms at 50K vectors) before stories |
| ECM shadow audit trail export (JSON) with paper-reconciliation status | Dr. Fischer's sign-off requirement; reconciliation status is the Phase 2 trust signal |
| Operator-action logging + downstream-state polling for RAO outcome reconciliation | The shadow audit chain's outcome layer. Operator self-reports execution; platform polls downstream systems where possible to verify. |

**Telematics Unified Control Panel (MVP — locked scope):**

| Capability | Rationale |
|---|---|
| Zone 1: Macro situational awareness — fleet compatibility matrix (DOSTO Neu + Enzo — both new Stadler deliveries; pilot on DOSTO Neu, Enzo onboarded on rollout success) + geospatial telemetry overlay with dead-zone detection | Daily-use surface for Teil-Projektleitung; foundation for warranty + OTA evidence |
| Zone 2: Micro visual train topology — SVG anatomy + inline metadata drawers per hardware node (MAC, serial, firmware; sourced from IntentPackets + local registry in MVP; SAP asset registry post-MVP) | Required for warranty triage and digital-twin validation flows |
| Zone 3: Kondukt AI Co-Pilot Console — chat + **Proactive Recommendation Cards in RAO format with three-tier confidence visual grammar** | The action surface; the same RAO pattern as Fleet Manager |
| Cross-zone interaction binding (Zone 1 vehicle select → Zone 2 topology; Zone 1/2 fault → Zone 3 card) | Without this, the three zones are three disconnected pages |
| OTA rollout monitoring: canary uptime tracking + firmware/configuration drift detection (local registry in MVP; SAP asset registry post-MVP) | Use Case 4 — ÖBB monitors and approves rollouts; platform surfaces drift, not executes rollout |
| Warranty evidence packaging Recommended Action Object (operator submits to vendor systems in MVP) | Use Case 5 — vendor leverage during Probebetrieb |
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
| pgvector P99: <500ms contractual ceiling / <250ms at 50K vectors internal target (ef_search=64) | Before Fleet Manager search stories | Keyword fallback only; semantic search deferred |
| Kondukt BYOK 429 handling + credential isolation | Before RAO stories | RAO feature deferred to post-PoC |
| RAO outcome reconciliation semantics (operator-execution polling against HAFAS read endpoints) | Before recommendation→action chain stories | Reduce scope to single-system recommendations only |
| SAP PM API contract for write-back | Post-MVP gate — SAP not in MVP scope (Phase 2 expansion) | Phase 2 ships HAFAS dispatch only initially; SAP work order write-back requires API contract with ÖBB TS |
| CRA-readiness tabletop exercise | Pilot manday 20–25 | Goes/no-goes the Phase 2 readiness recommendation independently of the trust criteria |

---

### Risk Mitigation Strategy

**Technical risks:**
- pgvector P99 <500ms contractual, <250ms internal — spike story with keyword fallback confirmed before Fleet Manager search stories
- Kondukt BYOK 429 + credential isolation — spike before any RAO story
- **Shadow audit-trail divergence from paper** — explicit reconciliation queue with 24h SLA on flagging divergences; 100% match rate is a Phase 2 gate
- Brain ingest gap — coordinate with brain team on WAL-before-truncate sync; landside surfaces staleness explicitly

**Market risks:**
- Dr. Fischer unavailable at pilot manday ~15 — audit trail still ships; sign-off milestone deferred
- Vendor portal access blocked — Telematics warranty story deferred or scoped to landside-only evidence

**Resource risks:**
- If team is constrained: defer Telematics surfaces (Gemini-derived); ship Depot Briefing + Fleet Manager core
- 48–72h forward planning view is nice-to-have, drop first

### Phase 2 Readiness Gates

Phase 2 (post-pilot release introducing platform-side dispatch and authoritative ECM recording) requires the **composite of all five** criteria:

1. **RAO acceptance rate** ≥ Roland-confirmed threshold (working assumption ≥70%)
2. **Shadow audit-trail match rate vs paper** = 100% across the full pilot
3. **Dr. Renate Fischer's positive sign-off** on audit-export quality and capability proof
4. **Outside-auditor tabletop verdict** positive (from manday 20–25 exercise — see §14)
5. **Zero unresolved pilot-kill triggers** in the final 30 days of pilot

All five required for full Phase 2 progression. Partial satisfaction triggers narrower Phase 2 scope (per §7 Claim 5) — e.g., dispatch on read-only HAFAS rotation updates only, deferring SAP write-back to a later phase. Phase 2 is itself phased.

Owner: Roland Ruisz signs off. Martin Lerch (Technical Services) and Dr. Renate Fischer (ECM accountable) co-sign.

---

## 9. Open Architecture Decisions

The following decisions are deferred to the architecture phase. They are not PoC blockers but must be resolved before the first story in their respective epic begins. The architect owns these inputs.

| Decision | Context | Gate |
|---|---|---|
| Kondukt intent disambiguation rules | FR identifying user intent and selecting Rail MCP tool. Rules-based, embedding-based, or LLM-classified is an architecture decision. | Before Fleet Manager Kondukt stories |
| RAO outcome reconciliation semantics | What constitutes "recommended" vs "reviewed" vs "operator-executed" vs "outcome-verified"; how outcome polling works against HAFAS read endpoints; how self-reported-unverified outcomes are surfaced. **Dispatch reconciliation moves to Phase 2 backlog.** | Before RAO outcome-polling stories |
| MQTT broker identity (consumer side) | Subscriber host, auth model, multi-train topic subscription strategy | Owned by brain team — landside consumes spec |
| Schema evolution strategy (landside consumer) | When brain bumps IntentPacket schema, how landside handles forward/backward compatibility for in-flight packets | Post-PoC; must not block PoC |
| Telematics surface scope | ~~Which Gemini-derived surfaces are MVP vs post-MVP~~ — **resolved 2026-05-25:** full Unified Telematics Control Panel locked into MVP. See §8 capability table. | n/a — closed |
| Landside→brain command channel (`trigger_edge_diagnostics`) | **Resolved 2026-05-25:** Dropped from MVP. The brain emits IntentPackets on every state transition within 2s — there is no polling cadence to bypass. Landside staleness is caused by hardware/network gaps that a diagnostic ping cannot resolve, or by the brain not detecting a state change yet, which a ping also cannot resolve. Warranty triage (Journey 6) requires immutable timestamped chain-of-custody evidence — on-demand diagnostics at investigation time would weaken, not strengthen, vendor disputes. Landside remains a pure IntentPacket subscriber for v1. **Revisit trigger:** post-pilot telemetry shows >X% of Telematics or Fleet Manager decisions made on IntentPackets older than the brain's 5-minute pilot-kill-trigger threshold — specifics named after pilot baseline established. Any future landside-initiated tool requires a joint brain+landside ADR signed by both tech leads before scoping begins. | n/a — closed |
| Fleet Manager AI interface frontend wrapper | Browser app shell architecture (React layout, state management, routing) over Kondukt agent backend | Before Fleet Manager UI stories |
| Phase 1 trust criteria thresholds | Specific numeric thresholds (especially RAO acceptance rate target) require Roland Ruisz agreement before pilot manday 1. Default working assumption: ≥70% acceptance rate, 100% shadow-match rate, qualitative sign-offs from Renate + outside auditor. | Pilot manday 1 |
| Phase 2 contractual structure | Is Phase 2 a contract renewal, a contract amendment, or a clause already in the Phase 1 contract that auto-activates on trust-criteria satisfaction? Pricing-model owner decides; legal counsel reviews. | Before procurement review (target: ~2 weeks pre-pilot) |
| Boundary doc structure & ownership | 3-layer structure (capability brochure body / sword; duties-matrix annex / § 1313a defence; compliance crosswalk annexes / shield) per §13. Body authored by Mary; duties matrix co-authored by Winston (legal precision) + Mary (operational language); compliance annexes authored by Winston with Austrian-qualified legal review. | Joint drafting session mid-June 2026 if legal counsel availability confirmed; doc v1.0 to Martin Lerch ~3–4 weeks before pilot manday 1; procurement copy ~2 weeks before pilot manday 1 |
| Austrian-qualified legal counsel engagement | Confirm availability by end of current week. Single critical-path dependency for the boundary-doc timeline. | This week |
| Martin Lerch / ÖBB IKT-Sicherheitsrichtlinie taxonomy request | Deferred. Abbas will ask Martin to flag *when* taxonomy alignment becomes necessary, by which MVP milestone. Standing question to Martin. | Awaiting Martin's signal |

---

## 10. Non-Functional Requirements

### Performance

| Requirement | Target | Context |
|---|---|---|
| pgvector semantic search | P99 < 500ms (contractual ceiling) / P99 < 250ms at 50K vectors, ef_search=64 (internal target) | Under realistic incident volume — spike gate before Fleet Manager search stories |
| IntentPacket landside ingest latency | < 2s from MQTT receipt to PostgreSQL commit | Excludes brain-side emission latency |
| Fleet Manager triage feed load | < 3s initial render | PostgreSQL query under PoC fleet size (~10–20 vehicles) |
| ECM shadow sign-off commit | < 1s write acknowledgement | Landside PostgreSQL direct write to shadow audit trail |
| Rail MCP Server tool call timeout | 8s hard timeout | Kondukt surfaces structured error; Fleet Manager decision logged with data-availability flag |
| RAO outcome-polling latency | < 30s per system after operator-execution mark | Polling cycle against HAFAS read endpoints; verification result surfaces on the shadow record |
| Kondukt response latency for query (BYOK) | P95 < 8s | Operator-owned LLM; depends on BYOK provider |

### Reliability

| Requirement | Target | Context |
|---|---|---|
| ECM audit trail integrity | Append-only, hash-chained, tamper-evident | No UPDATE or DELETE permitted; HMAC on every row; hash-chain links each row to previous (`sha256(prev_row_hash \|\| row_hmac)`); row-level trigger raises exception on modification attempt. MVP records shadow trail; Phase 2 promotes to authoritative. |
| Landside event delivery | No duplicate delivery after service restart | Idempotency on `packet_id` |
| Alembic migration write-lock ceiling | < 30s | Any migration exceeding this threshold requires an offline maintenance window; rollback path documented before merge |
| RAO outcome reconciliation | All operator-reported outcomes verified against downstream-system polling within 30s; unverified outcomes flagged within 24h | Reconciliation queue persistent in `recommended_action_outbox`; in-memory queues not permitted |
| Kondukt BYOK retry policy | Exponential backoff, budget-capped | Configurable per operator; default 3 retries with 1s/4s/15s delays |

### Security

| Requirement | Target | Context |
|---|---|---|
| ECM shadow sign-off identity | IdP-bound only | Shadow sign-off identity bound to ÖBB SSO (Keycloak in MVP, ÖBB IdP post-MVP). Paper sign-off identity remains ÖBB's existing process. |
| RAO 'Mark executed' confirmation identity | IdP session re-verified at moment of confirmation | This logs the operator-side action against a verified identity but does not authorise dispatch (no dispatch in MVP). |
| BYOK API key storage | Never in source control, logs, or inference paths | Env vars (PoC), Docker secrets (fleet) |
| LLM output sanitisation | Validated Pydantic schema before UI render; retry-with-error-feedback on schema failure (≤2 retries, then escalate to human-readable error surface) | Between Kondukt response and Fleet Manager render layer — prompt injection mitigation |
| Vendor portal credential isolation | Resource initialisation only — never inline in inference paths | Per-vendor credentials scoped to read-only operations |
| External data validation | Schema validation before any AI decision path | All IntentPackets, MCP tool inputs, vendor diagnostic feeds treated as untrusted |

### Scalability

| Requirement | Target | Context |
|---|---|---|
| PoC fleet size | 1–5 trains, 10–20 vehicles | Single ÖBB tenant, on-premise |
| Fleet rollout architecture | Per-operator deployment | Separate instances per operator — no shared-database multi-tenancy in initial fleet architecture |
| Kondukt API quota | Operator-owned | BYOK model — operator owns API quota; platform does not throttle |
| PostgreSQL read concurrency | Migrations safe under concurrent reads | No table locks on reads during pilot operations |
| Telematics fleet matrix render | Smooth at 109 vehicles (full Cityjet DOSTO Neu fleet) | UI virtualization required at fleet rollout scale |

### Accessibility

| Requirement | Target | Context |
|---|---|---|
| Depot Briefing tap targets | Minimum 56dp | Mobile PWA — depot personnel may be wearing gloves. See UX Constraints §"Glove gesture acceptance criteria" for full acceptance bundle. |
| Fleet Manager keyboard navigation | Full keyboard access — no mouse-only interactions | Disponent works long shifts; keyboard primary |
| Contrast ratio | WCAG AA minimum across all surfaces | Depot lighting varies; Fleet Manager desk may be 12h shift |
| Critical-action step ceiling | Maximum 2 deliberate steps (open + confirm) | Applies to ECM shadow sign-off, RAO Mark-executed, briefing ack. Standard flows: max 3 steps. |
| Multi-modal alert spec | Haptic ≥200ms + visual ≥500ms full-bleed + audio confirmation only | Audio is NOT the primary channel in any depot/ops surface. See §UX Constraints "Audio-Alert Policy". |
| Colour-blind state encoding | Every multi-state UI element distinguishable without colour | Shape + label + one-character text token in addition to colour. WCAG 1.4.1 non-optional. See §UX Constraints. |

### Maintainability

| Requirement | Target | Context |
|---|---|---|
| IntentPacket schema (landside consumer) | Tracks brain-owned v1.0.0 | Schema evolution coordinated with brain team |
| Alembic migrations | Reversible, rollback path documented before merge | See reliability: write-lock ceiling 30s |
| Recommended Action Object schema | **Schema of record** (2026-05-25): `{ intent_id: uuid, action_type: str, justification: str [mandatory], confidence: float \| null [reserved for Phase 2], confidence_source: enum{model, rule, human_override, null} [reserved], operations: [{ sequence: int, system: str, method: str, payload: {}, expected_outcome_signature: {} }] }`. Versioned — `intent_id` includes schema version prefix. `justification` is mandatory; Kondukt must not generate an object without it. `confidence` field reserved for Phase 2 calibration (defaults to null in MVP; tier selection uses rules-based proxies per §UX Constraints "Three-Tier Recommendation Card Visual Grammar"). `expected_outcome_signature` added so operator-reported outcomes can be programmatically reconciled against downstream-system polling. `operations[]` are ordered by `sequence`. Schema evolution requires a joint landside ADR before any field addition or removal. | Schema evolution does not break in-flight intents |

---

## 11. Functional Requirements

> *MVP is advisory-only — the platform produces recommendations and shadow audit entries, does not dispatch to operational systems, does not record authoritative ECM sign-offs. All FRs below are scoped accordingly. Phase 2 (post-pilot, gated on trust criteria) re-introduces dispatch and authoritative recording; the relevant FRs are flagged with [Phase 2 expansion: ...].*

### Depot Briefing & ECM Compliance

- FR1: ECM Manager can view a pre-arrival briefing for an inbound vehicle, generated from the vehicle's IntentPacket history. Pre-arrival window is triggered by HAFAS scheduled arrival time (via `get_hafas_timetable`) — the IntentPacket schema v1.0.0 carries no position field. Brief is considered "pre-arrival" when opened before the HAFAS scheduled arrival timestamp.
- FR2: ECM Manager can see all open faults on a vehicle, organised by technician discipline (electrical, mechanical, IT)
- FR3: ECM Manager can view a vehicle's blocking state when all technician faults are resolved but ECM 1 authorisation is pending
- FR4: ECM Manager can review a full fault resolution summary and sign off on a vehicle's return to service via a deliberate confirmation action. **MVP: the deliberate confirmation action records a shadow sign-off; the authoritative sign-off remains paper. Phase 2 expansion: same gesture writes authoritative.**
- FR5: ECM Manager can view a post-sign summary of exactly what was committed immediately after sign-off
- FR6: ECM Manager can correct a sign-off record by creating a superseding record, without modifying or deleting the original. Corrections apply identically to the shadow trail in MVP and to the authoritative trail in Phase 2.
- FR7: ECM Manager can export the full audit trail for a vehicle as structured JSON. Export includes paper-trail reconciliation status (matched / unverified / divergent) for every shadow record.
- FR8: Technician can view a role-filtered fault feed showing only faults within their discipline
- FR9: Technician can acknowledge a fault with a structured ACK record. **[NEEDS DECISION before story begins: ACK schema fields, fault code vocabulary (fixed list vs free-entry), whether free-text notes are permitted, and whether a secondary confirmation step is required. Requires input from an ECM Manager or ECM 4 Technician — Abbas to arrange.] Under advisory-only, ACK is a shadow record alongside the existing paper ACK process. The schema design must reflect this dual-record model — every shadow ACK references its corresponding paper-trail entry.**
- FR10: System maintains an append-only, **hash-chained** sign-off event log with IdP-bound identity, vehicle state snapshot, directive reference, and tamper-evidence signature. **MVP: shadow record. Phase 2 expansion: authoritative ECM record.**

### Fleet Manager Triage & Redeployment

- FR11: Fleet Manager can view a real-time triage feed of active fleet disruptions, with cost-impact one-liner per disruption
- FR12: Fleet Manager can review a Recommended Action Object describing assignment of a replacement vehicle to a disrupted service, with step-by-step execution guidance for the existing operational systems (HAFAS). **MVP: review + 'Mark executed' confirmation logs operator action; no dispatch. Phase 2 expansion: one-click approval interlock with platform-side dispatch to HAFAS.**
- FR13: Fleet Manager can view a staleness indicator on any fleet card where IntentPacket data is older than a threshold
- FR14: Fleet Manager can hand over their shift to an incoming Fleet Manager, with a snapshot of all open incidents and optional notes
- FR15: Fleet Manager can view an incoming shift handover summary at the top of the triage view on login
- FR16: Fleet Manager can review the conductor dismissal queue (from brain feed) and promote or discard individual dismissals as alert threshold tuning signals

### Fleet Manager AI Query (Kondukt)

- FR17: Fleet Manager can query the fleet via natural language and receive a structured response grounded in live fleet data
- FR18: Fleet Manager can query predicted maintenance windows and fleet availability over a 48–72h horizon
- FR19: Fleet Manager can search past incidents using semantic similarity and receive ranked results
- FR20: Fleet Manager can see which search mode is active (semantic or keyword fallback) for any query result
- FR21: Query system identifies user intent and selects the appropriate data source to ground the response
- FR22: System surfaces a structured error when a Rail MCP Server tool call times out, with the Fleet Manager's decision logged alongside the data-availability flag. Decision logged includes operator's own action taken (or non-action), not just platform-side state — the shadow audit chain ties data-availability to operator-reported outcome.

### Recommended Action Objects (RAOs)

- FR23: Kondukt can generate a Recommended Action Object conforming to the schema of record (`intent_id`, `action_type`, `justification` [mandatory], `confidence` [Phase 2-reserved, null in MVP], `confidence_source` [Phase 2-reserved], `operations[]` with `sequence`/`system`/`method`/`payload`/`expected_outcome_signature`), bundling operations across MVP-integrated systems (HAFAS only in MVP). `justification` must be non-empty — schema validator rejects objects without it. SAP PM and ServiceNow operations are post-MVP — the schema validator rejects them with a clear "system not integrated in this release" error
- FR24: Fleet Manager can review the full operations list of a Recommended Action Object before confirming — every system the operator must touch and every payload visible — including a step-by-step execution guide for the existing operational system
- FR25: The 'Mark executed' confirmation re-verifies the user's SSO session at the moment of click — not just at app load — and binds the confirmation to the verified identity. **MVP: the confirmation is operator self-report of having executed via existing systems. Phase 2 expansion: same gesture authorises platform-side dispatch.**
- FR26: Confirmed RAO is logged to `recommended_action_audit` with `intent_id`, operator identity, timestamp, operations the operator reported executing, downstream-state polling result per operation, and final outcome status (verified / self-reported-unverified / divergent)
- FR27: Operator-reported execution outcomes that cannot be verified by downstream polling are surfaced as distinct 'Self-reported — outcome unverified' UI state. Persistent unverified outcomes feed Phase 2 trust-criteria assessment
- FR28: The `recommended_action_outbox` table (PostgreSQL) persists all RAOs and their reconciliation state with `idempotency_key UNIQUE`; in-memory queues not used. Survives platform restart without loss.
- FR28a: Kondukt RAO output passes Pydantic schema validation before UI render; on validation failure, the platform retries the Kondukt call once with the schema error appended (`retry-with-error-feedback`); on second failure, the operator sees a human-readable 'Recommendation engine returned a malformed response — manual decision required' state with the raw Kondukt response available on request
- FR28b: Every RAO presented to the operator is rendered in one of three confidence tiers (Tier 1 confident / Tier 2 hedged / Tier 3 forced-two-step) per the rules-based tier selector specified in §UX Constraints "Three-Tier Recommendation Card Visual Grammar". Tier assignment is deterministic and unit-tested; the rule that fired is visible on the card. Phase 2 expansion: rule-based tier selection augmented with calibrated confidence model trained on Phase 1 outcomes.

### Dispatcher Coordination *(Post-MVP — Dispatcher persona deferred from MVP scope)*

- FR29: *(Post-MVP)* Dispatcher can view a relay card with LLM-generated passenger-safe language summarising an incident
- FR30: *(Post-MVP)* Dispatcher can copy the relay card content to clipboard for PA system entry
- FR31: *(Post-MVP)* Dispatcher can forward the relay card to the conductor (push notification dispatched via brain)

### IntentPacket Ingest (Landside Consumer)

- FR32: System consumes IntentPackets from the brain's MQTT broker on `oebb/{train_id}/intent` topic
- FR33: System persists ingested packets to PostgreSQL with idempotent insert on `packet_id`
- FR34: System surfaces ingest gap explicitly — Fleet Manager and Depot Briefing surfaces show staleness when no fresh packet has arrived
- FR35: System handles unknown `agent_state` values (from future brain schema versions) as raw events rather than silently discarding
- FR35a: System maintains `intent_packet_schema_registry` keyed by `schema_version`. On every inbound IntentPacket, diff observed fields against registry. Unknown field → log `intent_packet.unknown_field` event + metric; do NOT drop the packet. Major version mismatch → quarantine + alert. Minor version mismatch → log + accept.

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

**Zone 3 — Kondukt Console + Cross-Zone Interaction**
- FR50: Selecting a vehicle in Zone 1 (matrix or geospatial overlay) opens that vehicle's topology in Zone 2; a fault detected in Zone 1 or Zone 2 generates a Proactive Recommendation Card in Zone 3 in Recommended Action Object format with three-tier confidence visual grammar (see §UX Constraints)
- FR51: Teil-Projektleitung Telematik can view Zone 1, Zone 2, and Zone 3 in a single workspace (the Unified Telematics Control Panel) — no full-screen mode switching required to retain context across zones

**Telematics action flows**
- FR39: Teil-Projektleitung Telematik can monitor OTA rollout progress across the fleet: the platform displays the firmware and software version last reported by each unit, sourced from the most recent IntentPacket received from the brain (which retrieves version state from the onboard Stadler diagnostic system). Lukas can see at a glance which units have confirmed the new patch level, which are on a prior version, and which have not reported in (with last-seen timestamp). ÖBB monitors; rollout execution is performed by Nomad Rail operations or the manufacturer.
- FR40: System detects configuration drift by cross-referencing reported firmware/hardware revisions against the platform's local asset registry; incompatible or drifted units are surfaced for manual review. **MVP: local registry, not SAP asset registry (SAP integration post-MVP).**
- FR41: Teil-Projektleitung Telematik can generate a warranty evidence package as a Recommended Action Object — Kondukt correlates onboard IntentPacket history + landside service telemetry + local asset registry metadata into a structured PDF export attachable to vendor tickets. Export is downloaded by the operator and submitted manually to vendor ticketing systems. The platform does not file warranty claims directly. *(SAP asset metadata cross-reference is post-MVP.)*
- FR52: Teil-Projektleitung Telematik can view a Probebetrieb (trial run) validation surface during the 2-week passenger trial of a newly delivered train: system displays end-to-end network path for each detected fault, proving whether the issue is inside an ÖBB landside server or the manufacturer's onboard hardware
- FR53: Teil-Projektleitung Telematik can convert a predictive edge alert into a pre-emptive workshop reservation via the `reserve_maintenance_slot` MCP tool. **MVP: the recommendation generates the booking proposal; the operator confirms with the workshop via existing communication channels and marks the slot reserved in the platform. Phase 2 expansion: platform-side reservation via workshop scheduling API.**
- FR55: *(Removed — Lastenheft procurement clause export descoped. Format, consumer, and failure threshold were undefined and the feature cannot be evidenced during the pilot. Moved to Vision if reinstated post-pilot.)*
- FR56: Teil-Projektleitung Telematik can view a per-alert-type accuracy metric on every alert ("This alert type has X% accuracy on this route, Y false positives in Z days"); a high-visibility neutral "Dismiss" action converts user dismissals into tuning signals for the system's threshold model, with the dismissal recorded server-side per FR44

### Identity, Access & Audit

- FR42: All compliance actions can be bound to an ÖBB SSO IdP identity, not a local session. **MVP scope:** safety-critical-action surface is descoped (advisory-only); identity binding applies to ECM shadow sign-offs, RAO 'Mark executed' confirmations, technician shadow ACKs, and warranty-evidence export downloads.
- FR43: Each platform surface enforces role-based access. For controls a user's role cannot execute: the control is **rendered but visually disabled** with a tooltip explaining the permission gap (e.g. "Approval requires Fleet Manager role") — it is not hidden. Rationale: in safety-critical ops, operators need to know the full action space to make correct escalation decisions. Server-side authorization on the approval endpoint is the binding enforcement layer; the disabled UI state is defense-in-depth, not the security boundary.
- FR44: System records every acknowledgement, sign-off, dismissal, RAO confirmation, and status change as a server-side write to PostgreSQL, not component state
- FR58: System emits a structured security incident event to ÖBB's security monitoring pipeline for each of the following: authentication anomaly (failed IdP verification on a safety-critical action), unauthorised action attempt (including unauthorised RAO confirmation or shadow sign-off attempt), BYOK key exposure detection (key appears in any logged field), LLM injection detection (output sanitisation layer rejects a Kondukt response). **Event payload supports full NIS2 24-72-30 reporting cadence and CRA Article 14 reporting** — fields include: event type, surface, user identity (if known), timestamp, severity, scope, affected systems, `cra_extension: { product_identifier, cve_ids, actively_exploited, coordinated_disclosure_status }`. ÖBB IT Security owns regulatory filings; the platform owns event emission.

### Resilience & Consistency

- FR45: System provides a defined fallback state when the brain ingest pipeline is unavailable — last-known values displayed with explicit unavailability label
- FR46: System maintains write consistency for cross-boundary operations — partial failures are logged and surfaced for reconciliation, never silently dropped. **MVP: cross-boundary operations are operator-executed via existing systems; partial-failure surfacing applies to the platform's shadow recording layer and downstream-state polling. Phase 2 expansion: applies to platform-side dispatch.**
- FR47: Fleet Manager can view delay-impact attribution for redeployment decisions — actual vs. projected delay-minutes traceable to the Fleet Manager action. Attribution chain in MVP: operator-action-logged-at confirms platform-side timestamp; downstream-system polling confirms outcome; both feed the shadow audit chain.

### Phase 1 Trust & Phase 2 Readiness

- FR59: System captures every RAO with the recommendation→action→outcome chain: `generated_at`, `displayed_to_user`, `user_action_taken` (executed / modified / rejected / ignored), `action_logged_at`, downstream-state polling result, `outcome_verified_at`. This dataset is the Phase 2 trust-evaluation substrate.
- FR60: System reconciles every platform shadow sign-off against the corresponding paper sign-off entry within 24h of the paper signature. Reconciliation status (`matched` / `unverified` / `divergent`) is recorded against the shadow record and surfaced on the audit-trail export.
- FR61: System computes and displays the Phase 2 trust-criteria composite continuously, with separate visible threshold lines for: (a) RAO acceptance rate, (b) shadow audit-trail match rate, (c) qualitative sign-off status from Dr. Renate Fischer, (d) outside-auditor tabletop verdict, (e) pilot-kill-trigger history. All five required for full Phase 2 progression; partial satisfaction triggers narrower Phase 2 scope per §7 Claim 5.
- FR62: System generates a Phase 2 Readiness Report at pilot end (and continuously available throughout pilot) summarising the trust-criteria composite, the RAO outcome dataset, the shadow audit-trail integrity proof, and the outside-auditor verdict — packaged for the Roland Ruisz / Martin Lerch / Dr. Fischer sign-off meeting.

---

*This is the binding capability contract — **65 FRs across 9 capability areas** (was 58/8). Any feature not listed here will not exist in the final product unless explicitly added.*

*Onboard brain capabilities (Conductor App, Hailo inference, IntentPacket emission, departure clearance, shift-aware density, edge resilience) live in `oebb-brain/prd.md` — see `oebb-brain/` directory for the companion PRD.*

---

## 12. Phase 2 Readiness Framework

Phase 2 of the platform — platform-mediated dispatch and authoritative ECM recording — is a **separate post-pilot release** gated on the five composite trust criteria. This section consolidates the framework; cross-references FR59–FR62, §8 Phase 2 Readiness Gates, and §7 Claim 5.

### Trust criteria (composite — all five required for full Phase 2)

1. **RAO acceptance rate** ≥ Roland-confirmed threshold (working assumption ≥70%; modification + rejection rates also tracked)
2. **Shadow audit-trail match rate vs paper** = 100% across the full pilot — every paper sign-off has a matching platform shadow record; persistent divergences disqualify
3. **Dr. Renate Fischer's positive sign-off** on audit-export quality and capability proof (Journey 4 outcome at manday ~15 and continuous review thereafter)
4. **Outside-auditor tabletop verdict** positive (from the manday 20–25 exercise — see §14)
5. **Zero unresolved pilot-kill triggers** in the final 30 days of pilot

### Composite scoring rules

- Each criterion is independently measurable and displayed with its own threshold line in the Phase 2 Readiness dashboard (FR61).
- Criteria 1, 2, 5 are quantitative; 3 and 4 are qualitative go/no-go.
- Full satisfaction → recommend full Phase 2 (HAFAS dispatch + ECM authoritative recording).
- Partial satisfaction → recommend **narrower Phase 2 scope**:
  - e.g., only criterion 2 fails → recommend phased Phase 2: dispatch enabled, authoritative ECM recording deferred until match-rate gap is closed
  - e.g., only criterion 4 fails → defer all Phase 2 until tabletop exercise is re-run with auditor sign-off
  - e.g., criteria 3 + 4 both fail → Phase 2 deferred entirely; pilot extends or programme terminates

### Phase 2 release gate process

1. Pilot manday 120 (~Feb/Mar 2027): Phase 2 Readiness Report auto-generated (FR62).
2. Sign-off meeting: Roland Ruisz, Martin Lerch, Dr. Renate Fischer. Outside auditor present if available.
3. Decision: full Phase 2 / narrow Phase 2 / defer / terminate.
4. If full or narrow Phase 2: Phase 2 contract activated per pre-mapped clause (see §9 open decision "Phase 2 contractual structure").

### Phase 2 commercial framing

Phase 1 contract must include a **pre-mapped Phase 2 expansion clause**. Without pre-mapping, advisory-only looks like a downgrade to the budget holder above Roland. With pre-mapping, it looks like a roadmap. Pricing-model owner is responsible for the clause; legal counsel reviews.

Owner: Roland Ruisz signs off. Martin Lerch (Technical Services) and Dr. Renate Fischer (ECM accountable) co-sign.

---

## 13. Boundary Document Plan

The regulatory boundary documentation for ÖBB has a **3-layer structure**. The structure exists because procurement (ÖBB-Einkauf GmbH), legal counsel, and regulators all read this document — for different purposes. Each register is optimised for its actual reader.

### Layer 1 — `docs/product-boundary.md` — Capability brochure body (SWORD)

Procurement reads this. Opens affirmatively: *"Nomad delivers the following capabilities under the following accountability model..."* Leads with:
- Kondukt BYOK as cryptographic-sovereignty differentiator
- RAOs as auditability moat
- Operational-sovereignty retention (ÖBB executes, Nomad recommends) as the sword that incumbents (Nexxiot, Knorr-Bremse, Wabtec, Konux) cannot match in six months
- Phase 1 → Phase 2 progression as a contractual roadmap, not a future ask

**Bounded affirmative commitments only.** Every affirmative commitment must be bounded by (a) measurable SLA, (b) named operator precondition, or (c) defined exclusion. No unbounded warranties like "explainable AI."

### Layer 2 — Duties-matrix annex (SWORD IN SHIELD'S CLOTHING)

Procurement reads this AND legal validates. **Positive enumeration** of operator duties (not exclusion-based) with named ÖBB roles, verification mechanisms, failure-mode owners. Defends against the **§ 1313a ABGB / Erfüllungsgehilfe trap** — Austrian civil code defaults to *inclusion* of sub-performers in the contracting party's responsibility unless explicitly carved out. Silence = inclusion under ABGB.

Third-party components (Hailo runtime, OS, key-management lib) get explicit treatment — flow-down liability to suppliers OR explicit ÖBB-acceptance documentation. The Kondukt BYOK story helps for keys (demonstrably outside Erfüllungsgehilfe chain), doesn't help for the inference stack.

Co-authored: Winston drafts legal precision; Mary drafts operational language; one document, two hands.

### Layer 3 — Compliance crosswalk annexes (SHIELD)

Lawyers and regulators read these only. Three files:
- `docs/compliance/cra-mapping.md` — CRA Annex I obligation-by-obligation crosswalk
- `docs/compliance/ai-act-article-14-mapping.md` — Art 14(4)(a)–(e) clause-by-clause crosswalk in regulator vocabulary
- `docs/compliance/ikt-sicherheitsrichtlinie-mapping.md` — alignment with ÖBB's own ICT security taxonomy

### Reader-dependent conflict rule

When sword and shield framings tension: **body = sword wins** (procurement reads), **annex = shield wins** (regulators read). Each register optimised for its actual reader.

### Drafting timeline

- **End of current week (week of 25 May 2026):** Confirm Austrian-qualified legal counsel availability. Single critical-path dependency.
- **Mid-June 2026:** Joint drafting session (Winston + Mary + Austrian-qualified legal counsel + ideally someone who has read CRA Annex I). 90-minute working session; co-draft the *claim* and *evidence* sections live; prose follows.
- **Late-July / early-Aug 2026 (~3–4 weeks pre-pilot):** Doc v1.0 to Martin Lerch for informal ÖBB-side review.
- **~2 weeks pre-pilot:** Procurement copy to ÖBB-Einkauf GmbH.
- **Pilot manday 1 (end-Aug or Sept 2026):** Doc baseline locked.

### Advisory-only impact

Advisory-only MVP softens the AI Act Art 14 regulatory cliff (no automated decisions) but *promotes* the boundary doc from optional to load-bearing on the commercial side — the value proposition "you are paying for the recommendation engine with the following accountability split" has to be explicitly articulated. The boundary doc is more important under advisory-only, not less.

---

## 14. Tabletop Exercise Plan

The CRA-readiness tabletop exercise simulates the regulatory incident-response process before pilot end, surfacing gaps while they can still be fixed. **The auditor verdict is one of the five Phase 2 trust criteria** (§12) — without a positive verdict, full Phase 2 progression is gated.

### Timing

- **Pilot manday 20–25** (~late-Sept or late-Oct 2026 under §D13 calendar). Earliest point at which real artifacts exist — first RAO has been confirmed by a real Disponent, shadow audit trail has real entries, on-call rota is real.

### Scenario (advisory-only context)

**"Kondukt recommended an action that a Disponent followed via their existing system; the resulting HAFAS state is wrong. Who is accountable — Nomad's recommendation engine or the human's execution? Where does the audit trail live? What does the 24h NIS2 clock say? What does CRA reporting require?"**

The scenario exercises the CRA + NIS2 + ECM trail intersection AND the ambiguity inherent in advisory-only: did the recommendation cause the wrong outcome, or did the operator's execution?

### Two-run structure (non-negotiable)

1. **Nomad-internal dry-run, ~1 week prior.** Same scenario. Find embarrassing gaps before embarrassing yourselves in front of Roland. If dry-run is smooth, scenario was too soft — make it harder.
2. **Live run with ÖBB.** The one that counts.

### Role assignments

- **Facilitator:** Abbas. NOT an engineer — must call "freeze" and ask "who's making this decision right now, on what authority?"
- **Game Master / scenario injector:** Senior Nomad eng (MCP layer owner). Feeds new information every 15 min. Not to defend the system.
- **Auditor (red team):** **Outside auditor, hired for this — half-day of their time.** Someone who has actually run NIS2 or CRA conformity. The load-bearing investment.
- **Security team:** Martin Lerch as himself; Nomad security lead plays Nomad-side. Awkward "who tells whom first" question must surface.
- **Operator:** Roland Ruisz as himself — the Disponent who filed the report.
- **ECM accountable:** Dr. Renate Fischer as herself — running question: "is this entering my audit trail correctly?"
- **Observers (silent until debrief):** rest of Nomad eng. Take notes. Do not rescue.
- **Scribe:** one Nomad person; only job is logging decisions, timestamps, unanswered questions. Not optional.

If Roland or Renate cannot attend live → postpone, do not run without them.

### Required artifacts (five)

1. **Incident postmortem** (filled for simulated event) — becomes production template
2. **Updated runbook:** who-calls-whom decision tree, NIS2 24h triggers, CRA disclosure triggers, ECM-relevance flowchart. One page each, max.
3. **Findings log:** every "we didn't know what to do" moment, owner, fix-by date. Most valuable, most likely to get skipped — do not let it get skipped.
4. **Auditor's one-pager:** external verdict. What you show ÖBB procurement.
5. **Decision: do we run this again before pilot end?** Yes/no with criteria.

If only the postmortem template comes out, the exercise failed. Findings log + auditor verdict are the load-bearing outputs.

### Failure modes to watch for

- Anyone says "well, in production we'd..." more than twice → process exists only in their head
- 24h NIS2 clock starts; nobody can name *who pushes the button*
- ECM shadow trail and security incident log disagree on timeline; nobody notices until auditor points it out
- "I'd check with..." with no name attached
- The phantom-recommendation scenario gets resolved by "it's probably a UI glitch" and the room moves on (the real-world failure mode — explaining it away)

### Pitch to Roland and Renate

*"We're going to find the holes in your own incident process before they bite you in audit"* — not "help us prepare for CRA." That gets them in the room.

---

## 15. Phase 1 Sprint Scope (revised post-advisory-only)

Engineering must-act-now list, eng-day sized at PoC scale (1–5 trains, 10–20 vehicles). Two sprints.

### Sprint 1 — must-merge-before-pilot-day-1 (~5 eng-days)

| # | Item | Size | Why |
|---|---|---|---|
| 1 | Three-tier recommendation card UI (consume Sally's spec) | 1.0d | Disponent sees something day 1 |
| 2 | PWA wake-from-sleep auth gate (simplified, no SCA) | 0.8d | Disponent will background the tab; without this, day-1 demo breaks on first lunch |
| 3 | RAO schema lock with reserved Phase-2 fields (`confidence`, `confidence_source`, `expected_outcome_signature`) | 0.5d | Unblocks brain workspace coordination |
| 4 | Shadow audit-trail with hash-chain + append-only DB trigger | 1.0d | ECM persona in pilot scope; one-way door on integrity |
| 5 | Brain schema-bump trip-wire (`intent_packet_schema_registry`) | 1.0d | Cross-workspace contract |
| 6 | Stale-data banner state (shared by 3-tier card + PWA wake) | 0.5d | Shared dep |
| 7 | RAO outbox pattern (durable PostgreSQL queue, `idempotency_key UNIQUE`) | 0.5d | Survives platform restart; in-memory queue is a latent data-loss bug |

### Sprint 2 — must-merge-before-PoC-ends (~4 eng-days + 1 policy-day)

| # | Item | Size | Why |
|---|---|---|---|
| 1 | Structured `IncidentReport` endpoint (shared NIS2 + CRA, 24-72-30 cadence) | 1.5d | NIS2 + CRA reporting; one-way door if schema unstructured at audit time |
| 2 | Output schema validation (Pydantic + retry-with-error-feedback) | 1.5d | Table stakes for LLM I/O; 8% → 0.3% unparseable rate |
| 3 | SBOM in build pipeline (cyclonedx or syft) | 0.5d | CRA reporting |
| 4 | `SECURITY.md` + `.well-known/security.txt` + signed releases (cosign) | 0.5d | CRA disclosure process |
| 5 | Vulnerability handling process doc + threat model | 1.0d policy | Owner: Abbas + secops. Written-policy-not-code. |

### Explicit non-scope (NOT building in MVP)

- Live ENISA API integration (stub only — Phase 2)
- Automated CVE scanning in CI (Dependabot is enough; full Snyk/Aqua is post-commercial)
- Public security advisory page (CHANGELOG is enough until we have customers)
- Pen-test report (separate exercise, separate budget, pre-commercial not pre-PoC)
- ISO 27001 / SOC 2 alignment work
- Confidence-based dispatch routing (Phase 2)
- HAFAS write-back / SAP write-back / BOOM integration (Phase 2)
- Platform-side dispatch reconciliation queue (Phase 2 — MVP queue is for operator-action outcomes only)

### Tabletop exercise

Add 2-hour tabletop exercise at pilot manday 20–25 (§14). Outside auditor cost: half-day fee — load-bearing investment.

---
