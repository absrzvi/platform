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
  projectType: Landside fleet management harness for an operator-provided LLM — one product surface with four views (Daily Brief, Neglect-Risk Kanban, Live Board, Chat+Canvas) backed by a PostgreSQL+pgvector data store, Rail MCP server, allowlisted internet egress, and constitution-as-contract enforcement
  domain: Rail operations — regulated landside (EU ECM Directive 2019/779, NIS2, GDPR, AI Act, CRA). Safety-adjacent advisory only; no automated actions in MVP.
  complexity: High — multi-system landside ingest (IntentPackets via brain MQTT, vendor diagnostic feeds, HAFAS read), customer-LLM harness (BYOK with constitution + skills + Rail MCP + allowlisted egress), PostgreSQL+pgvector single-engine store, append-only ECM shadow audit trail with hash-chain integrity, Recommendation Provenance Replay archive as the headline pilot test artifact. MVP is read-only recommendation-only; the harness never takes autonomous action. Phase 2 (post-pilot, gated on trust criteria) reintroduces dispatch.
  projectContext: Greenfield landside harness; receives IntentPacket stream from oebb-brain (onboard) over MQTT, including TCMS data the brain polls via VLAN 7 SNMP from the onboard Stadler diagnostic plane. Vendor telemetry feeds also land here. SAP PM, ServiceNow, BOOM are post-MVP.
  deploymentModel: Single ÖBB instance (PoC); per-operator deployment at fleet rollout
  aiPosture: Advisory-only in MVP — Kondukt (the LLM persona any customer-provided LLM adopts inside the harness) produces validated structured recommendations enforced by per-skill schemas + provenance validators before emission; on validation failure the recommendation is suppressed, not regenerated ("silence is safe"). Human reviews, judges, acts via existing operational systems; the harness records the recommendation, the operator's action, and the outcome as a shadow audit chain reconciled against the paper sign-off process. Phase 2 (post-pilot) introduces platform-mediated dispatch with retained authoritative audit recording, gated on the five composite trust criteria.
  mvpPhase: advisory-only
  phase2Trigger: trust-criteria-met-at-pilot-end
---

# Product Requirements Document — ÖBB Landside Platform

**Author:** AbbasRizvi
**Date:** 2026-05-23 (split into landside-only on 2026-05-25; harness-model + party-mode + operator-corrected reframe on 2026-05-25)
**Companion document:** `oebb-brain/prd.md` — onboard brain PRD (Conductor App, Hailo inference, IntentPacket schema)
**Companion brief:** `_bmad-output/A-Product-Brief/project-brief.md` — the strategic anchor this PRD inherits from
**Companion log:** `_bmad-output/_progress/00-design-log.md` — full reframe trail with revisit triggers

---

## Scope Boundary

This PRD covers the **landside fleet management harness** — a single product surface for ÖBB Flottenmanagement that exposes train telemetry, fleet diagnostics, and recommendation artifacts through an LLM the customer provides (BYOK). The onboard "brain" (Hailo-10H inference, Conductor App PWA, IntentPacket schema, edge resilience) is a separate product documented in `oebb-brain/prd.md`.

The landside harness **consumes** IntentPackets from the brain over MQTT and persists them in a PostgreSQL + pgvector store (single engine, two paradigms: relational/timeseries tables for structured queries + pgvector for embedding-based semantic retrieval). The IntentPacket schema is owned by the brain project. Landside is a downstream subscriber, alongside HAFAS timetable data and Stadler vendor diagnostic feeds. **TCMS data path:** the brain polls onboard VLAN 7 via SNMP and forwards TCMS data landside in the IntentPacket stream — TCMS is brain-mediated, not Stadler-RDC-mediated. SAP PM, ServiceNow, and BOOM are post-MVP.

**MVP execution model.** The harness is read-only and recommendation-only throughout the 6-month pilot. Kondukt (the LLM persona any customer-provided LLM adopts inside the harness, defined by the constitution) produces structured recommendation artifacts that are validated by per-skill output schemas and provenance validators before emission. If validation fails, the recommendation is suppressed (logged with reason); no regeneration loop in MVP. *Silence is safe.* Operators continue to act via their existing systems (paper sign-off, HAFAS direct, radio confirmation, vendor portals). The harness records the recommendation, the operator's action, and the outcome as a shadow audit chain reconciled against the paper trail. Phase 2 — platform-mediated dispatch and authoritative recording — is gated on the five composite trust criteria measured at pilot end and is a separate commercial release.

---

## Executive Summary

ÖBB's Fleet Manager — Roland Ruisz, DI(FH), Flottenmanagement / System- und Betriebstechnik / Team Telematik, with Teil-Projektleitung Telematik for the Cityjet DOSTO Neu and Cityjet Enzo programmes — currently makes high-stakes decisions across fragmented technical systems: vendor diagnostic portals (Stadler, Siemens, Knorr-Bremse), the Nomad Monitoring System (current live view — Internet-on-Board layer only, no other subsystems), Power BI dashboards, ServiceNow tickets, MS Project schedules, Excel reconciliation files. The current fleet visibility tool monitors only IoB; CCTV (VLAN 5), PIS (VLAN 3), intercoms (VLAN 9), AFZ (VLAN 8), audio (VLAN 9), Wi-Fi (10/20/30/31/131/150), switches (trunk), TCMS (VLAN 7 SNMP via the brain), and Stadler RDC (VLAN 200/202) have no unified landside view. Each switch costs cognitive load; each gap is a place a fault goes unattributed. Recurring failures stay invisible across vendor silos. MTTR (sourced from the Stadler diagnostic system — component alert created → cleared) is not surfaced. Daily situational awareness is reassembled from scratch each shift.

**The platform is a harness for the Fleet Manager's LLM.** ÖBB Flottenmanagement plugs in an LLM API key (Anthropic, OpenAI, or other — BYOK; the harness never stores plaintext keys, never logs them, scopes per user, makes token usage transparent to the operator). The harness gives that LLM:

1. **A constitution** — the platform's voice, principles, and operator-facing identity (Kondukt). Any customer-provided LLM adopts the Kondukt identity inside the harness. Model-agnostic.
2. **A library of skills** — domain-specific procedures (read fleet telemetry, summarise a fault history, build a daily brief, draft a vendor-meeting report, generate a downloadable spreadsheet of a recurring-failure pattern).
3. **MCP server access** — the Rail MCP Server exposes structured tools over the PostgreSQL + pgvector data layer (train telemetry per subsystem, depot state, **MTTR computed by the harness from landside-observable Stadler alarm IntentPacket timestamps** — `t_close − t_open`, harness does not read Stadler's internal MTTR field, recurring-failure detection, semantic search across fault history).
4. **Tools for artifact production** — `.xlsx` and `.docx` generation with operator-triggered download.
5. **Allowlisted internet egress** — curated approved-domains list (vendor portals, CENELEC, ERA, EU regulatory texts) for deep-research troubleshooting. No open web. NIS2-defensible.

**The harness produces three MVP views** the Fleet Manager moves between (locked 2026-05-25 after operator correction — Daily Brief is a skill, not a view):

- **Live Board** — fleet-wide moving map + per-subsystem health across every onboard subsystem (CCTV, PIS, intercoms, AFZ, audio, Wi-Fi, switches, TCMS, Stadler diagnostic). Subsystems shown by operator-friendly name, not VLAN ID. Group-pivot both ways: by subsystem ("all CCTV faults across the fleet") OR by train ("everything wrong with 4736-101"). Telemetry-driven; Kondukt annotates with recurring-failure detection and cross-subsystem correlation. **Live Board stays glanceable** — clicking a train redirects to that train's detail card in Kanban (the depth surface).
- **Kanban** — open faults lane-sorted by neglect-risk ("which incident am I ignoring that will bite me by Friday"), NOT by queue order. Kondukt proposes lane placement + neglect-risk score; operator moves cards. **The Kanban train detail card is the depth surface.** Primary disclosure on card expansion: subsystem name, VLAN ID, IP, MAC, switch port, firmware version, time-since-fault. Progressive disclosure (operator clicks to reveal): MTTR (harness-computed from Stadler alarm IntentPacket arrival timestamps), recurring-failure history, last 24h packet trace, vendor portal deep-link. Live Board never carries this depth — it stays glanceable.
- **Kondukt** — the LLM-native deep-research and artifact-authoring surface: chat + side-canvas for in-progress reports and spreadsheets; allowlisted internet research; `.xlsx` and `.docx` export; **skill registry** for routine artifacts. The operator invokes skills by name or shortcut — *Daily Brief* (morning situation report, fleet overnight state, changes since last shift, recommended focus), *Recurring-failure pattern report*, *Vendor meeting prep doc*, *Fleet health summary*, and others. Each skill invocation produces an artifact in the canvas; the operator downloads what they want. Free-form chat is the catch-all for anything not codified as a skill.

The three views together carry the USP: multi-subsystem visibility on Live Board (no competitor produces this) + neglect-risk discipline on Kanban (engagement vs displacement) + LLM-native skill invocation + free-form research in Kondukt (agentic-OS for fleet operations).

**The harness is enforced, not just prompted.** The constitution is partially compiled into deterministic post-processing — per-skill output schemas, citation-trace validators, and an advisory-language linter validate every recommendation synchronously before emission. The validator rejects hallucinated provenance, unsupported claims, imperative-action language, and schema violations. Failed validations are suppressed and logged with reason; suppression is the MVP failure mode (regeneration deferred to Phase 1.5). This is the safety posture that makes a one-person Einzelunternehmen tolerable as a NIS2-scope supplier on a safety-adjacent system: the harness owns correctness, not the LLM.

**The pilot test artifact: Recommendation Provenance Replay.** Every recommendation the harness emits is stored as a triplet at emission time — Envelope (consist, action, advisory class, confidence band, timestamp, harness version, model+version, skill invoked) + Provenance Bundle (IntentPackets consumed, MCP queries issued, raw LLM output before parsing) + Validation Receipt (schema check, citation-trace check, advisory-class check, disposition: emitted / suppressed). Six months later, any recommendation in the pilot corpus replays deterministically via an auditor-runnable CLI from artifacts alone — no LLM session required. Aggregate pass criterion: ≥99% replay-verify cleanly; zero recommendations exist without a triplet; zero advisory-only language breaches. This is what Renate Fischer shows the outside auditor at the manday-20–25 tabletop, and what differentiates the harness from a chat-log wrapper.

**Secondary recommendation-consumer roles:** ECM Manager (pre-arrival fault brief; paper sign-off remains authoritative in MVP; harness records a shadow record reconciled against paper) and ECM 4 Technician (role-filtered fault feed; structured shadow ACK alongside existing paper process). The Dispatcher (Fahrdienstleiter) is deferred from MVP.

Every decision is made by a human and executed via the operator's existing systems. The harness records the recommendation, the operator's action, and the outcome as shadow audit evidence — proving the harness's capability to be the authoritative record once trust is established.

### What Makes This Special

**The harness as the product, not the LLM.** Customer brings the model — Anthropic, OpenAI, future entrants. Harness brings the chassis: constitution + skills + Rail MCP + pgvector data layer + allowlisted egress + validated structured output + provenance archive + ECM-grade shadow audit trail. No inference-resale economics; no model-churn risk; no vendor lock-in to a specific LLM provider. The customer's data stays close to the LLM provider relationship they already have. The chassis alone is not the moat; the *fit* — regulated-rail recommendation logic + ECM 2019/779 reconciliation + Roland's operational decisions encoded into the constitution over the pilot — emerges from usage and is what compounds.

**Multi-subsystem visibility no competitor produces.** Nomad Monitoring sees only IoB. Stadler sees only their own diagnostic plane. Knorr-Bremse sees brakes only. The harness sees every IP-addressed subsystem in one context, correlated, with per-train and per-subsystem pivots.

**ECM audit trail** — not a log, but the regulatory compliance record under EU 2019/779. Append-only, IdP-bound, HMAC-signed, hash-chained, with `supersedes_id` for corrections. In MVP, the trail shadows the paper sign-off process; in Phase 2, it becomes the authoritative regulatory compliance record under EU 2019/779.

**Kondukt Recommended Action Objects (RAOs)** — the landside differentiator. Where competitor dashboards show data and require humans to translate it into actions across 4–7 systems, this platform's Kondukt co-pilot produces structured multi-system action proposals as readable recommendation cards with step-by-step execution guidance. The Disponent reviews the recommendation, executes via existing systems, and logs the action taken back into the platform — building the recommendation→action→outcome dataset that gates Phase 2 dispatch capability.

**Vendor warranty leverage** — the platform's end-to-end network path visibility resolves "is it our landside or the manufacturer's hardware" disputes during the Probebetrieb commissioning phase, where a single telematics breakdown can freeze formal sign-off. This is a non-obvious commercial wedge for the Telematics Sub-Project Manager persona.

### Pilot Success Criteria

| Criterion | Target | Evidence method |
|---|---|---|
| **Recommendation Provenance Replay archive integrity** | ≥99% of emitted recommendations replay-verify cleanly via the auditor-runnable CLI; zero recommendations exist without a complete triplet (Envelope + Provenance Bundle + Validation Receipt); zero advisory-only language linter breaches | Headline pilot test artifact. Replay archive exported at pilot end (`pilot-corpus-2026-08-to-2027-02.tar.zst`) and demonstrated live at the manday-20–25 tabletop with the outside auditor running the Replay CLI on their own laptop. |
| **High-confidence inference rate** | ≥ Roland-confirmed threshold (working assumption ≥80% of emissions HIGH or MEDIUM confidence; LOW-confidence DISABLE rule remains in force per the 2026-05-25 party-mode lock) | Confidence distribution per skill per week, tracked across the pilot. If LOW-confidence rate exceeds 15% sustained over any 72h window, that's a calibration trip-wire (memory: `feedback_no_over_designed_systems` + 2026-05-25 design-system lock). Pilot pass criterion is the inverse signal — that the harness is producing actionable advice at scale, not constant DISABLEs. |
| **Train-to-landside-DB mapping correctness** | 100% of trains in pilot scope correctly mapped in the harness's local registry — every IntentPacket for train X lands in train X's data, every component MAC/IP in train X is registered against train X. Verification: weekly automated reconciliation between IntentPacket-claimed `train_id` and the brain's per-train MQTT topic; zero misroutes tolerated. | Misrouted data destroys the trust premise — a recommendation cites "fault on train 4736-101" but the underlying IntentPacket came from train 4736-103, the entire Provenance Replay archive becomes suspect. Mapping integrity is a one-way door at the data layer. |
| Recommendation acceptance rate | ≥ Roland-confirmed threshold (working assumption ≥70%) within first 30 pilot mandays, **measured per-view** (Live Board annotations / Kanban lane proposals / Kondukt skill invocations / Kondukt free-form chats) not aggregated | Acceptance + modification + rejection rates catalogued per recommendation. Per-view measurement prevents survivorship bias where one surface's adoption masks another's under-use. |
| ECM pre-arrival review | >80% of arrivals | IntentPacket generated time vs HAFAS scheduled arrival time vs ECM brief opened time. Sign-off itself is recorded on paper; the platform's shadow record is verified against the paper trail post-shift. **Pre-arrival trigger = HAFAS scheduled arrival (MVP default — IntentPacket schema v1.0.0 carries no GPS/position field).** |
| Fleet Manager decision latency reduction | Qualitative observation + measurable reduction vs baseline where instrumentable | Time-to-decision for disposition + redeployment calls; baseline early pilot mandays, measured later pilot mandays. Stays adoption-dominated pre-pilot — no euro figures committed (we don't have baselines yet; pilot is the instrument that produces them). |
| Shadow audit-trail match rate | 100% of paper sign-offs during pilot have matching platform shadow record | Field-for-field reconciliation within 24h of paper signature; divergences flagged and tracked. Reconciles 1:1 with the Recommendation Provenance Replay archive. |
| Phase 2 trust threshold met | Composite of acceptance rate + 100% shadow-match + Renate Fischer sign-off + outside-auditor tabletop verdict on the Provenance Replay archive + zero unresolved pilot-kill triggers in final 30 days | All five required for full Phase 2; partial satisfaction triggers narrower Phase 2 scope |
| Phase 2 readiness recommendation | Within 6 months of pilot start; Phase 2 readiness assessment generated continuously from the trust-criteria composite | Composite pilot health score AND Phase 2 trust-criteria status updated weekly with visible threshold lines for both |

### Project Classification

- **Project Type:** Landside fleet management harness for an operator-provided LLM — one product surface with four MVP views (Daily Brief, Neglect-Risk Kanban, Live Board, Chat+Canvas), plus depot-side recommendation-consumer surfaces (PWA brief for ECM Manager; technician ACK feed) and audit-export surface (ECM Lead)
- **Domain:** Rail operations — regulated landside (EU ECM Directive 2019/779, NIS2, GDPR, AI Act, CRA)
- **Complexity:** High — landside IntentPacket ingest + multi-vendor diagnostic feeds + customer-LLM harness (BYOK with constitution-as-contract + skills + Rail MCP + allowlisted egress) + PostgreSQL+pgvector single-engine store + append-only ECM shadow audit trail + Recommendation Provenance Replay archive
- **Project Context:** Greenfield landside; consumes IntentPackets from oebb-brain (including TCMS via VLAN 7 SNMP, brain-mediated)
- **Deployment Model:** Single ÖBB instance (PoC); per-operator deployment at fleet rollout
- **AI Posture:** Advisory-only in MVP, **enforced not just stated**. Kondukt — the LLM persona any customer-provided LLM adopts inside the harness — produces structured recommendation artifacts validated by per-skill schemas + provenance validators before emission. Failed validations suppress; no regeneration loop in MVP ("silence is safe"). Human reviews, judges, acts via existing operational systems; harness records the recommendation, the operator's action, and the outcome as a shadow audit chain. Phase 2 (post-pilot) introduces platform-mediated dispatch + authoritative recording, gated on the five composite trust criteria.

---

## Success Criteria

### User Success

| Persona | Success moment | Measurable signal |
|---|---|---|
| **Fleet Manager (Roland Ruisz)** | Opens the harness in the morning, runs the *Daily Brief* skill in Kondukt — the morning situation report appears in the canvas, with last night's fleet state and a recommended focus. Throughout the day, Live Board surfaces fleet-wide multi-subsystem health on a moving map (he clicks a train to redirect to its detail in Kanban). The Neglect-Risk Kanban surfaces which open faults are decaying without attention and carries the full per-component detail (VLAN, IP, MAC, firmware, MTTR, recurring-failure history) on each train's detail card. Kondukt lets him invoke skills (recurring-failure report, vendor meeting prep, fleet health summary) or chat free-form for ad-hoc research — and download docx/xlsx artifacts. | *Daily Brief* skill invoked ≥80% of shift starts. Kanban lane-proposal acceptance ≥70%. Live Board session count + pivot-axis usage. Kondukt skill invocation counts + free-form chat counts + artifacts downloaded. All measured per-view, not aggregated. |
| Dispatcher (Fahrdienstleiter) | Post-MVP — persona deferred from MVP scope | — |
| ECM Manager | Arrives at depot having already reviewed the recommendation-consumer fault brief; signs on paper per existing process; verifies that the harness's shadow record matches the paper sign-off before leaving the bay | >80% of arrivals with brief opened before vehicle arrives; 100% shadow-trail match against paper |
| Technician (ECM 4) | Logs a resolution ACK that appears in the ECM Manager's pre-arrival brief before the vehicle docks — no phone call (paper handover still required per existing depot process; harness shadow record runs alongside) | Fault reassignment rate per role per vehicle; % of faults with ACK logged in harness before vehicle arrival (shadow signal, not authoritative); % of harness shadow ACKs matching corresponding paper records |
| ECM Lead (Renate Fischer) | At manday ~15 reviews the audit-trail export + the Recommendation Provenance Replay archive integrity; runs the Replay CLI on a sample recommendation; signs off (or doesn't) on the harness's audit-trail capability as the path to Phase 2 authoritative recording | Qualitative sign-off; sign-off is one of the five composite Phase 2 trust criteria. Quantitative input: shadow-vs-paper match rate sustained at 100% across the full pilot. |

### Business Success

- **Adoption-dominated scorecard (locked 2026-05-25 post-operator-correction).** We do not have baseline euro figures pre-pilot — committing to euro thresholds would be making numbers up. The pilot is the instrument that produces them. Pre-pilot, the scorecard is acceptance rate + qualitative trust signals + sign-offs + the five composite trust criteria. Post-pilot, the harness's recommendation→action→outcome chain dataset is what generates the euro-denominated story for Phase 2.
- **Recommendation acceptance rate ≥70% within 30 mandays** — primary metric, measured per-view (Live Board annotations / Kanban lane proposals / Kondukt skill invocations / Kondukt free-form chats) not aggregated.
- **High-confidence inference rate ≥80% HIGH+MEDIUM** (working assumption; Roland-confirmed pre-pilot) — the inverse of the LOW-confidence DISABLE trip-wire; signal that Kondukt is producing actionable advice at scale.
- **Train-to-landside-DB mapping correctness = 100%** — zero misroutes between IntentPacket-claimed `train_id` and harness data path. Verified weekly. One-way door at the data layer.
- **Recommendation Provenance Replay archive integrity** — ≥99% replay-verify cleanly; zero recommendations without triplets; zero advisory-only language breaches. Headline buyer-facing artifact.
- **Shadow audit-trail completeness** — 100% of paper ECM sign-offs during the pilot have a corresponding harness shadow record. Mismatches surface within 24h and are reconciled before pilot manday close.
- **Phase 2 readiness recommendation** — composite trust-criteria score (five criteria, see §12) crosses threshold within 6 months. Presented to Roland Ruisz, Martin Lerch, Dr. Renate Fischer. Sign-off binary (3/3 sign-offs + clean Replay archive = extend; anything less = don't).
- **Qualitative outcomes captured for post-pilot ROI narrative (not pilot pass criteria):** prevented delay-minutes counterfactual (faults detected before service disruption, cross-referenced against industry-standard or ÖBB-acknowledged mean delay per fault class — best-effort, not audit-grade); recurring-failure detections surfaced by Live Board + Chat+Canvas that paper process did not catch; Chat+Canvas-authored vendor-facing reports Roland used in real escalations; ECM audit hours at Renate's quarterly review compared to her pre-pilot baseline.

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
| ECM shadow audit write failure | Unresolved >24h | Halt harness-side shadow recording; ECM paper process continues unaffected; flag to Martin Lerch for infrastructure review |
| Shadow audit-trail divergence | >2 mismatches per pilot week between paper and harness records | Harness-side root-cause investigation; if divergence persists, Phase 2 readiness assessment downgrades |
| **Recommendation Provenance Replay validator failure** | Any recommendation emitted without a complete Validation Receipt; any recommendation in the corpus that fails replay-verification; any advisory-only language linter breach | **Immediate suppression of the affected skill until root cause identified.** Validator failures are existential — they violate the constitution-as-contract guarantee. Halt emission for the affected skill, surface "Skill suspended — provenance check failed" to operator, root-cause within 24h. Repeated validator failures across multiple skills = pilot pause pending investigation. |
| **BYOK LLM unreachable** (customer-provided key invalid; provider 429/5xx sustained; rate limit exceeded) | >24h sustained | Harness transitions to degraded mode: Live Board and Kanban remain functional (telemetry-driven, no LLM in the path); Kondukt shows "Unavailable — your LLM provider returned {status}. Token usage may be exhausted, or your API key may be invalid." Kondukt skill invocations (Daily Brief, Recurring-failure report, etc.) suppressed until provider reachable. Operator falls back to current practice. |
| **Train-to-landside-DB mapping divergence** | Any detected mismatch between IntentPacket-claimed `train_id` and the brain's MQTT topic the packet arrived on; OR any registered component (MAC/IP) appearing on a train it is not mapped to | Halt harness data writes for the affected train pending reconciliation. Mapping integrity is one-way door at the data layer — silent misrouting would invalidate the entire Provenance Replay archive. Root-cause within 24h. |
| pgvector semantic search unavailable | Keyword fallback active >48h continuous | Flag to Martin Lerch; infrastructure review required |
| Rail MCP Server tool failure | Any single tool unavailable >12h | Surface "Data source unavailable" to operator; review by Nomad ops |
| Landside ingestion gap from brain | >72h without IntentPacket sync on any active train | Coordinate with brain team — root cause may be onboard or transport |

### Measurable Outcomes (Pilot Evidence Targets)

| Evidence | Method | Target |
|---|---|---|
| **Recommendation Provenance Replay archive** | Every recommendation persisted as a triplet (Envelope + Provenance Bundle + Validation Receipt) at emission time; archive exported at pilot end; auditor-runnable CLI verifies the archive on the auditor's laptop | ≥99% replay-verify cleanly; zero recommendations without triplets; zero advisory-only language breaches |
| **High-confidence inference rate** | Confidence distribution per skill per week tracked across the pilot; HIGH + MEDIUM + LOW counts emitted alongside the triplet. LOW-confidence DISABLE rule (party mode 2026-05-25) remains in force — LOW emissions are visibly distinct and Modify is the only execution path. | ≥80% HIGH or MEDIUM (working assumption; Roland-confirmed pre-pilot). LOW-confidence rate >15% sustained over 72h = calibration trip-wire. |
| **Train-to-landside-DB mapping correctness** | Weekly automated reconciliation: every IntentPacket arrived on `oebb/{train_id}/intent` matches `IntentPacket.train_id` field; every registered MAC/IP belongs to its mapped train. Surfaced as a `mapping_health` metric in the harness ops dashboard. | 100% — zero misroutes tolerated. Mapping divergence is a pilot-kill trigger. |
| ECM trail completeness | Log IntentPacket generated time vs HAFAS scheduled arrival time vs ECM open time vs sign-off time. Pre-arrival = ECM brief opened before HAFAS scheduled arrival. | >80% pre-arrival review rate |
| Recommendation acceptance rate (per-view) | Acceptance action log per recommendation, segmented by `view` (Live Board annotation / Kanban lane proposal / Kondukt skill invocation / Kondukt free-form chat); modification rate tracked separately — partial-follow signals close-but-not-trusted | ≥70% per-view acceptance, rejection reasons catalogued |
| Recommendation→action→outcome chain | Capture every recommendation with: generated_at, displayed_to_user, user_action_taken (executed/modified/rejected/ignored), action_logged_at, downstream-state polling result, outcome_observed_at | This is the dataset Phase 2 trust evaluation depends on; reconciles 1:1 with the Replay archive |
| Audit trail usage correlation | Sign-off timestamp vs briefing-read timestamp — did reading the brief change the outcome? | Qualitative + quantitative |
| Per-view engagement | Live Board session count + pivot-axis usage (subsystem-pivot vs train-pivot) + click-through-to-Kanban rate; Kanban card-detail opens; Kondukt skill invocation counts (Daily Brief, Recurring-failure, Vendor meeting prep, Fleet health, etc.) + free-form chat counts + artifacts downloaded + allowlisted-egress fetches | Adoption-dominated; absolute numbers per view |

**Pilot calendar anchor:** Pilot manday 1: **end of August 2026 (target) or September 2026 (slip-safe target).** Hailo hardware delivery is the gating dependency. Pilot schedule is tracked in **mandays**, not calendar days. Type Approval achieved April 21, 2026; pre-pilot critical path (Apr–Aug 2026) includes whole-application build and Nomad Rail sole-proprietary company formation. "Baseline mandays" = first ~10 mandays of active harness use; "measurement mandays" = mandays 20–30. Roland Ruisz sign-off meeting targeted around pilot manday 15 (mid-Sept / mid-Oct 2026). Phase 2 readiness recommendation within 6 months of pilot manday 1 (Feb/Mar 2027). **Pre-pilot gate:** confirm Hailo-8 and Hailo-10 order placed and delivery date confirmed before pilot manday 1 is set.

**Baseline methodology (prevented delay-minutes) — approximation approach:** A precise historical baseline from ÖBB's Power BI Diagnoseplattform or equivalent is not expected to be available for the pilot. The prevented delay-minutes figure will be calculated as an approximation: fault detected by platform → no service disruption → cross-reference against industry-standard or ÖBB-acknowledged mean delay for the same fault class. Roland Ruisz to agree on the approximation methodology and the fault-class reference values before pilot manday 1. This is a best-effort evidence narrative for the ROI conversation, not a statistically rigorous baseline — the goal is a credible order-of-magnitude figure, not audit-grade precision.

**ECM completeness gap (80% → 95%):** The 80% pre-arrival review target is platform-delivered. The 95% business success target requires process change (ECM Manager opens brief before leaving for depot, not on arrival). Martin Lerch owns the process workstream. **Phase 1 does not change the paper sign-off process; Phase 2 replaces it.**

---

## Product Scope

### MVP — Minimum Viable Product

**One harness, three Fleet Manager views + one recommendation-consumer surface (Depot PWA) + one audit-export surface (ECM Lead). Constitution-as-contract enforcement on every LLM emission. Recommendation Provenance Replay archive as the headline pilot test artifact.**

**Fleet Manager surface (the harness — Roland Ruisz's primary product):**

1. **Live Board** — fleet-wide moving map + per-subsystem health, glanceable. Subsystems shown by operator-friendly name only (no VLAN IDs surface here): CCTV, PIS (interior displays + side + front destination displays), AFZ passenger counter, intercoms (incl. PRM variants), audio amplifiers + ADU, Wi-Fi access points, FIS switches, Frontkupplung, energy meter, Service FIS/CCTV diagnostic, Zugfunk, TCMS (received from the brain), Stadler diagnostic plane (RDC + OBS), Firewall RCU. **Group-pivot both ways:** by subsystem ("all CCTV faults across the fleet") OR by train ("everything wrong with 4736-101"). Time-since-fault per component; MTTR sourced from Stadler diagnostic system (alert created → cleared) read by the harness. Recurring-failure detection annotations from Kondukt with provenance. Both in-service trains and depot trains visible. **Click a train → redirects to that train's detail card in Kanban** — Live Board does not expand inline; it stays glanceable, Kanban is the depth surface. The cross-subsystem visibility no competitor produces.

2. **Kanban (Neglect-Risk Triage)** — open faults lane-sorted by *neglect-risk* ("which incident am I ignoring that will bite me by Friday"), NOT by queue order. Lanes: New → Investigating → Recommendation Pending → Cleared. Kondukt proposes lane placement + neglect-risk score with provenance; operator moves cards. **The train detail card on Kanban is the depth surface.** Operator-friendly subsystem names on the Kanban card summary. **Primary disclosure** when the card expands: subsystem name, VLAN ID (5/3/8/9/10/20/30/31/131/150/trunk/12/2/7/200/202), IP address, MAC address, switch port, firmware version, time-since-fault. **Progressive disclosure** (operator clicks to reveal additional detail per component): MTTR computed by harness from Stadler alarm IntentPacket timestamps, recurring-failure history, last 24h packet trace, vendor portal deep-link. Keeps the expanded card readable; surfaces the heavy diagnostic depth only when the operator asks for it. Continuous all-day surface.

3. **Kondukt** — the LLM-native deep-research + artifact-authoring surface. Chat + side-canvas for in-progress reports and spreadsheets. The operator either **invokes a skill** by name or shortcut, or chats free-form. Kondukt can query the pgvector + relational store via Rail MCP tools, use allowlisted internet egress for deep research (Stadler / Siemens / Knorr-Bremse vendor docs, CENELEC, ERA, EU regulatory texts; no open web), and produce `.xlsx` and `.docx` artifacts the operator downloads. Every chat turn carries a Validation Receipt; downloaded artifacts are working artifacts (not audit records). This is the agentic-OS surface that distinguishes the harness from a dashboard.

**Kondukt skill registry (MVP — versioned in-product):**

| Skill | What it produces | Operator invocation |
|---|---|---|
| **Daily Brief** | Morning situation report — fleet overnight state, changes since last shift, recommended focus, top incidents needing triage. The first-kill artifact: which of last night's anomalies actually changes today's plan. | `Daily Brief` skill button or chat shortcut at session start |
| **Recurring-failure pattern report** | Cross-fleet pattern analysis (e.g. "Bus controller memory utilisation across DOSTO Neu for the last 90 days") with provenance citations to IntentPackets | Skill button or chat prompt |
| **Vendor meeting prep doc** | Vendor-facing report (e.g. for Stadler / Siemens / Knorr-Bremse escalations) with allowlisted vendor-portal references; downloadable as `.docx` | Skill button or chat prompt |
| **Fleet health summary** | Weekly fleet-wide health overview (subsystem reliability, MTTR trends, recurring failures) — exportable for Roland's internal reports | Skill button or chat prompt |

Additional skills emerge over the pilot from operator usage and are added to the registry via versioned ADR. All skills emit recommendations through the same constitution-as-contract enforcement layer; all skill invocations produce Recommendation Provenance Replay triplets.

**Recommendation-consumer surfaces (downstream of the harness):**

5. **Depot Briefing PWA** — mobile-first. Role-filtered fault feed (ECM Manager / Electrical / Mechanical / IT), ECM 1 blocking authorisation state, pre-arrival brief generated from IntentPackets + harness-summarised fault history. **ECM 1 shadow sign-off via a simple click (MVP)** — paper sign-off remains authoritative; harness records the shadow. Deliberate-confirmation gestures (hold-to-record, Mark-executed) are descoped from MVP 2026-05-25; hold-to-record gesture spec is preserved in the design-system reference as a Phase 2 candidate when the platform's sign-off becomes authoritative. Technician structured ACK alongside paper ACK. BOOM integration descoped — structured JSON export only.

6. **ECM Audit Export surface** — Renate Fischer's review interface. Full audit-trail export (JSON), paper-vs-shadow reconciliation status, `supersedes_id` correction view, **Recommendation Provenance Replay archive integrity demonstration** (the auditor-runnable CLI runs against the archive in front of her at manday ~15). Reconciliation is the load-bearing trust artifact.

**Cross-cutting infrastructure (MVP):**

- **PostgreSQL + pgvector** — single landside data store. All IntentPackets + all subsystem telemetry + all recommendations + all shadow audit entries + all Recommendation Provenance triplets land here. One engine, two paradigms (relational/timeseries + embeddings). NIS2-defensible single-instance posture.
- **Rail MCP Server** — `rail-mcp-server/` subpackage, HTTP MCP endpoint. Exposes structured tools over the data layer to the customer-provided LLM. Tools include: fleet telemetry per subsystem, depot state, MTTR query against Stadler RDC data, recurring-failure detection, fault history semantic search, neglect-risk scoring inputs, Chat+Canvas artifact builders (xlsx/docx generators). Spec frozen in `rail-mcp-server-spec-2026-05-23.md`; updated to harness model (the previous 9-tool list narrowed to the 4-view set).
- **Constitution + Skills registry** — versioned in-product; operator can inspect what Kondukt is told to be. No hidden prompting. Each skill carries its output schema, citation-trace rule, and advisory-language linter — partially compiled into deterministic post-processing.
- **Recommendation Provenance Replay archive** — append-only, content-addressed blob store + `recommendation` Postgres table with foreign keys to Provenance and Validation Receipt. Auditor-runnable Replay CLI (~400 LOC Python, single binary) verifies the archive against the validator without an LLM in the loop.
- **ECM shadow audit trail** — `ecm_signoff_events` table, append-only, IdP-bound, HMAC-signed, hash-chained, with `supersedes_id` corrections. MVP shadows paper; Phase 2 promotes to authoritative. Reconciles 1:1 with the Recommendation Provenance Replay archive.
- **IntentPacket landside ingest** — REST + SSE from oebb-brain MQTT broker (`oebb/{train_id}/intent`); idempotent insert on `packet_id`; schema-version drift detection.
- **BYOK key vault** — encrypted at rest, per-user scope, never logged, never transmitted to harness vendor logs. Token usage transparent to the operator.
- **Allowlist egress proxy** — curated approved-domains list (vendor portals, CENELEC, ERA, EU regulatory texts). Every fetch logged. No open web.

### Growth Features (Post-MVP)

- Regeneration loop on validator failure (deferred from MVP; suppression is the MVP failure mode)
- Drift Visibility module (moved from MVP to ECM module — quarterly cadence; firmware/configuration version comparison across the fleet)
- Vendor warranty recovery workflow (vision tier — detection capability available via Chat+Canvas in MVP; the structured-export-to-vendor workflow is Phase 1.5 candidate)
- BOOM API integration when API contract confirmed with ÖBB IT
- ServiceNow ITSM bidirectional integration
- SAP PM deeper integration — work order creation from harness recommendations (Phase 2 expansion)
- Multi-tenant deployment — credential isolation, per-operator Rail MCP Server instances
- Dismissal signal arbitration queue (conductor dismissals from brain feed into harness threshold tuning)
- ÖBB SSO IdP integration (MVP uses Keycloak with manually-created users)

### Vision (Future)

- Push-based fleet intelligence — Kondukt proactively surfaces patterns without query
- ECM audit trail integration with ÖBB's internal compliance documentation system
- Lastenheft procurement requirements export — clauses generated from cumulative operational evidence
- Multi-operator rail network deployment (non-ÖBB carriers)
- Phase 2 dispatch capability — platform-mediated HAFAS rotation updates + SAP work order creation, with authoritative ECM recording, gated on five trust criteria
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
- **Sign-off gesture in MVP — simple click.** The ECM 1 shadow sign-off in MVP is a single click ("Record sign-off") that writes the `ecm_signoff_events` shadow row. No deliberate-confirmation gesture (hold-to-record, countdown, atomic-write-at-expiry) in MVP — the operator's intent is captured by the click + IdP-bound identity at the moment of click + the immutability of the resulting row (append-only DB trigger). After write, record is permanent — no undo. Idempotency key generated client-side; retries safe (409 Conflict treated as success). **Descoped from MVP 2026-05-25:** the 3-second-hold → 5-second-countdown → atomic-write-at-expiry gesture described in `design-artifacts/ecm-signoff-interaction-spec.md`. That spec is preserved in the design-system reference as a **Phase 2 candidate** — when the platform's sign-off becomes authoritative under EU 2019/779, the deliberate-confirmation gesture is reconsidered.

---

## User Journeys

### Journey 1 — ECM Manager: "The Pre-Arrival Brief" (Primary — Success Path)

**Meet Klaus.** Klaus is an ECM 1 Manager at Wien Westbahnhof depot. He is accountable under EU Directive 2019/779 for every vehicle that returns to service. He arrives cold — he was not on shift when the train came in last night. Unit 4722 is in Bay 3.

**Before the harness.** Klaus walked to Bay 3, asked the technician what happened, read a paper fault log, and made a judgment call. If he missed something and the vehicle failed in service, the liability was his and the paper trail was thin.

**Now.** At 05:50, fourteen minutes before Unit 4722 docks, the harness generates a depot briefing from the journey's IntentPackets (received from oebb-brain over MQTT). Klaus opens the Depot Briefing PWA on his phone. Role: ECM Manager. He sees: two faults sorted by severity. CRITICAL: Door motor fault, Car 4 Door 2L — resolved by conductor at 07:14, replacement part fitted at 03:22 by Electrical team. WARNING: SNMP camera firmware version mismatch on VLAN 5 (Überw. Kamera A7 — IP 172.18.192.138) — acknowledged by IT technician, deferred to next maintenance window with reason logged. Each finding cites its IntentPacket source. The brief itself is a recommendation artifact: the validator passed it through (schema OK, citations OK, advisory-only language OK); its triplet sits in the Provenance Replay archive.

**The climax.** Klaus reviews the resolution chain on the harness — detection → conductor ACK → technician assignment → part fitted → test result. He walks to Bay 3 and confirms what he has read with the technician on site. He signs off **on paper** per ÖBB's existing ECM 1 process. Then he taps "Record sign-off" in the PWA — a single click in MVP — and the `ecm_signoff_events` shadow record writes: his IdP identity, UTC timestamp, full vehicle state snapshot at sign-off, directive reference EU 2019/779. The paper is the sign-off; the harness's record is the shadow.

**The new reality.** Unit 4722 clears to service at 06:08. Klaus has the paper signed and a digital shadow that proves — if the paper were ever lost or disputed — what he actually authorised. Over the 6-month pilot Klaus and his peers build a dataset of paper-and-shadow pairs that proves the harness's audit-trail capability is sound. That dataset *plus* the Recommendation Provenance Replay archive together are the evidence Phase 2 trust criteria rests on.

*Capabilities revealed: Pre-arrival brief generation, role-filtered fault feed, ECM 1 blocking state, ECM shadow sign-off (simple click — MVP gesture), append-only shadow sign-off log, full resolution chain view, paper-trail reconciliation.*

---

### Journey 2 — Fleet Manager: A Day in the Harness (Operations — Success Path)

**Meet Roland.** Roland Ruisz, DI(FH) — Flottenmanagement / Team Telematik / Teil-Projektleitung Telematik for Cityjet DOSTO Neu + Enzo. The Fleet Manager. One role, two programme hats. He has opened the harness in a browser on his ÖBB workstation. His Anthropic API key is already configured in BYOK — he set it up on day one of pilot manday.

**06:30 — Kondukt: Daily Brief skill.** Roland opens the harness. The Live Board is the default landing surface — a quick scan, fleet looks ordinary. He clicks into Kondukt and invokes the *Daily Brief* skill. Kondukt runs the skill: pulls overnight IntentPackets via Rail MCP, runs the recurring-failure detection skill against the last 24h, validates the output through the constitution-as-contract layer (per-skill schema + provenance + advisory-language linter), and emits the brief into the canvas. The brief says: *"Overnight: 3 trains with intermittent reservation-screen blackouts on the DOSTO Neu fleet — pattern across all three is a memory-utilisation crossover on the bus controller at 94%; recommend prioritising bus-controller diagnostic pull from Stadler today. 2 CCTV firmware mismatches on Cityjet Enzo (units 4736-103 and 4736-108) — both below threshold; defer. 1 PIS display on 4736-101 unreachable for 14 hours — surface flagged for depot triage."* Each finding cites its IntentPacket source. Every claim has a confidence pill (2 HIGH, 1 MEDIUM in this brief). Roland reads, agrees with the prioritisation. The brief is logged as a recommendation triplet — Envelope + Provenance Bundle + Validation Receipt — in the Recommendation Provenance Replay archive. *First-kill decision: which of last night's anomalies actually changes today's plan. Made in 4 minutes.*

**07:00–10:00 — Kanban.** Roland moves to the Kanban view. Lanes: New → Investigating → Recommendation Pending → Cleared. Cards are open faults, lane-sorted by *neglect-risk* — not by queue order, by which incident is decaying without attention. The bus-controller pattern from the Daily Brief is at the top of the New lane with a HIGH neglect-risk score and a Kondukt-proposed move to Investigating. Roland accepts the move. He clicks the card to expand it — the **train detail card** opens with primary per-component disclosure: subsystem, VLAN ID, IP, MAC, switch port, firmware version, time-since-fault. The PIS display fault on 4736-101 is also here — Roland sees the specific bildschirm (VLAN 3, IP 172.17.192.180, MAC visible, firmware version, port e2-2 on the A1 switch). He clicks the component row to reveal the secondary disclosure: MTTR (harness-computed: arrival timestamp of the Stadler alarm-raised IntentPacket → arrival timestamp of the alarm-cleared IntentPacket), recurring-failure history for this component, last 24h packet trace, vendor portal deep-link. Card carries the recommendation triplet; his move + his card-expansion are recorded against his IdP identity.

**10:14 — Live Board.** Roland pivots to Live Board. The fleet's moving map — 47 in-service trains across Austria, 12 depot trains across Wien, Floridsdorf, Simmering, Linz. He groups by subsystem: "CCTV across the fleet." 8 active CCTV faults, 3 of them on the same camera model. Kondukt has annotated the cluster: *"Recurring pattern detected — 3 of 8 active CCTV faults share hardware revision A7, mean-time-since-firmware-update is 47 days. Confidence: HIGH. Source: IntentPackets across 14 days; provenance shown."* Roland pivots to per-train view: he clicks train 4736-101 on the map. **Live Board redirects to that train's detail card in Kanban** — Live Board stayed glanceable, Kanban now shows him the per-subsystem depth (no VLAN IDs were shown on Live Board; they appear in the Kanban detail card he just opened). He looks at the CCTV row: Überw. Kamera A7, IP 172.18.192.138, firmware version, time-since-fault. Then back to Live Board for the next pattern. *No competitor sees this fleet-wide subsystem view with the depth-on-demand redirect.*

**11:40 — Kondukt: Vendor meeting prep skill.** Roland needs to prep for Friday's vendor meeting with Stadler — the bus-controller memory pattern is going to be raised. He invokes the *Vendor meeting prep* skill in Kondukt: *"Build a docx report on the bus-controller memory utilisation pattern across the DOSTO Neu fleet for the last 90 days. Cross-reference against Stadler's published firmware change history for the bus controller."* Kondukt invokes Rail MCP tools: pulls 90 days of IntentPackets, runs pgvector semantic similarity against the fault corpus, queries the recurring-failure detection skill. It uses allowlisted internet egress to fetch Stadler's published firmware change log from the approved vendor portal — every fetch logged. It assembles a docx in the canvas: cover page, pattern summary, fleet table (units affected, dates, IntentPacket IDs), Stadler firmware timeline overlay, plain-language conclusion with provenance citations. Roland reads the draft, asks Kondukt (free-form chat) to soften one phrase ("don't say 'Stadler's fault' — say 'pattern correlated with firmware version 2.3.1 release date'"). Kondukt revises. Roland clicks Download. The docx lands on his desktop. Both the skill invocation and the free-form chat turn are validated and logged as triplets; the downloaded artifact is a working artifact (not in the audit trail, per the audit scope lock).

**The harness records.** Every recommendation Kondukt produced today — the Daily Brief skill output, the Kanban move proposals, the Live Board annotations, the Vendor meeting prep skill output, the free-form chat revisions — is in the Recommendation Provenance Replay archive as a triplet. Roland's actions are in the shadow audit trail (recommendation → operator action → outcome). The two reconcile 1:1. Six months later, an auditor with the Replay CLI can pull any of those recommendations and verify, deterministically, that the harness checked the LLM's output against the constitution before letting it speak.

**The new reality.** Roland didn't switch tabs to Power BI, Nomad Monitoring, Stadler diagnostic, ServiceNow, or his Excel reconciliation. The harness was the only landside surface he opened. Across the day he made disposition decisions, triaged accumulating faults, surfaced a cross-subsystem pattern, produced a vendor-facing artifact — all in one context, all with provenance, all with his judgment on the action side and Kondukt's recommendation on the advice side.

*Capabilities revealed: Three-view harness (Live Board glanceable + Kanban depth-on-demand via train detail card + Kondukt for skills and free-form chat), Kondukt skill registry (Daily Brief, Vendor meeting prep, Recurring-failure pattern, Fleet health), Live Board → Kanban click-through redirect, per-train detail card with full per-component diagnostic depth (VLAN, IP, MAC, firmware, MTTR, recurring-failure history) in Kanban not Live Board, allowlisted internet egress, xlsx/docx artifact production, complete recommendation→action→outcome shadow audit chain reconciled 1:1 with the Provenance Replay archive.*

---

### Journey 3 — Dispatcher: "The Relay" *(Post-MVP — Dispatcher persona deferred from MVP scope)*

**Meet Ana.** Ana is a Fahrdienstleiter at Salzburg control room. She monitors signal and switch systems across her section. At 14:32 an IntentPacket arrives: Train 4712, 14-minute delay, signal fault at Salzburg Hbf.

**What Ana sees.** In the Fleet Manager AI Interface, alongside the triage alert, a relay card appears: "Train 4712 is delayed 14 minutes due to signal fault at Salzburg Hbf. Expected recovery: on departure." Plain language. Passenger-safe. No fault codes.

**The action.** Ana taps "Forward to Conductor." The relay card is pushed to the conductor's PWA notification (via brain). Ana taps "Copy" and pastes it into the PA system announcement. She did not rewrite anything. She did not call the conductor. Thirty seconds from alert to passenger announcement.

**The new reality.** The relay card was generated by Kondukt from the IntentPacket `incident_summary`. Ana's job is coordination, not translation. The platform handled the translation.

*Capabilities revealed: Relay card component, LLM-generated passenger-safe language, copy-to-clipboard, forward-to-conductor push notification, Dispatcher coordination loop.*

---

### Journey 4 — ECM Lead: "Reviews the Platform" (Commercial Confidence Journey)

**Meet Dr. Renate Fischer.** Dr. Fischer is Head of Vehicle Maintenance at ÖBB Technische Services — the ECM-accountable role. She was not in the pilot planning meeting. Martin Lerch brings her in at approximately pilot manday 15. Her question is not "does it work?" — it is "is this harness's audit-trail capability sound enough that I would be willing, after pilot end, to make it the authoritative ECM record?"

**What she reviews — Part A: the shadow audit trail.** She opens the ECM shadow audit-trail export for Unit 4722 alongside the original paper sign-off scan. She compares: IntentPacket generated at 05:50 (14 minutes pre-arrival), harness shadow record opened by Klaus at 05:53, fault resolution chain identical to the paper — detection → conductor ACK → technician assignment → part fitted → test result, shadow sign-off at 06:08 (Klaus's IdP identity, not a local username), vehicle state snapshot at sign-off, directive reference EU 2019/779, HMAC + hash-chain tamper evidence on each row. The shadow record and the paper match field-for-field.

**What she reviews — Part B: the Recommendation Provenance Replay.** Then Abbas hands her a laptop with the Replay CLI. He asks her to pick any recommendation from the harness archive at random. She picks one: a Daily Brief finding from manday 12 that said "Bus-controller memory pattern on three DOSTO Neu units suggests memory leak." She runs the Replay CLI against the archive on the laptop. The CLI fetches the triplet (Envelope + Provenance Bundle + Validation Receipt), pulls the original IntentPackets and MCP queries the recommendation cited, re-runs the validator against the LLM's raw output (stored in the Provenance Bundle, no LLM session needed), and confirms: every claim cites a real IntentPacket; the advisory-only language linter passed; the schema matched. Verdict: PASS. *"Wenn ich das in einem halben Jahr nochmal mache, läuft es genauso?"* she asks. Yes, Abbas confirms — that's the point. The validator has no LLM in its loop; it's pure functions over content-addressed blobs. The archive is immutable. *"Gut."*

**Her first question.** "Can I export the audit trail in a format our documentation system accepts?" PoC answer: structured JSON export, copy-paste ready. BOOM integration deferred — noted, not a blocker.

**Her second question.** "What happens if Klaus signs off incorrectly — can we correct it without deleting the record?" She is shown the `supersedes_id` pattern — a new row points to the original, original is never touched. She also asks the new question that advisory-only invites: "And what happens if the paper says one thing and your shadow says another?" Shown the reconciliation queue and the divergence flag — divergences are surfaced within 24h, never silently ignored, and persistent divergence downgrades the Phase 2 readiness assessment.

**Her third question (new, from the harness review).** "And what happens if Kondukt recommends something that turns out to be wrong?" She is shown the Validation Receipt's `disposition` field — every recommendation is either *emitted* (passed validation) or *suppressed* (failed validation, logged with reason, never reached the operator). She sees the suppression log for the week — 7 recommendations suppressed across the fleet, each with a reason (citation-trace failure x3, advisory-language breach x2, schema validation x2). None of them reached an operator. *"The harness owns the correctness. The LLM is supplied by the customer, but the LLM cannot escape what the constitution says it must be."*

**The outcome.** She does not object. She requests to be a reference customer for the audit-trail capability + the Provenance Replay archive — both jointly — and she conditions her support for Phase 2 on the shadow trail's match rate against paper holding at 100% across the full pilot AND the Replay archive integrity holding at ≥99% replay-verify clean across the full pilot. Roland gets his Phase 2 readiness signal. Late-stage objection risk eliminated.

*Capabilities revealed: ECM shadow audit-trail export, full resolution chain view, supersedes_id correction flow, IdP-bound identity display, HMAC + hash-chain tamper evidence, paper-trail reconciliation surface, divergence flagging, structured JSON export, **Recommendation Provenance Replay CLI demonstrated live on the auditor's laptop**, Validation Receipt suppression-log inspection.*

---

### Journey 5 — The ROI Conversation (Commercial — Pilot Manday ~15)

**Meet Roland and Martin.** It is approximately pilot manday 15. Roland Ruisz (operations + telematik project leadership) and Martin Lerch (ÖBB Technische Services) sit down with the pilot evidence view. They are not assessing features — they are assessing whether to sign a Phase 2 readiness recommendation.

**What they see (adoption-dominated, locked 2026-05-25 post-operator-correction — no euro figures pre-committed; the pilot is producing them, not proving them).** The pilot scorecard composite:

- **Per-hat recommendation acceptance rate** — Disponent-hat decisions on Daily Brief + Kanban: 78% (target ≥70% ✓). Telematik-hat reads on Live Board annotations: 71% (target ≥70% ✓). Teil-PL-hat artifacts from Chat+Canvas: 8 artifacts downloaded and used in real vendor / programme meetings, with qualitative endorsement from Roland.
- **Recommendation Provenance Replay archive integrity** — 99.7% of recommendations replay-verify cleanly (target ≥99% ✓). 0 recommendations without triplets (target 0 ✓). 0 advisory-only language breaches (target 0 ✓). Auditor demo at manday 20–25 already locked.
- **ECM pre-arrival review rate** — 83% (target >80% ✓).
- **Shadow audit-trail match rate** — 100% across all N paper sign-offs in pilot to date (target 100% ✓).
- **Pilot-kill triggers in last 30 days** — 0 unresolved (target 0 ✓).
- **Phase 2 readiness composite — AT THRESHOLD** on the five trust criteria.

**Below: qualitative outcomes captured for the post-pilot ROI narrative (not pilot pass criteria — the pilot produces this evidence, it does not prove it).** Recurring-failure detections from Live Board that surfaced patterns Roland's existing process did not catch: 4 confirmed. Chat+Canvas-authored vendor-facing reports Roland used in real Stadler / Siemens escalations: 3. Prevented delay-minutes counterfactual (best-effort approximation, not audit-grade): 6 faults detected pre-disruption, industry-standard mean delay 19 minutes per fault class — order-of-magnitude estimate ~115 delay-minutes prevented across pilot to date. ECM audit hours at Renate's last quarterly review compared to her pre-pilot baseline: ~30% reduction in audit prep time. *These are the numbers that go into the Phase 2 commercial proposal, but they are not what the pilot is being measured on.*

**Martin's question.** "What does this cost us to run per train per month?" The harness surfaces infrastructure cost estimate (€ per consist per month, BYOK token usage projected from pilot data, hosting + storage). Roland cross-references against the qualitative outcomes — *"Wir haben drei Vendor-Argumente in diesem Quartal gewonnen, die wir vorher verloren hätten, weil wir die Evidence nicht zusammenbringen konnten. Das allein bezahlt es schon."* The harness pays for itself within the pilot's qualitative evidence; the precise euro multiplier is a post-pilot research output.

**The renewal-conversation script (sign-offs as gate, trust criteria as texture).** At pilot end: 3/3 sign-offs (Roland + Renate + outside-auditor tabletop verdict on the Provenance Replay archive) = extend into Phase 2; anything less = don't. The five composite trust criteria are the *texture* — how we describe what we learned in the renewal conversation. The sign-offs are the *gate* — binary.

**The outcome.** Roland does not need to argue the case to his CFO from a euro figure alone. The harness produced the *evidence dataset* (adoption + Replay archive + sign-offs + recurring-failure detections + vendor-meeting reports + ECM audit time savings) that the Phase 2 commercial proposal builds on. Phase 2 is a contract renewal with a pre-mapped path, not a fresh procurement fight. The euro-denominated ROI story is post-pilot research; the pilot's job was to *produce* that evidence, not to *commit* to it pre-pilot.

*Capabilities revealed: Pilot scorecard composite (per-hat adoption + Provenance Replay integrity + ECM pre-arrival + shadow-match + pilot-kill-trigger-count + Phase 2 readiness threshold), qualitative outcome capture surface, infrastructure cost projection, renewal-conversation script auto-generated against the five-criteria gate, export for executive presentation including Phase 2 readiness verdict.*

---

### Journey 6 — *(Retired 2026-05-25 — Vendor Warranty Triage workflow removed from MVP)*

The vendor warranty *recovery workflow* is post-MVP. Vendor warranty *detection* — surfacing patterns that look warranty-recoverable across the fleet — is supported in MVP through Live Board recurring-failure detection + Chat+Canvas vendor-facing report generation (see Journey 2). The structured-warranty-export-to-vendor workflow is a Phase 1.5 candidate if the detection capability generates real evidence during the pilot.

---

### Journey 7 — *(Retired 2026-05-25 — OTA Rollout Management removed from MVP)*

OTA rollout management is post-MVP (vendor OTA tooling remains canonical). Configuration / firmware version drift visibility was moved to the ECM module (quarterly cadence — not a daily harness view). The Drift Visibility module is a separate workstream from this PRD; the daily harness does not include it.

---

### Journey 8 — Harness Degraded: Fleet Manager Under Stale Data + Unreachable LLM (Edge Case)

The harness has three independent degradation surfaces. Each fails differently. The harness must continue to serve the operator under each.

**Surface 1: Stale telemetry (brain inference gap).** 09:14. The onboard brain inference stalls on a specific train — IntentPacket emission has paused for 80 seconds. The landside MQTT consumer keeps the last retained packet but no new ones arrive. On the Live Board, train 4712's card shows a staleness indicator — last updated timestamp, amber border. Kanban annotations for this train carry a `data_freshness_ts` of 94 seconds ago. Roland sees it. He does not act on stale data. Brain inference resumes at 09:17. Fresh IntentPacket arrives. The card updates. No data lost; no decision silently affected.

**Surface 2: Rail MCP tool timeout.** Roland asks Kondukt to run the recurring-failure pattern skill across the bus controllers. Kondukt invokes the `get_recurring_failure_pattern` MCP tool. The tool times out at 8 seconds. **The Validation Receipt for Kondukt's response now has a citation-trace failure** — Kondukt cannot cite IntentPackets it does not have. The validator rejects the emission. *The recommendation is suppressed*, logged with reason: `tool_timeout: get_recurring_failure_pattern`. Roland sees: *"Kondukt could not complete this skill — the recurring-failure analysis tool timed out. Try again, or simplify the query."* He simplifies. The next attempt succeeds. Phase 2 trust evaluation tracks whether decisions made under tool-timeout conditions diverge in outcome from decisions made with full data.

**Surface 3: BYOK LLM unreachable (the harness-LLM contract surface).** At 14:32, Anthropic's API returns sustained 429 responses to Roland's configured key — token budget exhausted for the day (BYOK = his money; his org's quota; the harness has no control over this). The harness detects three consecutive 429s, marks the LLM as degraded. **Live Board and Kanban remain fully functional** — both are telemetry-driven, no LLM in the data path. Live Board still shows the moving map + per-subsystem health; Kanban still displays existing cards (the LLM proposed their placement earlier, the data is durable). **Kondukt shows degraded state:**

- Kondukt session banner: *"Unavailable — your LLM provider returned 429 (rate limit). Your token budget may be exhausted, or your API key may need attention. Kondukt skills and chat are paused until the harness can reach your LLM. Previous artifacts and chat history remain readable."*
- Skill invocations (Daily Brief, Recurring-failure report, Vendor meeting prep, Fleet health summary) all suppressed with provider-status reason.
- Free-form chat input disabled; existing artifacts and conversations remain readable; download remains functional.
- Kanban: new Kondukt lane-placement proposals paused; existing cards remain movable manually by Roland (the operator can always move cards; the LLM was advising on placement, not gating it).
- Live Board: existing Kondukt annotations remain visible (read from the archive); no new annotations until provider recovers.

Roland falls back to current practice — direct telemetry view via Live Board, his own judgment on the Kanban, no LLM-authored research. The harness logs the degradation event for the pilot scorecard. When the BYOK provider recovers (next billing cycle, key refreshed, quota restored), the harness reconnects automatically; Kondukt skill invocations resume on next call.

**Constitution-as-contract under degradation.** Notice what does *not* happen: the harness does not attempt to emit recommendations using a fallback LLM or stale cached LLM outputs. The constitution-as-contract is binary — either the validator can verify the LLM's output against the Provenance Bundle in real time, or the recommendation is suppressed. Silence is safe. The pilot scorecard counts suppressions; it does not count fabricated emissions. This is the architectural reason a one-person Einzelunternehmen can supply this on a NIS2-scope system.

*Capabilities revealed: Three-surface degradation (telemetry staleness / MCP tool timeout / BYOK LLM unreachable), validator-driven suppression on tool-timeout citation failure, BYOK-unreachable banner with telemetry views still functional, automatic recovery on provider reconnect, suppression event logging for pilot scorecard, **the harness-LLM contract binary behaviour: emit-or-suppress, never fabricate**.*

---

### Journey 9 — ECM 4 Technician: "The Fault ACK" (Primary — Operational Path)

**Meet Marco.** Marco is an ECM 4 Electrical Technician at Wien Westbahnhof depot. He covers the overnight maintenance window. He is not accountable under 2019/779 — that is Klaus's responsibility — but nothing moves on Klaus's screen until Marco logs his work.

**Before the harness.** Marco read fault codes off the diagnostic terminal, wrote the resolution on a paper sheet, handed it to the shift supervisor, who transcribed it to the system. If he used the wrong fault code, the ECM sign-off referred to a resolution that didn't match the physical repair.

**Now.** At 03:15, Marco opens the Depot Briefing PWA on his tablet. He taps "Electrical" — his discipline. He sees one fault in his queue: CRITICAL, Car 4 Door 2L motor fault. He opens it. The full IntentPacket chain is visible: conductor ACK at 01:20, fault isolated to door motor assembly, part requisitioned.

**The ACK.** Marco fits the replacement part at 03:22. He taps "Acknowledge — Resolved" in the PWA. **He also signs the existing paper fault log per current depot process.** The harness's shadow ACK record writes: his IdP identity, UTC timestamp, resolution action ("Replacement part fitted — Door Motor Assembly Car 4 Door 2L"), and the fault ID it references. No free text. The fault moves from OPEN to RESOLVED in Klaus's pre-arrival shadow brief.

**What he cannot do.** Marco cannot sign off the vehicle — that requires ECM 1 authority (and remains a paper process in MVP). He cannot see faults outside Electrical — Mechanical and IT faults are not in his feed. He cannot modify or delete his shadow ACK after it is written.

**The new reality.** Klaus sees Marco's shadow ACK in the harness's resolution chain at 05:50, and verifies it against the paper fault log on arrival. Over 6 months, the consistent match between Marco's shadow ACKs and the paper fault log builds the dataset that proves Phase 2 can drop the paper step.

*Capabilities revealed: Role-filtered fault feed (Electrical discipline only), structured shadow ACK with IdP-bound identity, append-only ACK record, fault state transitions (OPEN → RESOLVED), shadow ACK with paper-trail reconciliation, ACK match rate as Phase 2 trust input.*

---

### Journey Requirements Summary

| Capability area | Revealed by |
|---|---|
| Pre-arrival depot brief generation | Journey 1 |
| Role-filtered fault feed (ECM/Electrical/Mechanical/IT) | Journey 1 |
| ECM 1 blocking authorisation + simple-click sign-off (MVP) | Journey 1 |
| Append-only sign-off with full resolution chain | Journeys 1, 4 |
| **Live Board view — glanceable moving map + per-subsystem health (operator-friendly names, no VLAN IDs); group-pivot subsystem ↔ train; click-through redirect to Kanban detail card** | Journey 2 |
| **Kanban view — lane-sort by neglect-risk, not queue order; train detail card is the depth surface with progressive disclosure (primary: VLAN/IP/MAC/firmware/time-since-fault; secondary on click: MTTR/recurring-failure-history/24h-packet-trace/vendor-portal-deep-link)** | Journey 2 |
| **Kondukt view — chat + canvas + skill registry + free-form research + allowlisted internet egress + xlsx/docx export** | Journey 2 |
| **Kondukt skill registry — Daily Brief, Recurring-failure pattern report, Vendor meeting prep doc, Fleet health summary (versioned in-product; skill set evolves through pilot via ADR)** | Journey 2 |
| **MTTR computed by harness from landside-observed Stadler alarm IntentPacket arrival timestamps** (`t_close − t_open`) | Journey 2 |
| **Recurring-failure detection annotations from Kondukt (with provenance)** | Journey 2 |
| **Per-subsystem coverage**: CCTV, PIS, intercoms, AFZ, audio, Wi-Fi, switches, TCMS, Stadler diagnostic plane | Journey 2 |
| Rail MCP tool calls (telemetry, depot state, MTTR, recurring-failure, semantic search, neglect-risk inputs, skill executors, xlsx/docx artifact builders, allowlisted internet fetch) | Journey 2 |
| Shadow audit-trail with paper reconciliation | Journeys 1, 4, 9 |
| **Recommendation Provenance Replay archive — triplet at emission; auditor-runnable CLI; reconciles 1:1 with shadow audit trail** | Journey 4 (auditor demo); Journey 2 (emission) |
| **Constitution-as-contract enforcement — per-skill schemas + provenance validators + advisory-language linter; failed validations suppressed (no MVP regeneration)** | Journeys 2, 4, 8 |
| Phase 2 trust-criteria composite scoring | Journey 5 |
| Pilot scorecard composite (adoption-dominated, per-hat) | Journey 5 |
| Renewal-conversation script (3/3 sign-offs as gate, five trust criteria as texture) | Journey 5 |
| Relay card + LLM passenger-safe language | Journey 3 *(post-MVP — Dispatcher persona deferred)* |
| Forward-to-conductor push notification (to brain) | Journey 3 *(post-MVP — Dispatcher persona deferred)* |
| ECM audit trail export + supersedes_id correction | Journey 4 |
| **Replay CLI demonstrated live on auditor's laptop at manday ~15 + manday 20–25 tabletop** | Journey 4 |
| **Validation Receipt suppression-log inspection surface** | Journey 4 |
| **Three-surface harness degradation**: stale telemetry / MCP tool timeout / BYOK LLM unreachable | Journey 8 |
| **Validator-driven suppression on tool-timeout citation-trace failure** | Journey 8 |
| **BYOK-unreachable banner — telemetry views remain functional, LLM-driven views show degraded state, no fallback emission** | Journey 8 |
| Role-filtered fault ACK (Electrical discipline) | Journey 9 |
| Structured ACK with IdP-bound identity, append-only | Journey 9 |
| Fault state transition (OPEN → RESOLVED) visible in ECM brief | Journey 9 |
| Warranty *recovery workflow* | *(Retired from MVP — Journey 6 removed. Warranty *detection* in Live Board + Chat+Canvas in MVP.)* |
| OTA rollout management | *(Retired from MVP — Journey 7 removed. Drift Visibility moved to ECM module, not daily harness.)* |
| Lastenheft procurement clause export | *(Descoped — moved to Vision)* |

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

**Recommendation Outbox Pattern**
- All Kondukt emissions (skill outputs + free-form chat turns) and their reconciliation state persist in `recommendation_outbox` (PostgreSQL table) with `status` enum, `idempotency_key UNIQUE`. In-memory queues are not used. Worker polls + `SELECT ... FOR UPDATE SKIP LOCKED`. Survives harness restart without loss — critical for the Provenance Replay archive integrity guarantee.

**Recommendation Provenance Replay Archive**
- Every Kondukt emission produces a triplet at emission time (per FR29): Envelope (in `recommendation` table) + Provenance Bundle (in content-addressed blob store, SHA-256-keyed) + Validation Receipt (linked record). Append-only enforcement at the database level (row-level trigger raises exception on UPDATE/DELETE). No regeneration loop in MVP — failed validations suppress.
- The Replay CLI is an auditor-runnable single-binary tool that takes a `recommendation_id`, fetches the triplet, re-runs the validator against the stored Provenance Bundle, and asserts the Validation Receipt matches. Runs on a laptop with no harness infrastructure.

**Security**
- BYOK API keys stored in encrypted vault (PoC: env vars + at-rest encryption; fleet: Docker secrets + KMS). Never in source control, never in logs, never transmitted to harness vendor logs. Token usage transparent to operator on every Kondukt session.
- All external data (IntentPackets, MCP tool inputs, allowlisted-egress fetches) treated as untrusted — schema validation before any LLM-decision path
- **Constitution-as-contract enforcement layer** (per FR23–FR27) is between Kondukt response and any UI render. Validators are deterministic, no LLM in the validation loop. Failed validations suppress; suppression is logged.
- **Recommendation review + operator-action logging** — the harness does not dispatch in MVP. Operator action confirmations (Kanban moves, recommendation acceptances) are recorded against the operator's IdP-bound session as shadow audit events. Re-verified IdP session is not required for confirmation gestures because no operational system is being mutated by the harness.

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

**ECM Shadow Sign-Off (MVP — simple click)**

In MVP, ECM 1 shadow sign-off is a **single click** on "Record sign-off" in the Depot Briefing PWA. The click writes the `ecm_signoff_events` shadow row with the operator's IdP identity captured at moment of click. Idempotency key generated client-side at click; safe retries (409 Conflict treated as success). After write, record is immutable (append-only DB trigger per FR51) — no undo. No client-side deliberate-confirmation gesture in MVP.

**Why simple click and not deliberate-confirmation gesture in MVP:** the harness's shadow sign-off in MVP is *additional* to the paper sign-off (which remains authoritative under EU 2019/779). The deliberate-confirmation friction is appropriate where the click *is* the regulatory authority; in MVP, the paper carries that weight. The click captures intent + identity + immutable record — sufficient for the shadow trail.

**Phase 2 candidate — Hold-to-Record deliberate-confirmation gesture (descoped from MVP 2026-05-25)**

When the platform's sign-off becomes authoritative under EU 2019/779 in Phase 2, a deliberate-confirmation gesture is reconsidered. Full interaction arc preserved in `design-artifacts/ecm-signoff-interaction-spec.md`. The spec described here is **Phase 2 candidate material, not MVP**:

- Two-phase gesture: hold-to-initiate (3s) → 5-second visible countdown → single atomic write at expiry
- Six screen states (Idle / Hold Active / Countdown / Writing / Confirmed / Error)
- Hold threshold: 3 seconds (not 2 — accidental-trigger range in gloves at 2s)
- Client-side timer only — no server-side pending state
- Idempotency key generated at countdown initiation; safe retries
- Countdown pauses on focus loss (radio call, incoming notification)
- Protokoll-ID mandatory in confirmed state — chain of custody requires the operator to read it aloud
- Green only appears in confirmed state — not in button, not in progress indicators
- Glove-gesture acceptance criteria (≥90% first-try success, median ≤9s, p95 ≤14s, false-abort <5%, subjective trust ≥4/5) — applies to **Phase 2 gate**, not MVP

**Glove walkthrough timing — re-scoped to Phase 2.** The pre-pilot glove-gesture walkthrough (ECM 4 Technician + ÖBB winter glove SKU + ≥20 sessions across two depot lighting conditions) was originally a pre-MVP-pilot gate. Descoped from MVP 2026-05-25 along with the gesture itself. Re-introduced as a **pre-Phase-2 gate** if the Phase 2 readiness verdict includes promoting the platform's sign-off to authoritative.

**pgvector Fallback Transparency**
- Fleet Manager always knows which search mode is active — semantic or keyword
- UI label "Keyword search active" shown whenever fallback is in effect
- No silent degradation permitted

**Recommendation Interaction (MVP — no deliberate-confirmation gesture)**
- **No "Mark executed" gesture in MVP** (descoped 2026-05-25). Roland's interactions with a recommendation card — accept / modify / reject / lightweight mark-seen click — produce the action record on the `operator_action` row linked by `recommendation_id`. There is no separate dispatch-style confirmation surface.
- The recommendation card surfaces the action context for the operator: which system to touch (HAFAS, vendor portal, depot work-order system, etc.) and the operational artifact (e.g. exact HAFAS payload) where applicable. The operator acts via existing systems; the harness does not dispatch.
- Operator interactions log: IdP identity at moment of interaction, timestamp, interaction type, optional modification text + `modification_distance` if modify action.
- Platform polls downstream systems where possible to verify outcome (FR33). Outcome states: `verified` / `self-reported-unverified` / `divergent`. Persistent `self-reported-unverified` outcomes surface as distinct UI state — not buried in error logs.

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

Three landside roles. The Fleet Manager role spans one person's full work (Roland Ruisz: Flottenmanagement + Team Telematik + Teil-Projektleitung Telematik for DOSTO Neu + Enzo — one role, two programme hats). ECM Manager + ECM 4 Technician are recommendation-consumer roles on the Depot Briefing PWA.

| Role | Surface | Permissions |
|---|---|---|
| **Fleet Manager (Roland Ruisz)** | Harness — all three MVP views (Live Board, Kanban, Kondukt) | Full read across all harness views; recommendation review + acceptance / modification / rejection logging; Kanban lane moves + train detail card expansion; Kondukt skill invocation + free-form chat + artifact downloads; BYOK key management for own session. **Does not dispatch — acts via existing systems (HAFAS direct, vendor portals, paper sign-off, radio).** |
| **ECM Manager (ECM 1)** | Depot Briefing PWA + ECM Audit Export | Pre-arrival brief review; **paper sign-off (existing process); shadow sign-off via simple click in MVP** (deliberate-confirmation gesture descoped 2026-05-25; hold-to-record gesture preserved in design-system reference as Phase 2 candidate); audit trail read; blocking state control; ECM audit export download. **In MVP the paper sign-off remains authoritative.** |
| **ECM 4 Technician** | Depot Briefing PWA (role-filtered: Electrical / Mechanical / IT) | Role-filtered fault feed; **shadow ACK of faults in own discipline (paper ACK remains existing process)**; no cross-discipline access; no ECM 1 sign-off authority |
| ECM Lead (Dr. Renate Fischer) | ECM Audit Export surface (read-only) + Replay CLI access | Full audit-trail export read; reconciliation queue view; Recommendation Provenance Replay archive access + CLI verification. Sign-off as one of the five composite Phase 2 trust criteria. |

**Retired role context:** *"Teil-Projektleitung Telematik" as a separate permission tier* — collapsed into the Fleet Manager role (same person, same surface; Roland's commissioning hat is one of his contexts, not a distinct permission scope).

All identity via IdP-bound session — no local user table for safety/compliance actions. **MVP uses Keycloak with manually created users.** ÖBB SSO IdP integration is a post-MVP change request, costed separately.

**IdP per-operator**
MVP uses Keycloak with manually created users — no ÖBB SSO integration. ÖBB SSO IdP integration is a post-MVP change request, costed separately. Fleet rollout: each operator brings their own IdP. Per-operator SSO customisation will be requested — budget 2–3 week integration tax per operator or build a standardised SAML bridge.

---

### Integration List

| System | Integration type | PoC status | Fleet gate |
|---|---|---|---|
| Identity provider (IdP) | Identity for ECM shadow sign-off + Kanban moves + Chat+Canvas session identity | **MVP: Keycloak with manually created users.** ÖBB SSO IdP integration is post-MVP — scoped as a change request. | Per-operator SAML bridge at fleet rollout |
| MQTT broker | IntentPacket consumption from oebb-brain (including TCMS via brain VLAN 7 SNMP forward) | Onboard subscriber only | **Blocking pre-fleet:** broker identity, failover, and multi-train topic isolation specified by brain team |
| Rail MCP Server | HTTP MCP endpoint — tools over the PostgreSQL+pgvector data layer (telemetry per subsystem, depot state, MTTR computed by harness from Stadler alarm IntentPacket timestamps, recurring-failure detection, semantic search, neglect-risk scoring inputs, xlsx/docx artifact builders, allowlisted internet fetch) | Spec frozen — updated to harness-model scope (toolset narrowed from the prior 9-tool list to the four-view operational set) | Required for the harness |
| Customer LLM (BYOK) | Operator brings their own LLM API key (Anthropic, OpenAI, or other). Harness supplies constitution + skills; LLM supplies inference. | Spike: BYOK key handling + 429 backoff + provider-agnostic constitution. Test that swapping providers does not break the contract. | BYOK cost simulator required before fleet sales |
| **Allowlisted egress proxy** | Curated approved-domains list for LLM internet research. Vendor portals (Stadler, Siemens, Knorr-Bremse), standards bodies (CENELEC, ERA), EU regulatory texts. Every fetch logged. | Allowlist drafted before pilot; sign-off by ÖBB IT Security (PV/BCC) + Nomad legal | Per-operator allowlist customisation at fleet rollout |
| HAFAS | Timetable read for pre-arrival brief generation; rotation write-back deferred to Phase 2 | MVP reads only — operator dispatches via HAFAS directly | 48h horizon |
| Stadler diagnostic system (RDC) | Read MTTR data + component-alert-create-to-cleared timestamps. Read via the brain (which polls VLAN 200/202 + VLAN 7) and forwards landside; harness does not query Stadler directly. | Brain-mediated; spec frozen with brain team | Brain workspace owned |
| SAP PM/EAM | Asset metadata + work order write-back | **Post-MVP** — API contract required with ÖBB TS before integration stories | Read for asset metadata; write for work order routing |
| ServiceNow ITSM | Ticket creation from harness recommendations | Post-MVP | API contract required |
| BOOM | Depot work order integration | Descoped — structured JSON export only | API contract required from ÖBB IT before integration stories |
| pgvector | Semantic search over IntentPacket fault summaries + harness recommendation history + vendor doc embeddings (for Chat+Canvas RAG-style retrieval) | Spike + P99 confirmation required | Fallback: keyword ranking |
| Vendor diagnostic portals (Stadler, Siemens, Knorr-Bremse) | Allowlisted internet fetch (read-only documentation/changelogs/published advisories). NOT vendor diagnostic API integration. | Allowlist-based MVP; vendor API integration is post-MVP and requires vendor credential model | Vendor credential model required for API integration |

---

## 7. Innovation Focus

### What Is Novel Here (Landside)

The product is **a harness for a customer-provided LLM** over a regulated-rail data layer. The harness owns correctness; the customer brings the model. Every recommendation the LLM produces is validated by per-skill schemas + provenance validators + an advisory-language linter before emission — if validation fails, the recommendation is suppressed (logged with reason), not regenerated. Every emitted recommendation, every operator action, and every outcome are captured in two parallel artifacts: the **Recommendation Provenance Replay archive** (an auditor-runnable verification surface) and the **ECM shadow audit trail** (the regulatory record). The two reconcile 1:1.

The novel claims, in MVP-applicable form:

---

**Claim 1 — A constitution-bound LLM harness is the product, not the LLM**

ÖBB Flottenmanagement plugs in their LLM API key (Anthropic, OpenAI, or other — BYOK). The harness supplies constitution + skills + Rail MCP server + tools + allowlisted internet egress + the PostgreSQL+pgvector data layer + the audit trail + the provenance archive. The same constitution gives any customer-provided LLM the Kondukt identity. The customer is not buying inference; they are buying the regulated-rail scaffolding that turns their LLM into a fleet co-pilot. No inference-resale economics; no model-churn risk; no vendor lock-in to a specific LLM. The customer's data stays close to the LLM provider relationship they already have.

*Validation: Roland Ruisz signs off at manday ~15 + pilot end. Pilot scorecard captures per-hat adoption (Disponent / Telematik / Teil-PL) for Daily Brief, Kanban, Live Board, Chat+Canvas. Auditor verifies that swapping the LLM provider (Anthropic → OpenAI or vice versa) does not break the constitution-as-contract guarantees — done as part of the manday 20–25 tabletop.*

---

**Claim 2 — The constitution is enforced as contract, not stated as prompt**

The constitution is not just a system prompt. Per-skill output schemas, citation-trace validators, and the advisory-language linter are deterministic post-processing on every recommendation the LLM emits. If a recommendation cites an IntentPacket that doesn't exist, the validator rejects it. If a recommendation uses imperative-action language ("dispatch consist X"), the linter rejects it. If the schema is malformed, the validator rejects it. Rejected recommendations are suppressed (logged with reason); no regeneration loop in MVP. **Silence is safe.** This is the safety posture — *the harness owns correctness, not the LLM* — that makes a one-person Einzelunternehmen tolerable as a NIS2-scope supplier on a safety-adjacent system. § 1313a ABGB / Erfüllungsgehilfe does not let us hide behind "the LLM said it"; we have to constrain the LLM. The harness does.

*Validation: Recommendation Provenance Replay archive shows ≥99% replay-verify clean; zero recommendations exist without complete triplets; zero advisory-only language linter breaches across the full pilot. Demonstrated live to the outside auditor at manday 20–25.*

---

**Claim 3 — Multi-subsystem visibility no competitor produces**

Nomad Monitoring sees only Internet-on-Board. Stadler diagnostic sees only Stadler's plane. Knorr-Bremse sees only brakes. Power BI is a dashboard layer over whatever silo you point it at. The harness sees every onboard subsystem in one context — CCTV, PIS (interior displays + side + front destination displays), AFZ passenger counter, intercoms (incl. PRM variants), audio amplifiers + ADU, Wi-Fi access points, FIS switches, Frontkupplung, energy meter, Service FIS/CCTV diagnostic, Zugfunk, TCMS (received from the brain via VLAN 7 SNMP forward), Stadler diagnostic plane (RDC + OBS), Firewall RCU. Live Board surfaces these by operator-friendly name (no VLAN IDs); the depth — VLAN, IP, MAC, firmware version, MTTR, recurring-failure history — lives in the Kanban train detail card. Per-subsystem and per-train pivots on Live Board with click-through redirect to Kanban for diagnostic depth. MTTR sourced from the Stadler diagnostic system. Recurring-failure detection annotations from Kondukt with provenance. Cross-subsystem correlation (intercom drops + CCTV drops + Wi-Fi AP drops on the same train within the same window = network-path issue, not three independent subsystem failures) is something cloud-only competitors structurally cannot do.

*Validation: Live Board pivot-axis usage during pilot — both subsystem-pivot and train-pivot; Live-Board-to-Kanban click-through rate. At least 4 cross-subsystem correlations surfaced by Kondukt during the pilot, with provenance, that Roland's existing process did not catch.*

---

**Claim 4 — Regulatory compliance as a trust-building primary feature**

The ECM shadow audit trail is not a log — in MVP it is the proof artifact that the harness's audit-trail capability *can be* the regulatory record under EU 2019/779. Every shadow sign-off is an append-only, hash-chained record with IdP-bound identity, vehicle state snapshot, directive reference, and HMAC tamper evidence. Corrections are new rows with `supersedes_id`. The pilot's reconciliation against paper trails — 100% match rate target — is the evidence Dr. Fischer needs to authorise Phase 2. The Recommendation Provenance Replay archive reconciles 1:1 with the shadow audit trail — each shadow audit entry touching a harness-influenced action maps to the recommendation triplet that produced the recommendation acted on.

*Validation: Shadow audit export matches paper sign-off field-for-field in 100% of pilot sign-offs — confirmed by Dr. Fischer at manday ~15 and continuously thereafter. Replay CLI demonstrated live on Dr. Fischer's laptop and at the manday 20–25 outside-auditor tabletop.*

---

**Claim 5 — Advisory-first earns the right to dispatch**

No competitor in this market offers a phased trust progression where Phase 1 is recommendation + provenance archive + audit-capability proof, and Phase 2 unlocks dispatch only after five composite trust criteria are met (acceptance rate + shadow match rate + Renate sign-off + outside-auditor tabletop verdict + zero unresolved pilot-kill triggers). This sequencing converts the buyer's risk-committee objection ("we cannot let an AI dispatch on day one") into a contractual phase the buyer's procurement office can sign with confidence. **Operational-sovereignty retention is the sword:** ÖBB executes; the harness recommends; the harness owns correctness; the customer owns the LLM relationship. No incumbent can match this in six months without rebuilding the constitution-as-contract enforcement layer + the provenance archive + the harness-LLM contract.

*Validation: Roland's Phase 2 readiness sign-off at manday ~15 on the basis of the five-criteria composite. Outside auditor's verdict at manday 20–25 on the Recommendation Provenance Replay archive integrity.*

---

### Innovation Risks

| Risk | Mitigation |
|---|---|
| **Constitution-as-contract enforcement layer** — per-skill schemas + provenance validators + advisory-language linter | Winston's spec (party mode 2026-05-25). Schemas + validators are inseparable engineering item; regeneration deferred from MVP. Validator unit-tested per skill before pilot. ≥99% replay-verify clean is a pilot pass criterion. |
| **Recommendation Provenance Replay archive** — content-addressed blob store + Postgres append-only table + auditor-runnable CLI | Boring stack (~2000 LOC). Tested against multiple LLM providers (Anthropic, OpenAI) to confirm validator is model-agnostic. CLI runs on auditor's laptop without harness infrastructure. |
| **BYOK key handling security** — encrypted at rest, per-user scope, never logged, never in inference path | Spike: BYOK 429 + credential isolation before harness LLM-emission stories. ÖBB legal review on key storage posture. |
| **LLM output sanitisation** — between Kondukt response and UI render | Pydantic schema validation + advisory-language linter; on validation failure, suppress (log + UI banner: "Recommendation engine returned a malformed response — manual decision required"). Prompt injection threat model documented before pilot. |
| **Allowlisted egress policy** — curated approved-domains list | List drafted before pilot; signed off by ÖBB IT Security (PV/BCC) and Nomad legal. Every fetch logged. Allowlist editable via versioned config + ADR. |
| **Phase 1 trust criteria not met at pilot end** | Five-criteria composite is weighted, not binary — partial trust unlocks a narrower Phase 2 scope. Phase 2 release is itself phased. |
| **MVP suppression rate too high** (validator rejects too many emissions; harness reads as broken) | Suppression rate is a pilot metric. >5% sustained = investigate validator calibration. If validator is over-strict, tune the schemas. If LLM is under-performant, that is a customer model-choice signal — visible to operator before they switched providers. |

---

## 8. Project Scoping & Feature Prioritisation

### Strategy & Philosophy

**Approach:** Compliance-first, advisory-only landside harness PoC — single release covering one Fleet Manager harness surface with three views (Live Board + Kanban + Kondukt) + Depot Briefing PWA (recommendation-consumer) + ECM Audit Export (Dr. Fischer review surface). All in advisory mode; constitution-as-contract enforcement on every LLM emission; Recommendation Provenance Replay archive as the headline pilot test artifact. Phase 2 (post-pilot) reintroduces dispatch and authoritative recording, gated on the five composite trust criteria.

Commercial pilot narrative leads with the **harness as the product** (BYOK + constitution-as-contract + provenance archive), supported by the ECM proof point (Dr. Fischer sign-off on shadow-vs-paper match rate + Replay archive integrity) and the Live Board / Kondukt USP (multi-subsystem visibility + LLM-native skill invocation + free-form research + artifact authoring).

**Resource Requirements:** One full-stack dev (React + Python/FastAPI), one AI/ML engineer (constitution + skills authoring, schema/validator/linter design, LLM output sanitisation, BYOK integration), one DevOps/infra (Docker, MQTT subscriber, PostgreSQL + pgvector, content-addressed blob store for provenance archive, allowlist egress proxy).

---

### Complete Feature Set

**Must-Have Capabilities — Harness Infrastructure:**

| Capability | Rationale |
|---|---|
| MQTT subscriber for IntentPacket ingest (incl. TCMS via brain VLAN 7 SNMP forward) | Foundation — the data layer the harness operates over |
| PostgreSQL + pgvector single-engine store | All IntentPackets + telemetry + recommendations + audit triplets land here. NIS2-defensible single-instance. |
| Rail MCP Server — HTTP MCP endpoint exposing structured tools (telemetry per subsystem, depot state, MTTR computed by harness from Stadler alarm IntentPacket timestamps, recurring-failure detection, semantic search, neglect-risk scoring inputs, xlsx/docx artifact builders) | Customer-LLM's access surface to the landside data layer |
| BYOK key vault — encrypted at rest, per-user, never logged, token usage transparent | Customer-provided LLM with NIS2-defensible key handling |
| Allowlist egress proxy — curated approved-domains list (vendor portals, CENELEC, ERA, EU regulatory texts) | Deep-research capability without open-web compliance risk |
| Constitution + Skills registry, versioned in-product | Operator can inspect what Kondukt is told to be. No hidden prompting. |
| **Per-skill output schemas + provenance validators + advisory-language linter** | The constitution-as-contract enforcement layer. Failed validations suppress. The architecture spine. |
| **Recommendation Provenance Replay archive** — append-only Postgres + content-addressed blob store + auditor-runnable CLI | The headline pilot test artifact. Reconciles 1:1 with the shadow audit trail. |

**Must-Have Capabilities — Fleet Manager Harness Views (the three MVP views):**

| Capability | Rationale |
|---|---|
| **Live Board view** — fleet-wide moving map + per-subsystem health (operator-friendly names, no VLAN IDs); group-pivot by subsystem OR by train; **MTTR computed by harness** from landside-observed Stadler alarm IntentPacket arrival timestamps (`t_close − t_open`); recurring-failure detection annotations from Kondukt with provenance; click-through redirect to Kanban detail card on train selection | The multi-subsystem visibility USP. No competitor produces this. Glanceable, not deep. |
| **Kanban view (neglect-risk triage)** — lane-sort by neglect-risk (decay-without-attention), not queue order; Kondukt proposes lane placement + score with provenance; operator moves cards. **Train detail card carries full diagnostic depth** with progressive disclosure: primary view on expansion = subsystem name + VLAN ID + IP + MAC + switch port + firmware version + time-since-fault; secondary view on click = MTTR computed by harness from Stadler alarm IntentPacket timestamps + recurring-failure history + last 24h packet trace + vendor portal deep-link | The continuous all-day surface AND the depth surface. Reframed from "triage queue" because queue-order thinking is engagement, not displacement. |
| **Kondukt view** — chat + side-canvas; skill registry (Daily Brief, Recurring-failure pattern report, Vendor meeting prep doc, Fleet health summary, with the registry evolving via ADR) + free-form chat; Rail MCP tool access + allowlisted internet egress; xlsx/docx export | The LLM-native deep-research + artifact-authoring surface. Both skill invocation (codified routines) and free-form chat (catch-all). The agentic-OS surface that distinguishes the harness from a dashboard. |

**Must-Have Capabilities — Downstream Recommendation-Consumer Surfaces:**

| Capability | Rationale |
|---|---|
| Depot Briefing PWA — ECM Manager pre-arrival brief generated from IntentPackets + harness-summarised fault history | Primary commercial proof point for Dr. Fischer |
| Depot Briefing PWA — ECM 1 blocking state + simple-click sign-off (MVP) + append-only shadow audit trail | Regulatory capability proof. Phase 1 records shadow via single click; Phase 2 promotes to authoritative (deliberate-confirmation gesture reconsidered at that point — hold-to-record spec preserved in design-system reference). |
| Depot Briefing PWA — role-filtered technician feed (Electrical/Mechanical/IT); structured shadow ACK alongside paper | Operational value; required for depot pilot |
| ECM Audit Export surface — JSON export with paper-reconciliation status, supersedes_id correction view, **Replay CLI demonstration surface** | Dr. Fischer's sign-off requirement; reconciliation status + Replay archive integrity are the Phase 2 trust signals |

**Must-Have Capabilities — Shared Infrastructure:**

| Capability | Rationale |
|---|---|
| Append-only `ecm_signoff_events` table with hash-chain + HMAC + IdP-bound identity + supersedes_id | The regulatory audit record (shadow in MVP, authoritative in Phase 2) |
| `recommendation` table + Provenance Bundle blob store + Validation Receipt records | The Provenance Replay archive substrate |
| pgvector semantic search + keyword fallback | Spike gate: P99 <500ms contractual (target <250ms at 50K vectors) before harness LLM stories |
| Operator-action logging + downstream-state polling for recommendation outcome reconciliation | The shadow audit chain's outcome layer |

**Nice-to-Have Capabilities (in scope, lower priority):**

| Capability | Notes |
|---|---|
| Fleet Manager — 48–72h forward planning view | Drop first if resource-constrained |
| Structured JSON export formatting refinements | BOOM-ready but not BOOM-integrated |
| Dismissal signal arbitration queue | Conductor dismissals from brain feed into Fleet Manager threshold tuning; requires brain coordination |

**Spike Gates (must resolve before dependent stories begin):**

| Spike | Gate | Outcome if missed |
|---|---|---|
| pgvector P99: <500ms contractual ceiling / <250ms at 50K vectors internal target (ef_search=64) | Before harness Kondukt semantic-search stories | Keyword fallback only; semantic search deferred |
| BYOK key handling + provider-agnostic constitution (test Anthropic + OpenAI both work against the same constitution + skills) | Before harness LLM-emission stories | Defer harness to single-provider mode if provider-agnostic guarantee not met |
| **Constitution-as-contract validator spike** — per-skill schema + provenance validator + advisory-language linter as a working end-to-end loop on the *Daily Brief* skill (the first skill in the registry). Includes the Replay CLI prototype. | Before any harness view stories | Without this, the harness ships without its safety posture — pilot cannot start |
| **Train-to-landside-DB mapping integrity** — weekly reconciliation between IntentPacket `train_id` and MQTT topic; component MAC/IP registry validation | Before harness goes live in pilot | Mapping integrity is a one-way door at the data layer; without verification, the entire Provenance Replay archive is suspect |
| Recommendation outcome reconciliation semantics (operator-action polling against HAFAS read endpoints) | Before harness recommendation→action chain stories | Reduce scope to single-system recommendations only |
| Allowlisted egress proxy + per-fetch logging | Before Kondukt internet-research stories | Disable internet research; Kondukt operates over MCP-tool data only |
| SAP PM API contract for write-back | Post-MVP gate — SAP not in MVP scope (Phase 2 expansion) | Phase 2 ships HAFAS dispatch only initially; SAP write-back requires API contract with ÖBB TS |
| CRA-readiness tabletop exercise | Pilot manday 20–25 | Goes/no-goes the Phase 2 readiness recommendation independently of the trust criteria |

---

### Risk Mitigation Strategy

**Technical risks:**
- pgvector P99 <500ms contractual, <250ms internal — spike with keyword fallback before harness Chat+Canvas stories
- BYOK key handling + provider-agnostic constitution — spike before harness LLM-emission stories
- **Constitution-as-contract validator failure** — validator + Replay CLI must run end-to-end on at least one skill before any harness view ships. ≥99% replay-verify clean is a pilot pass criterion. Validator failure on any skill in pilot = suppression of that skill until root cause identified.
- **Shadow audit-trail divergence from paper** — explicit reconciliation queue with 24h SLA on flagging divergences; 100% match rate is a Phase 2 gate
- Brain ingest gap — coordinate with brain team on WAL-before-truncate sync; landside surfaces staleness explicitly
- **BYOK LLM unreachable** (customer-provided key invalid, rate-limited, provider outage) — telemetry views remain functional, LLM-driven views show degraded state with no fallback emission ("silence is safe"). See Journey 8.

**Market risks:**
- Dr. Fischer unavailable at pilot manday ~15 — audit trail still ships; sign-off milestone deferred
- Outside auditor for manday 20–25 tabletop unavailable — load-bearing investment; tabletop must happen with named auditor; if unavailable, postpone (don't substitute internal Nomad person)
- **Constitution mis-calibration** (validator over-strict, too many suppressions, harness reads as broken) — suppression rate >5% sustained triggers validator calibration review

**Resource risks:**
- If team is constrained: defer Chat+Canvas allowlisted internet egress (operate over MCP data only); defer Live Board fleet-wide moving map (start per-train view, add fleet-wide later); ship Daily Brief + Kanban + per-train Live Board + Chat+Canvas-with-MCP-only first
- The four MVP views are non-negotiable (Live Board and Chat+Canvas are the USP); any per-view feature is negotiable

### Phase 2 Readiness Gates

Phase 2 (post-pilot release introducing platform-mediated dispatch and authoritative ECM recording) requires the **composite of all seven** criteria:

1. **Recommendation acceptance rate ≥ Roland-confirmed threshold** (working assumption ≥70%), measured per-view (Live Board / Kanban / Kondukt skills / Kondukt free-form), not aggregated
2. **Shadow audit-trail match rate vs paper** = 100% across the full pilot
3. **Recommendation Provenance Replay archive integrity** — ≥99% replay-verify clean; zero recommendations without complete triplets; zero advisory-only language linter breaches
4. **High-confidence inference rate** — ≥80% of emissions HIGH or MEDIUM confidence (working assumption; Roland-confirmed pre-pilot); LOW-rate <15% sustained over any 72h window
5. **Train-to-landside-DB mapping correctness** = 100% across the full pilot; zero misroutes
6. **Dr. Renate Fischer's positive sign-off** on audit-export quality + Replay archive integrity
7. **Outside-auditor tabletop verdict** positive (from manday 20–25 exercise — see §14) + **zero unresolved pilot-kill triggers** in the final 30 days of pilot

All seven required for full Phase 2 progression. Partial satisfaction triggers narrower Phase 2 scope (per §7 Claim 5) — e.g., dispatch on read-only HAFAS rotation updates only, deferring SAP write-back further. Phase 2 is itself phased.

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
| Telematics surface scope | ~~Which Gemini-derived surfaces are MVP vs post-MVP~~ — **resolved 2026-05-25 (harness reframe):** the Unified Telematics Control Panel framing is retired. Three harness views are locked into MVP: Live Board + Kanban + Kondukt (see §8 capability table). Roland's Teil-Projektleitung Telematik work is served by these three views, not by a separate surface. | n/a — closed |
| Harness surface count | ~~Whether the harness ships 5 / 4 / 3 views~~ — **resolved 2026-05-25 (operator correction post-party-mode):** three views — Live Board + Kanban + Kondukt. Daily Brief is a *skill* inside Kondukt, not a separate view. Drift Visibility moved to ECM module. | n/a — closed |
| Constitution-as-contract enforcement | Schema + provenance validators + advisory-language linter as deterministic post-processing on every Kondukt emission. Suppression is MVP failure mode; regeneration deferred to Phase 1.5. | **resolved 2026-05-25 (party mode + operator confirmation):** locked. Per-skill schemas, per-skill provenance validators, advisory-language linter, Replay CLI prototype must all ship before any harness view goes to pilot. |
| BYOK provider-agnostic guarantee | Test that the constitution holds across at least two LLM providers (Anthropic + OpenAI) before pilot — same constitution, same skills, same validators, swappable provider | Spike gate (see §8 Spike Gates) |
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
| Recommendation outcome reconciliation | All operator-reported outcomes verified against downstream-system polling within 30s; unverified outcomes flagged within 24h | Reconciliation queue persistent in `recommendation_outbox`; in-memory queues not permitted |
| Validation Receipt write latency | <500ms per emission (synchronous before emission reaches operator) | Schema check + provenance check + advisory-language linter on the critical path; designed for deterministic post-processing |
| Provenance Bundle write latency | <1s per emission (content-addressed blob store write) | Async eligible for blob write; recommendation can be emitted as soon as triplet write begins; archive read consistency required for Replay CLI |
| Replay CLI verification time | <3s per recommendation on auditor laptop | Auditor runs interactively during tabletop; performance must be acceptable for live demo |
| Kondukt BYOK retry policy | Exponential backoff, budget-capped | Configurable per operator; default 3 retries with 1s/4s/15s delays |

### Security

| Requirement | Target | Context |
|---|---|---|
| ECM shadow sign-off identity | IdP-bound only | Shadow sign-off identity bound to ÖBB SSO (Keycloak in MVP, ÖBB IdP post-MVP). Paper sign-off identity remains ÖBB's existing process. |
| Recommendation acceptance / modification / rejection identity | IdP-bound session at moment of action; recorded against `operator_action` linked to `recommendation_id` | Logs operator-side action against verified identity; does not authorise dispatch (no dispatch in MVP). |
| Constitution + skill registry inspectability | Operator can view the constitution text, the skill schemas, and the validator rules in-product | No hidden prompting; FR22 |
| BYOK key vault | Encrypted at rest, per-user scope, never logged, never transmitted to harness vendor logs; token usage transparent to operator | FR58; NIS2-defensible |
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
| Critical-action step ceiling | Maximum 2 deliberate steps (open + click) | Applies to ECM shadow sign-off (simple-click MVP), recommendation interactions (accept / modify / reject / mark-seen), technician ACK, briefing ack. Standard flows: max 3 steps. Deliberate-confirmation gestures (hold-to-record, etc.) descoped from MVP 2026-05-25. |
| Multi-modal alert spec | Haptic ≥200ms + visual ≥500ms full-bleed + audio confirmation only | Audio is NOT the primary channel in any depot/ops surface. See §UX Constraints "Audio-Alert Policy". |
| Colour-blind state encoding | Every multi-state UI element distinguishable without colour | Shape + label + one-character text token in addition to colour. WCAG 1.4.1 non-optional. See §UX Constraints. |

### Maintainability

| Requirement | Target | Context |
|---|---|---|
| IntentPacket schema (landside consumer) | Tracks brain-owned v1.0.0 | Schema evolution coordinated with brain team |
| Alembic migrations | Reversible, rollback path documented before merge | See reliability: write-lock ceiling 30s |
| Recommendation triplet schema | **Schema of record (harness-model, 2026-05-25):** every Kondukt emission produces a triplet — (a) **Envelope** `{ recommendation_id: uuid, train_id: str, recommended_action: str, advisory_class: enum{disposition, triage_move, annotation, artifact_emission, free_form}, confidence: enum{HIGH, MEDIUM, LOW}, timestamp: iso8601, harness_version: str, model_provider: str, model_version: str, skill_invoked: str [optional — null for free-form], operator_id: str, justification: str [mandatory] }`; (b) **Provenance Bundle** `{ intent_packets_consumed: [packet_id], mcp_queries: [{ tool, args, response_hash }], allowlisted_fetches: [{ url, response_hash }], llm_raw_output: str }` stored as content-addressed blob (SHA-256); (c) **Validation Receipt** `{ schema_check: pass|fail, citation_trace_check: pass|fail, advisory_language_check: pass|fail, modification_distance: float|null [populated on operator-modified LOW emissions], disposition: enum{emitted, suppressed}, suppression_reason: str|null }`. Schema is versioned in-product. `justification` is mandatory; emission without it is rejected by the validator. Schema evolution requires a joint landside ADR before any field addition or removal. | Schema evolution does not break in-flight emissions; Replay CLI is forward-compatible across minor versions |

---

## 11. Functional Requirements

> *MVP is advisory-only and read-only — the harness produces validated recommendations and shadow audit entries, does not dispatch to operational systems, does not record authoritative ECM sign-offs. All FRs below are scoped accordingly. Phase 2 (post-pilot, gated on the seven composite trust criteria) re-introduces dispatch and authoritative recording; the relevant FRs are flagged `[Phase 2 expansion: ...]`.*

### Live Board (Fleet Manager surface — view 1 of 3)

- **FR1:** Fleet Manager can view a fleet-wide moving map showing all in-service trains and all depot trains, with per-train position (last-known GPS from IntentPacket), severity-banded by highest open fault, and a `data_freshness_ts` staleness indicator when ingest is delayed.
- **FR2:** Live Board renders per-subsystem health for every monitored subsystem (CCTV, PIS, intercoms, AFZ, audio, Wi-Fi, switches, Frontkupplung, energy meter, Service FIS/CCTV diagnostic, Zugfunk, TCMS, Stadler diagnostic plane, Firewall RCU). Subsystems are labelled by operator-friendly name only — **no VLAN IDs on Live Board** (VLAN identifiers live in the Kanban train detail card per FR9).
- **FR3:** Fleet Manager can group-pivot Live Board both ways: **by subsystem** ("all CCTV faults across the fleet, by train") OR **by train** ("everything wrong with 4736-101, by subsystem"). Pivot state is preserved per-user across sessions.
- **FR4:** Live Board displays time-since-fault per affected component. **MTTR computed by the harness** from landside-observable Stadler alarm IntentPacket arrival timestamps: `t_open` = landside arrival time of the Stadler "alarm raised" IntentPacket; `t_close` = landside arrival time of the corresponding Stadler "alarm cleared" IntentPacket; `MTTR = t_close − t_open`. Stadler's internal MTTR field is **not** read. Rationale: (a) the landside-observed window is the operator-experienced MTTR; (b) brain→landside forwarding latency is included on both ends (honest number); (c) the harness owns this metric without trusting Stadler's internal accounting. MTTR is surfaced on the Kanban detail card (secondary disclosure), not on Live Board itself.
- **FR5:** Live Board displays Kondukt-generated cross-subsystem correlation annotations (recurring-failure detection, multi-subsystem pattern detection) inline. Each annotation carries its Validation Receipt status and confidence pill; the underlying recommendation triplet is reachable from the annotation (per FR30).
- **FR6:** Selecting a train on Live Board (map marker or per-subsystem row) **redirects to that train's detail card in Kanban with the card auto-expanded** (per FR10). Live Board itself does not expand inline — it stays glanceable.
- **FR7:** Live Board distinguishes stale-data-due-to-geographic-blackout from stale-data-due-to-hardware-fault by correlating the staleness event against the train's GPS position against a regional cellular dead-zone overlay; staleness indicators are labelled with the disambiguated cause.

### Kanban (Fleet Manager surface — view 2 of 3)

- **FR8:** Fleet Manager can view open faults as Kanban cards across four lanes (New / Investigating / Recommendation Pending / Cleared). Lane order on the board is **lane-fixed**; within each lane, cards sort by **neglect-risk score** (highest risk first), not by chronological queue order. Kondukt computes neglect-risk per card with provenance citations (per FR29); operator can override sort with a manual sort toggle.
- **FR9:** Kondukt proposes lane-placement moves with provenance (e.g. "Move card X from New → Investigating because IntentPacket Y was emitted at Z, citing source") rendered as a small Validation-Receipt-backed banner on the card. Operator moves cards manually (drag-and-drop or keyboard); every move logged per FR32.
- **FR10:** Kanban train detail card uses **progressive disclosure** on expansion. Primary disclosure (always visible on expanded card): subsystem name, VLAN ID, IP address, MAC address, switch port, firmware version, time-since-fault per component. Secondary disclosure (revealed on operator click): MTTR computed by harness from Stadler alarm IntentPacket timestamps, recurring-failure history for the component, last 24h packet trace, vendor portal deep-link. Secondary-disclosure click-throughs logged per FR32.
- **FR11:** Kanban filters: by train (multi-select), by subsystem class (multi-select), by lane, by neglect-risk band (HIGH / MEDIUM / LOW). Filter state persisted per-user.
- **FR12:** Fleet Manager can hand over their shift to an incoming Fleet Manager. Handover summary surfaces: all open Kanban cards by lane, all in-flight Kondukt skill invocations from the last shift, all suppressed-emission events from the last shift, all unverified-outcome states. Incoming Fleet Manager sees the handover summary at the top of Kanban on login.

### Kondukt (Fleet Manager surface — view 3 of 3)

- **FR13:** Fleet Manager can invoke a skill from the Kondukt skill registry. MVP starting registry: *Daily Brief*, *Recurring-failure pattern report*, *Vendor meeting prep doc*, *Fleet health summary*. Registry is versioned in-product; operator can inspect skill definitions (constitution excerpt + schema + provenance rules + advisory-language linter constraints). New skills added via versioned ADR.
- **FR14:** Skill invocation produces a validated artifact in the canvas. Every artifact emission passes through the constitution-as-contract layer (per FR23–FR25) before reaching the operator. Failed validations suppress (per FR25); the operator sees the suppression banner with reason.
- **FR15:** Fleet Manager can chat free-form with Kondukt (catch-all for anything not codified as a skill). Free-form chat turns are still validated emissions — every turn produces a Validation Receipt and a triplet in the Provenance Replay archive.
- **FR16:** Kondukt can call Rail MCP tools to ground its emissions in landside data. Available tools (MVP): `get_fleet_telemetry`, `get_depot_state`, `get_mttr_from_stadler`, `get_recurring_failure_pattern`, `search_fault_history` (semantic + keyword), `get_neglect_risk_inputs`, `build_xlsx`, `build_docx`, `fetch_allowlisted_url`. Tool spec is owned by `rail-mcp-server/`.
- **FR17:** Kondukt can use **allowlisted internet egress** via `fetch_allowlisted_url`. Allowlisted domains include Stadler / Siemens / Knorr-Bremse vendor portals, CENELEC, ERA, EU regulatory texts. The allowlist is configured per-deployment + signed off by ÖBB IT Security; every fetch logged with URL, response status, response hash. Off-allowlist URL requests are rejected before fetch.
- **FR18:** Operator can download Kondukt-produced artifacts as `.xlsx` or `.docx`. Downloaded artifacts are working artifacts (NOT part of the audit trail per the audit scope lock); the chat transcript that produced them is in the Provenance Replay archive.
- **FR19:** Kondukt surfaces a structured error when a Rail MCP Server tool call times out. The corresponding emission attempt is suppressed (per FR25); the operator sees: "Kondukt could not complete this skill — the {tool_name} tool timed out. Try again, or simplify the query." Decision logged includes operator's own action taken (or non-action) per FR32.
- **FR20:** Kondukt surfaces a degraded-mode banner when the customer's BYOK LLM is unreachable (sustained 429/5xx; key invalid; rate-limit exceeded). Skill invocations and free-form chat are paused with provider status surfaced. Live Board and Kanban remain functional (they do not depend on the LLM). Previous artifacts and chat history remain readable.
- **FR21:** Kondukt search modes: semantic (pgvector) primary, keyword fallback when pgvector unavailable. UI explicitly labels active search mode for every result; no silent degradation.

### Constitution-as-Contract Enforcement (Harness-LLM safety posture)

- **FR22:** The constitution is **versioned in-product**. Operator can inspect the constitution that Kondukt is bound by (text + diff history). No hidden prompting.
- **FR23:** Per-skill **output schemas** are defined in the skill registry. Every Kondukt emission (skill output AND free-form chat turn) is validated against the relevant schema before reaching the operator. Schema violations cause suppression (per FR25).
- **FR24:** Per-skill **provenance validators** check that every claim in a Kondukt emission cites a verifiable source — an IntentPacket, an MCP tool response, an allowlisted URL fetch — present in the Provenance Bundle. Unsupported claims cause suppression. **Provenance validators are deterministic, no LLM in the validation loop.**
- **FR25:** **Suppression is the MVP failure mode.** When a validator rejects an emission, the emission is logged with reason (`schema_violation` / `unsupported_citation` / `advisory_language_breach` / `tool_timeout` / etc.) and NOT shown to the operator. **Regeneration loop is deferred from MVP** — the LLM is not given a second chance with feedback in MVP; the recommendation is simply suppressed. Operator sees a banner: "Kondukt suppressed a recommendation — reason: {reason}. Run again or simplify your query." [Phase 2 expansion: regeneration loop with bounded retry-with-error-feedback.]
- **FR26:** **Advisory-language linter** rejects emissions containing imperative action language ("dispatch X", "execute Y", "send Z"). The constitution mandates operator-subject phrasing ("Recommend dispatching X", "Operator should execute Y"). Linter is deterministic and unit-tested per skill.
- **FR27:** Confidence indicator on every Kondukt emission. Three-tier visual grammar — HIGH / MEDIUM / LOW — per the 2026-05-25 design-system lock. **LOW-confidence DISABLE rule (party mode 2026-05-25):** LOW-confidence emissions are visibly distinct (outline pill, desaturated rail) AND the operator's primary action surface is the *Modify* path (cursor pre-placed in justification field); the *Accept* path is non-interactive (`aria-disabled="true"`) with tooltip explaining the LOW-confidence policy. Modify-on-LOW must visibly differ from Modify-on-HIGH.
- **FR28:** **Modification distance** captured on every operator-modified LOW-confidence emission (lexical/semantic diff between Kondukt's draft and operator's committed text). Stored as `modification_distance` on the emission triplet; required Phase 2 trust-signal field (memory: design-system lock).

### Recommendation Provenance Replay Archive

- **FR29:** **Every Kondukt emission produces a triplet** at emission time: (a) **Envelope** — `recommendation_id`, `train_id`, `recommended_action`, `advisory_class`, `confidence` (HIGH/MEDIUM/LOW), `timestamp`, `harness_version`, `model_provider`, `model_version`, `skill_invoked`, `operator_id`; (b) **Provenance Bundle** — IntentPackets consumed, MCP queries issued + responses received, allowlisted URLs fetched + content hashes, raw LLM output before parsing; (c) **Validation Receipt** — schema check result, citation-trace check result, advisory-language linter result, `modification_distance` if applicable, `disposition` (emitted / suppressed). Triplet is durable; written before the recommendation reaches the operator.
- **FR30:** The `recommendation` table (PostgreSQL, append-only) holds Envelopes + Validation Receipts. Provenance Bundles are stored in a **content-addressed blob store** (SHA-256 keys, on disk, manifest indexed). No updates ever; no deletes ever. Database-level trigger raises exception on UPDATE/DELETE attempt.
- **FR31:** **Replay CLI** — auditor-runnable command-line tool (Python, ~400 LOC, single binary). Takes a `recommendation_id`, fetches the triplet, re-runs the validator against the stored Provenance Bundle, asserts the Validation Receipt matches. Runs on a laptop without harness infrastructure. **Demonstrated live at manday ~15 (Dr. Fischer review) and manday 20–25 (outside-auditor tabletop).**
- **FR32:** Operator actions on recommendations are recorded as `operator_action` rows linked by `recommendation_id`: action type (`accepted`, `modified`, `rejected`, `ignored`, `kanban_lane_moved`, `card_expanded`, `card_secondary_disclosed`), operator IdP identity, timestamp, optional modification text + `modification_distance`. Together with the triplet, this forms the **recommendation → action → outcome chain** that gates Phase 2.
- **FR33:** Downstream-state polling reconciles operator-reported outcomes. When the operator marks a recommendation acted-upon, the harness polls available downstream read endpoints (HAFAS read, vendor portal read) for verification. Outcome states: `verified` / `self-reported-unverified` / `divergent`. Persistent `self-reported-unverified` outcomes feed Phase 2 trust assessment.
- **FR34:** **Provenance Replay archive integrity check** runs continuously throughout pilot. Aggregate metrics surfaced on the harness ops dashboard: replay-verify-clean rate, recommendations-without-triplet count (target 0), advisory-only language breaches (target 0). Any breach is a pilot-kill trigger (per §3 Pilot-Kill Triggers).
- **FR35:** Provenance Replay archive **export at pilot end** as `pilot-corpus-{start_date}-to-{end_date}.tar.zst`, includes all triplets + the Replay CLI binary + a manifest. Handed to Dr. Fischer + outside auditor.

### IntentPacket Ingest (Landside Consumer)

- **FR36:** System consumes IntentPackets from the brain's MQTT broker on `oebb/{train_id}/intent` topic, including TCMS data the brain polls via VLAN 7 SNMP and forwards.
- **FR37:** System persists ingested packets to PostgreSQL with **idempotent insert on `packet_id`** — safe against duplicate delivery after replay or service restart.
- **FR38:** System surfaces ingest gap explicitly — Live Board and Kanban show staleness when no fresh packet has arrived per `data_freshness_ts` threshold.
- **FR39:** System handles unknown `agent_state` values (future brain schema versions) as raw events rather than silently discarding.
- **FR40:** System maintains `intent_packet_schema_registry` keyed by `schema_version`. On every inbound IntentPacket, diff observed fields against registry. Unknown field → log `intent_packet.unknown_field` event + metric; do NOT drop the packet. Major version mismatch → quarantine + alert. Minor version mismatch → log + accept.
- **FR41:** **Train-to-landside-DB mapping integrity.** Weekly automated reconciliation: every IntentPacket arrived on `oebb/{train_id}/intent` must match its `IntentPacket.train_id` field; every registered component MAC/IP must belong to its mapped train; mismatches halt harness data writes for the affected train pending reconciliation (per §3 Pilot-Kill Triggers). Surfaced as `mapping_health` metric on the harness ops dashboard.

### Depot Briefing PWA (recommendation-consumer surface)

- **FR42:** ECM Manager can view a pre-arrival briefing for an inbound vehicle, generated from the vehicle's IntentPacket history + Kondukt-summarised fault context. Pre-arrival window is triggered by HAFAS scheduled arrival time (via `get_hafas_timetable`) — the IntentPacket schema v1.0.0 carries no position field. Brief is considered "pre-arrival" when opened before the HAFAS scheduled arrival timestamp.
- **FR43:** ECM Manager can see all open faults on a vehicle, organised by technician discipline (Electrical, Mechanical, IT).
- **FR44:** ECM Manager can view a vehicle's blocking state when all technician faults are resolved but ECM 1 authorisation is pending.
- **FR45:** ECM Manager can review a full fault resolution summary and sign off on a vehicle's return to service via a **single click** ("Record sign-off") in MVP. The click writes the `ecm_signoff_events` shadow row with the operator's IdP identity captured at moment of click. Append-only enforcement at DB level (FR51) makes the record immutable; no client-side deliberate-confirmation gesture in MVP. **MVP: the click records a shadow sign-off; the authoritative sign-off remains paper.** [Phase 2 expansion: when the sign-off becomes authoritative, the deliberate-confirmation gesture (hold-to-record: 3s hold + 5s countdown + atomic write at expiry) — preserved in design-system reference at `design-artifacts/ecm-signoff-interaction-spec.md` — is reconsidered for Phase 2.]
- **FR46:** ECM Manager can view a post-sign summary of exactly what was committed immediately after sign-off (Protokoll-ID, vehicle, location, timestamp, technician chain).
- **FR47:** ECM Manager can correct a sign-off record by creating a superseding record (`supersedes_id` pointing to the corrected row), without modifying or deleting the original. Corrections apply identically to the shadow trail in MVP and to the authoritative trail in Phase 2.
- **FR48:** ECM Manager can export the full audit trail for a vehicle as structured JSON. Export includes paper-trail reconciliation status (`matched` / `unverified` / `divergent`) for every shadow record AND includes the Recommendation Provenance Replay triplets for every harness recommendation that touched the vehicle's resolution chain.
- **FR49:** Technician can view a role-filtered fault feed showing only faults within their discipline.
- **FR50:** Technician can acknowledge a fault with a structured ACK record. **[NEEDS DECISION before story begins: ACK schema fields, fault-code vocabulary, whether free-text notes are permitted, whether secondary confirmation is required. Input from an ECM Manager or ECM 4 Technician required.] Under advisory-only, ACK is a shadow record alongside the existing paper ACK process; every shadow ACK references its corresponding paper-trail entry.**
- **FR51:** System maintains an **append-only, hash-chained** sign-off event log (`ecm_signoff_events` table) with IdP-bound identity, vehicle state snapshot, directive reference (`EU 2019/779`), and HMAC tamper-evidence signature. `signature` field provides per-row tamper evidence; hash chain links each row to previous (`sha256(prev_row_hash || row_hmac)`) proving both non-modification AND non-deletion across the trail. **MVP: shadow record. [Phase 2 expansion: authoritative ECM record.]**
- **FR52:** Reconciles every harness shadow sign-off against the corresponding paper sign-off entry within 24h of paper signature. Reconciliation status (`matched` / `unverified` / `divergent`) recorded against the shadow record and surfaced on audit-trail export.

### Identity, Access & Audit

- **FR53:** All compliance actions are bound to an **IdP-bound** identity, not a local session. **MVP uses Keycloak with manually created users.** Identity binding applies to ECM shadow sign-offs (FR45), Kanban lane moves (FR9), Kondukt skill invocations (FR13), free-form chat turns (FR15), card-detail expansions (FR10), and technician shadow ACKs (FR50). ÖBB SSO IdP integration is post-MVP.
- **FR54:** Each harness surface enforces role-based access. For controls a user's role cannot execute, the control is **rendered but visually disabled** with a tooltip explaining the permission gap. Server-side authorization on the action endpoint is the binding enforcement layer; the disabled UI state is defence-in-depth.
- **FR55:** System records every Kanban lane move, sign-off, dismissal, Kondukt skill invocation, free-form chat turn, recommendation acceptance/modification/rejection, and status change as a server-side write to PostgreSQL — never component state.
- **FR56:** **Simple-click sign-off (MVP)** for ECM shadow sign-off (FR45). Single click on "Record sign-off" writes the row. Idempotency key generated client-side at click, sent as `X-Idempotency-Key` header; safe retries (409 Conflict treated as success). After write, record is permanent — no undo (append-only DB trigger per FR51). **Descoped from MVP 2026-05-25:** the hold-to-record deliberate-confirmation gesture (3s hold + 5s countdown + atomic write at expiry). The full gesture spec is preserved in `design-artifacts/ecm-signoff-interaction-spec.md` as a **Phase 2 candidate** — when the platform's sign-off becomes authoritative under EU 2019/779, the deliberate-confirmation gesture is reconsidered. **Glove-gesture acceptance criteria + ECM 4 Technician walkthrough remain in the design-system reference for Phase 2 — not a pre-MVP-pilot gate.**

### Security & Compliance Events

- **FR57:** System emits a structured **security incident event** to ÖBB's security monitoring pipeline for each of: authentication anomaly (failed IdP verification on a safety-relevant action), unauthorised action attempt, BYOK key exposure detection (key appears in any logged field), LLM injection detection (validator rejects a Kondukt response with `advisory_language_breach` or suspected prompt-injection schema violation). **Payload supports full NIS2 24-72-30 reporting cadence + CRA Article 14** — fields: `event_type`, `surface`, `user_identity` (if known), `timestamp`, `severity`, `scope`, `affected_systems`, `cra_extension: { product_identifier, cve_ids, actively_exploited, coordinated_disclosure_status }`. ÖBB IT Security owns the regulatory filing; the harness owns event emission.
- **FR58:** **BYOK key handling.** Customer-provided LLM API keys are encrypted at rest, scoped per user, never logged, never transmitted to harness vendor logs. Token usage is transparent to the operator on every Kondukt session (cumulative + per-skill). Key invalidation surfaces immediately as the FR20 degraded banner.

### Resilience & Consistency

- **FR59:** System provides a defined fallback state when the brain ingest pipeline is unavailable — last-known values displayed with explicit unavailability label; Live Board cards show staleness amber border per FR1; Kondukt cannot ground new emissions in stale data, validator rejects emissions with stale provenance per FR24.
- **FR60:** System maintains write consistency for cross-boundary operations — partial failures are logged and surfaced for reconciliation, never silently dropped. **MVP: cross-boundary operations are operator-executed via existing systems; partial-failure surfacing applies to harness shadow recording + downstream-state polling. [Phase 2 expansion: applies to platform-side dispatch.]**
- **FR61:** **Recommendation outbox.** All Kondukt emissions and their reconciliation state persist in `recommendation_outbox` (PostgreSQL table, `idempotency_key UNIQUE`); in-memory queues are not used. Survives harness restart without loss. Worker polls + `SELECT ... FOR UPDATE SKIP LOCKED`.

### Phase 1 Trust & Phase 2 Readiness

- **FR62:** System captures every recommendation with the full recommendation → action → outcome chain: `generated_at`, `displayed_to_user`, `user_action_taken` (executed / modified / rejected / ignored), `action_logged_at`, downstream-state polling result, `outcome_verified_at`. This is the Phase 2 trust-evaluation substrate, reconciling 1:1 with the Provenance Replay archive.
- **FR63:** System computes and displays the **seven-criteria Phase 2 trust-criteria composite** continuously with separate visible threshold lines for each: (a) recommendation acceptance rate per-view, (b) shadow audit-trail match rate, (c) Provenance Replay archive integrity (replay-verify-clean rate), (d) high-confidence inference rate (HIGH+MEDIUM), (e) train-to-landside-DB mapping correctness, (f) Dr. Renate Fischer qualitative sign-off status, (g) outside-auditor tabletop verdict + pilot-kill-trigger history. All seven required for full Phase 2; partial satisfaction triggers narrower Phase 2 scope per §7 Claim 5.
- **FR64:** System generates a **Phase 2 Readiness Report** at pilot end (and continuously available throughout pilot) summarising the seven-criteria composite, the recommendation outcome dataset, the shadow audit-trail integrity proof, the Provenance Replay archive integrity proof, and the outside-auditor verdict — packaged for the Roland Ruisz / Martin Lerch / Dr. Fischer sign-off meeting.

### Dispatcher Coordination *(post-MVP — Dispatcher persona deferred from MVP scope)*

- **FR65:** *(Post-MVP)* Dispatcher relay surface — relay card with LLM-generated passenger-safe language, copy-to-clipboard, forward-to-conductor push notification dispatched via brain.

---

*This is the binding capability contract — **65 FRs across 12 capability areas.** Any feature not listed here will not exist in the final product unless explicitly added via versioned ADR. The Kondukt skill registry is versioned-in-product; new skills are additions to FR13's registry, not new FRs.*

**Retired from previous PRD versions (logged here for traceability):**
- Telematics Project Management — Unified Control Panel (old FR36–FR58 Zone 1/2/3 framing) — collapsed into Live Board + Kanban
- Warranty evidence packaging (old FR41) — retired from MVP; detection capability surfaces via Live Board recurring-failure annotations + Kondukt skill registry (Vendor meeting prep doc)
- OTA rollout monitoring (old FR39) — retired from MVP
- Configuration drift detection as a daily-harness FR (old FR40) — moved to ECM module (quarterly cadence)
- Workshop reservation tool `reserve_maintenance_slot` (old FR53) — retired
- Lastenheft procurement clause export (old FR55) — vision tier
- Dismissal signal arbitration as MVP FR (old FR16) — moved to post-MVP per-alert-type accuracy tuning (post-MVP)

*Onboard brain capabilities (Conductor App, Hailo inference, IntentPacket emission, departure clearance, shift-aware density, edge resilience) live in `oebb-brain/prd.md` — see `oebb-brain/` directory for the companion PRD.*

---

## 12. Phase 2 Readiness Framework

Phase 2 of the platform — platform-mediated dispatch and authoritative ECM recording — is a **separate post-pilot release** gated on the five composite trust criteria. This section consolidates the framework; cross-references FR59–FR62, §8 Phase 2 Readiness Gates, and §7 Claim 5.

### Trust criteria (composite — all seven required for full Phase 2)

1. **Recommendation acceptance rate** ≥ Roland-confirmed threshold (working assumption ≥70%), measured per-view (Live Board / Kanban / Kondukt skills / Kondukt free-form), not aggregated. Modification + rejection rates also tracked.
2. **Shadow audit-trail match rate vs paper** = 100% across the full pilot — every paper sign-off has a matching harness shadow record; persistent divergences disqualify
3. **Recommendation Provenance Replay archive integrity** — ≥99% of emissions replay-verify clean via the auditor-runnable CLI; zero recommendations without complete triplets; zero advisory-only language linter breaches across the full pilot
4. **High-confidence inference rate** — ≥ Roland-confirmed threshold (working assumption ≥80% HIGH+MEDIUM); LOW-confidence rate <15% sustained over any 72h window
5. **Train-to-landside-DB mapping correctness** = 100% across the full pilot; zero misroutes between IntentPacket-claimed `train_id` and harness data path; zero un-mapped components
6. **Dr. Renate Fischer's positive sign-off** on audit-export quality + Provenance Replay archive integrity (Journey 4 outcome at manday ~15 and continuous review thereafter)
7. **Outside-auditor tabletop verdict** positive (from the manday 20–25 exercise — see §14) + **zero unresolved pilot-kill triggers** in the final 30 days of pilot

### Composite scoring rules

- Each criterion is independently measurable and displayed with its own threshold line in the Phase 2 Readiness dashboard (FR63).
- Criteria 1, 2, 3, 4, 5 are quantitative; 6 and 7 are qualitative go/no-go.
- Full satisfaction → recommend full Phase 2 (HAFAS dispatch + ECM authoritative recording).
- Partial satisfaction → recommend **narrower Phase 2 scope**:
  - e.g., only criterion 2 fails → recommend phased Phase 2: dispatch enabled, authoritative ECM recording deferred until match-rate gap is closed
  - e.g., only criterion 3 fails → defer all Phase 2 (Replay archive integrity is the load-bearing safety artifact; without it no dispatch)
  - e.g., only criterion 7 fails → defer all Phase 2 until tabletop exercise is re-run with auditor sign-off
  - e.g., criterion 4 borderline → narrower Phase 2 scope on lower-stakes recommendations only; calibration review before full progression
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

**"Kondukt produced a recommendation in the Daily Brief skill that cited bus-controller memory utilisation as the root cause of reservation-screen blackouts on three DOSTO Neu units. The Disponent followed the recommendation, took the action via Stadler's vendor portal, and the resulting Stadler diagnostic state turned out to mismatch what Kondukt cited. Who is accountable — the customer's LLM provider (BYOK), Nomad's harness (constitution + validators), or the operator's execution? Where does the audit trail live? Can the Provenance Replay archive reproduce the original Kondukt emission from artifacts alone, and does the validator confirm what it claimed at emission time? What does the 24h NIS2 clock say? What does CRA reporting require?"**

The scenario exercises the CRA + NIS2 + ECM trail intersection AND the four-way ambiguity inherent in a harness-LLM-customer-operator chain: was the wrong outcome caused by the LLM (the customer's choice of model + key), the harness (constitution + validators that should have caught the issue), the operator's execution, or a stale-data condition in the IntentPacket layer? The Provenance Replay archive is the load-bearing forensic artifact.

**Live demonstration during the tabletop:** the outside auditor runs the **Replay CLI** on a laptop against the recommendation in question. The validator re-runs against the stored Provenance Bundle, deterministically; it either confirms the original Validation Receipt (the harness did what it said it did at emission time — the issue is downstream of emission) or it surfaces a divergence (the harness has a validator-vs-runtime drift — existential bug). This is the moment that matters in the room.

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

## 15. Phase 1 Sprint Scope (harness-model revised)

Engineering must-act-now list, eng-day sized at PoC scale (1–5 trains, 10–20 vehicles). Two sprints.

### Sprint 1 — must-merge-before-pilot-day-1 (~7 eng-days)

| # | Item | Size | Why |
|---|---|---|---|
| 1 | **Constitution-as-contract validator** for first skill (Daily Brief) — per-skill schema + provenance validator + advisory-language linter; Pydantic + deterministic post-processing; suppression on validation failure (no regeneration in MVP) | 1.5d | Load-bearing. Without it, the harness ships without its safety posture. Pilot cannot start. |
| 2 | **Recommendation Provenance Replay archive** — `recommendation` table (Postgres append-only) + Provenance Bundle content-addressed blob store (SHA-256) + Validation Receipt write path | 1.0d | The headline pilot test artifact. One-way door at the data layer. |
| 3 | **Replay CLI prototype** — auditor-runnable single binary; takes `recommendation_id`, re-runs validator against stored bundle, asserts Receipt matches | 0.5d | Demonstrated at manday ~15 and manday 20–25. |
| 4 | **Kondukt three-tier confidence visual grammar** (consume Sally's spec) — HIGH/MEDIUM/LOW + LOW-confidence DISABLE rule + Modify-on-LOW pre-placed cursor + modification_distance capture | 1.0d | Operator sees confidence on day 1; LOW-confidence DISABLE is locked. |
| 5 | **PWA wake-from-sleep auth gate** (simplified, no SCA) | 0.8d | Disponent will background the tab; without this, day-1 demo breaks on first lunch |
| 6 | **Shadow audit-trail with hash-chain + append-only DB trigger** | 1.0d | ECM persona in pilot scope; one-way door on integrity |
| 7 | **Brain schema-bump trip-wire** (`intent_packet_schema_registry`) | 1.0d | Cross-workspace contract |
| 8 | **Train-to-landside-DB mapping verifier** — weekly reconciliation between IntentPacket `train_id` and MQTT topic + component MAC/IP registry validation | 0.5d | Pilot-kill trigger if not in place; data-layer one-way door |
| 9 | **Recommendation outbox** (`recommendation_outbox` durable Postgres queue, `idempotency_key UNIQUE`) | 0.5d | Survives harness restart; in-memory queue is a latent data-loss bug |
| 10 | **BYOK key vault** — encrypted at rest, per-user, never logged; token usage exposure to operator | 0.5d | NIS2-defensible; one-way door on key handling |
| 11 | Stale-data banner state (shared by Live Board + Kanban + Kondukt degraded mode) | 0.5d | Shared dep |

### Sprint 2 — must-merge-before-PoC-ends (~5 eng-days + 1 policy-day)

| # | Item | Size | Why |
|---|---|---|---|
| 1 | **Allowlisted egress proxy** — domain allowlist + per-fetch logging + content hashing of fetched responses | 1.0d | NIS2-defensible internet research; one-way door on egress policy |
| 2 | **Structured `IncidentReport` endpoint** (shared NIS2 + CRA, 24-72-30 cadence) | 1.5d | NIS2 + CRA reporting; one-way door if schema unstructured at audit time |
| 3 | **BYOK provider-agnostic spike** — test the same constitution + skills + validators against Anthropic + OpenAI; confirm both work | 1.0d | Spike gate for the "harness is provider-agnostic" claim; required before pilot |
| 4 | **SBOM in build pipeline** (cyclonedx or syft) | 0.5d | CRA reporting |
| 5 | **`SECURITY.md` + `.well-known/security.txt` + signed releases** (cosign) | 0.5d | CRA disclosure process |
| 6 | **Phase 2 Readiness dashboard** — seven-criteria composite display with separate threshold lines (FR63) | 1.0d | Continuous visibility throughout pilot; Phase 2 sign-off artifact |
| 7 | **Vulnerability handling process doc + threat model** | 1.0d policy | Owner: Abbas + secops. Written-policy-not-code. |

### Explicit non-scope (NOT building in MVP)

- Live ENISA API integration (stub only — Phase 2)
- Automated CVE scanning in CI (Dependabot is enough; full Snyk/Aqua is post-commercial)
- Public security advisory page (CHANGELOG is enough until we have customers)
- Pen-test report (separate exercise, separate budget, pre-commercial not pre-PoC)
- ISO 27001 / SOC 2 alignment work
- Confidence-based dispatch routing (Phase 2)
- HAFAS write-back / SAP write-back / BOOM integration (Phase 2)
- Harness-side dispatch reconciliation queue (Phase 2 — MVP queue is for operator-action outcomes only)
- **Regeneration loop** on validator failure (Phase 1.5 — MVP failure mode is suppression, "silence is safe")
- **Warranty recovery workflow** (Phase 1.5 if pilot evidence supports — detection is in MVP via Live Board + Kondukt)
- **OTA rollout management** (vendor tooling remains canonical)
- **Drift Visibility** as a daily-harness view (moved to ECM module — quarterly cadence)
- **Multi-tenant deployment** (Phase 2; PoC is single-instance per operator)
- **ÖBB SSO IdP integration** (post-MVP change request — MVP uses Keycloak with manually-created users)

### Tabletop exercise

Add 2-hour tabletop exercise at pilot manday 20–25 (§14). Outside auditor cost: half-day fee — load-bearing investment.

---
