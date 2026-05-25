# Project Brief: ÖBB Landside Platform

> Simplified Brief — Essential context for design work

**Created:** 2026-05-25
**Author:** Abbas Rizvi
**Brief Type:** Simplified
**Mode:** Suggest (synthesized from prd.md, commercial-model, GTM notes, ÖBB role research, UX decisions)

---

## Project Scope

A landside fleet operations platform for ÖBB's rail operations team — three coordinated surfaces sharing one IntentPacket stream from the onboard "brain" plus vendor telematics:

1. **Fleet Manager AI Interface (web)** — Disponent receives a real-time triage feed of fleet disruptions. Kondukt (operator-owned BYOK LLM) produces structured **Recommended Action Objects (RAOs)** — multi-system action proposals rendered as readable cards with step-by-step execution guidance.
2. **Depot Briefing PWA (mobile)** — ECM Manager and ECM 4 Technicians (Electrical, Mechanical, IT) receive a role-filtered fault brief before a vehicle docks; one information model, four role lenses.
3. **Telematics Control Board (web)** — Teil-Projektleitung Telematik monitors Probebetrieb commissioning, OTA rollouts, configuration drift, and packages end-to-end network-path evidence for vendor warranty disputes.

Plus two cross-cutting capabilities:

- **ECM audit trail** — append-only, IdP-bound, HMAC-signed, hash-chained record under EU 2019/779. MVP runs as a **shadow audit** alongside the authoritative paper sign-off; Phase 2 promotes it to authoritative.
- **Rail MCP Server** — Kondukt BYOK integration; operator brings their own LLM API key; platform produces structured RAOs.

**Scope boundary:** The onboard brain (Hailo-10H inference, Conductor App PWA, IntentPacket schema, edge resilience) is a separate product in `oebb-brain/`. Landside consumes; brain does not consume landside output. SAP PM, ServiceNow, BOOM are post-MVP.

**Execution posture (MVP):** Advisory-only for the 6-month pilot. The platform recommends; operators act via existing systems (HAFAS direct, paper sign-off, radio); the platform shadows what they did. Phase 2 — platform-mediated dispatch and authoritative recording — is gated on five composite trust criteria measured at pilot end.

---

## Challenge / Opportunity

**The problem.** ÖBB's landside operations make high-stakes decisions across fragmented systems: vendor diagnostic portals, Power BI dashboards, ServiceNow tickets, vendor work orders, MS Project schedules, Excel reconciliation files. Each switch costs cognitive load; each gap is a place a fault goes unattributed. The Disponent's "cost per delay-minute" KPI exists but is invisible at the moment of decision. The ECM Manager's regulatory sign-off under EU 2019/779 is paper-based and reconstructable only after the fact. The Telematics Sub-PM cannot prove "is it our landside or the vendor's hardware" during Probebetrieb without manually correlating logs across 4–7 systems.

**Why now.** Three forces converge:
- **EU AI Act + NIS2 + ECM Directive 2019/779** raise the cost of opaque, ungovernable rail-operations AI; advisory-only posture with human oversight and an append-only audit trail is the compliance-native architecture.
- **Onboard edge inference** (Hailo-10H) is newly viable, producing structured IntentPackets — a feed competitors with cloud-only architectures cannot match.
- **ÖBB internal driver** — Roland Ruisz (operations, dual-hatted Disponent + Teil-Projektleitung Telematik) has named the Probebetrieb warranty-evidence problem and the cost-per-delay-minute visibility gap as live operational pain. The pilot window is end-Aug / Sept 2026 (manday 1), with company formation and Hailo procurement as pre-pilot blockers.

**The opportunity.** Build the platform that closes the *information* gap during the pilot, earn trust through five measurable criteria, then close the *execution* gap in Phase 2 — becoming the regulated-rail operations system of record for the operator.

**Differentiators that competitor cloud dashboards cannot replicate:**
- Prevented delay-minutes counterfactual — measurable, ÖBB-validated KPI rendered at decision time.
- ECM audit trail engineered as the regulatory record under EU 2019/779, not as a log.
- Kondukt RAOs — structured multi-system action proposals, not data dashboards.
- Vendor warranty leverage through end-to-end network-path evidence during commissioning.

---

## Design Goals

**Functional**

- Disponent makes a redeployment decision with cost consequence visible *before* confirming — every time, not as a special view.
- ECM Manager reviews a structured fault brief before a vehicle docks (>80% of arrivals); platform's shadow record matches the paper sign-off field-for-field (100%).
- ECM 4 Technician logs a resolution ACK that appears in the ECM Manager's pre-arrival brief before docking.
- Teil-Projektleitung Telematik resolves a vendor warranty dispute in a single session using platform-packaged evidence.
- Recommendation acceptance rate ≥ 70% (working assumption, confirmed with Roland) within 30 pilot mandays.

**Experience**

- **Glanceable under pressure** — Disponent is on a 12-hour rotating shift, often under disruption stress. Information density must be high but readable in one scan.
- **Plain language for sign-off, technical depth for hands-on** — same fault, two presentations, governed by role on a single card schema (`summary` vs `technical_detail`).
- **Decision-traceable** — every shadow audit entry answers: who decided, when, what action was taken, what was the outcome.
- **Recommendation, not command** — the Disponent reads, judges, executes elsewhere, logs back. The platform's tone is co-pilot, not autopilot. Phase 2 changes this contract; MVP must not pre-empt it.
- **One information model, role-filtered** — Depot Briefing PWA uses progressive disclosure by role (ECM Manager | Electrical | Mechanical | IT), not tabs.

**Business**

- Phase 2 readiness verdict at pilot week 24, with the five trust criteria visibly tracked weekly throughout the pilot.
- Reference customer + validated ROI numbers + named pipeline (~€470K ARR fleet potential) sufficient to support a fundable event with Speedinvest / i5invest.
- Commercial wedge through the Telematics Sub-PM persona's warranty-dispute use case (non-obvious to competitors).

---

## Constraints

**Regulatory (one-way doors)**

- **EU ECM Directive 2019/779** — ECM audit trail is a first-class requirement; in MVP it shadows paper, in Phase 2 it must be authoritative-ready.
- **EU AI Act** — high-risk classification likely; advisory-only posture is compliance-native and must not regress in MVP.
- **NIS2** — supplier-chain requirements apply to Nomad as a vendor with footprint on ÖBB's operational network.
- **GDPR** — DPA required before any ÖBB operational data touches Nomad systems.

**Architectural**

- **IntentPacket schema v1.0.0** is owned by `oebb-brain/`; landside is a downstream subscriber. Schema currently carries no GPS/position field — pre-arrival trigger defaults to HAFAS scheduled arrival until/unless brain adds position.
- **Kondukt BYOK** — operator brings their own LLM API key; platform must not require Nomad to hold operator LLM credentials.
- **Single ÖBB instance** for PoC; per-operator deployment at fleet rollout.
- **Hailo hardware** — one Hailo-8 + Hailo-10H M.2 pair per consist; platform fee unit is 1 train consist.

**Process / commercial**

- Pilot: **3 consists, 6 months, €15,000 total** (€7,500 on signature, €7,500 week 4) — below ÖBB's procurement-committee threshold so Roland can approve directly.
- Manday 1: end-Aug / Sept 2026; Hailo procurement and Einzelunternehmen registration are pre-pilot blockers.
- Pilot success gate: composite health score + Roland Ruisz sign-off + Renate Fischer sign-off + outside-auditor tabletop verdict + zero unresolved pilot-kill triggers in final 30 days.
- **Persona scope** — Dispatcher (Fahrdienstleiter) deferred from MVP. Primary personas: Disponent (Roland), ECM Manager, ECM 4 Technician, Teil-Projektleitung Telematik (Roland, same person, second hat).

**Design / brand**

- One-person company (Einzelunternehmen, Vienna) — no design system, no existing brand assets to honour. Greenfield surface.
- Bias toward smaller surface + earned trust over over-designed systems (operator preference: Abbas).

---

## Next Steps

This simplified brief provides essential context for design work. The following phases can now proceed:

- [ ] **Phase 2: Trigger Mapping** — connect business goals (delay-minute reduction, ECM audit readiness, warranty leverage, Phase 2 trust criteria) to user psychology per persona (Disponent under disruption stress, ECM Manager pre-arrival, Technician on bay, Telematics Sub-PM packaging vendor evidence).
- [ ] **Phase 4: UX Design** — proceed with surface-by-surface specs once Trigger Map is locked.
- [ ] **Phase 5: Design System** — not enabled for MVP (`design_system_mode: none`); revisit at fleet rollout.

---

_Generated by Whiteport Design Studio — Saga / Suggest mode_
