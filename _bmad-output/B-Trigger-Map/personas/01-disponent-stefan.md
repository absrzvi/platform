# Persona Detail: Stefan the Disponent (Fleet Manager)

**Priority:** Primary — daily user, RAO acceptance owner, prevented-delay-minutes carrier
**Real-world equivalent:** Roland Ruisz (ÖBB operations, dual-hatted with Telematik PM)
**Surface:** Fleet Manager AI Interface (primary), shadow audit log (secondary)

---

## Profile

Stefan runs a 12-hour rotating shift in the Wien operations centre — day/night/weekend rotation, weekend availability required. He's solution-focused, stress-resilient, accountable for fleet availability and cost. His role formally is *Disponent:in im Eisenbahnwesen* — rail operations dispatcher / fleet coordinator. He plans and assigns vehicles and locomotive operators to services, coordinates maintenance and service schedules, monitors ongoing traffic, responds to disruptions, and collaborates with workshops and vehicle maintenance.

He works monitor-heavy with multiple systems open — HAFAS for rotations, vendor diagnostic portals for telemetry, Power BI for historical performance, ServiceNow for tickets. Switching between them mid-incident is where decisions get slow and traceability gets thin.

## Behavioural anchors

- **High-frequency / high-stakes** — he makes redeployment decisions multiple times per shift, each with cost consequence
- **Cross-system orchestrator** — IntentPackets with `request_landside_tasking: true` land here; he is the routing person
- **KPI-aware** — "prevented delay-minutes" maps directly to cost management framing he already owns
- **Crisis-mode native** — he is the entry state when a disruption fires, and the platform's full-width crisis view is built around that mental mode

---

## Positive Drivers (Wants)

### W1. Cost consequence visible **before** confirming a redeployment
Stefan doesn't want a dashboard that shows the cost *after* he's clicked. He wants the cost-impact one-liner *on the triage card*, before he opens the recommendation. Then the RAO refines the number when he asks "what does deploying 4819 cost?" The platform's job is to put the number in front of him at the moment he is about to make the call.

### W2. Receive structured proposals he can judge, not raw telemetry he has to translate
The RAO is the format. It contains the system, method, payload, justification, step-by-step procedure, and confidence. Stefan reads, judges, executes in the trusted system (HAFAS), and logs back. He gets to apply human judgment to a fully-specified proposal — the cognitive load shifts from translation to evaluation.

### W3. Build muscle memory for the gesture that becomes authoritative in Phase 2
The 3-second-hold → 5-second-countdown → atomic-write gesture is preserved from MVP shadow audit into Phase 2 authoritative record. Stefan learns it under low-stakes conditions (paper is the safety net in MVP) so that when Phase 2 ships, the gesture is reflex.

### W4. Be the operator whose shift produced the prevented-delay-minutes number Roland walks into the board meeting with
This is identity, not just function. Stefan is the persona whose decisions are the evidence for Phase 1 ROI and Phase 2 trust. The platform makes his impact legible.

---

## Negative Drivers (Fears)

### F1. Autopilot accusation
If anything in the UI grammar suggests the platform executed the call, Stefan wears the consequence. The advisory-only contract has to be visible in the verbs, the buttons, the recording surface — "Mark executed via HAFAS" not "Execute," "Recommendation" not "Action," shadow audit not authoritative record.

### F2. Reconstructability gap
If an incident is escalated and Stefan can't reconstruct who decided, when, and what action was taken from the platform's record, his liability widens. Every shadow entry must answer who-when-what-outcome.

### F3. A "co-pilot" that turns out to be a slower way to do what he already does in HAFAS
If the platform adds steps without saving steps, adoption collapses inside the pilot and the acceptance rate never crosses 70%. Step-by-step execution guidance has to be *additive* — the procedure Stefan was going to follow anyway, now pre-staged with the HAFAS payload pre-computed.

### F4. Phantom recommendations
RAOs that fire without an underlying fault, fire in the wrong direction, or recommend something safety-adjacent that isn't sound. One phantom in the early pilot mandays poisons acceptance for the next 30. The tabletop exercise (mandays 20–25) explicitly tests this.

---

## Must-address design implications

| Driver | Design implication |
|---|---|
| W1 | Cost-impact one-liner is **part of the triage card**, not a drill-down. RAO refines but does not replace it. |
| W2 | RAO renders as a readable card with payload + justification + step-by-step + "Mark executed" button — never as a "do it" button. |
| W3 | Hold-to-record gesture identical across MVP shadow and Phase 2 authoritative writes. UI shift between modes happens behind the gesture, not in the gesture itself. |
| W4 | Pilot evidence dashboard surfaces "decisions you made this shift" as a personal-impact view — not just an aggregate metric. |
| F1 | Recommendation framing: every button verb tested against "could this read as the platform acting." "Mark executed," "Record sign-off," "Forward to conductor" — operator is always the subject. |
| F2 | Every RAO captures: generated_at, displayed_to_user, user_action_taken, action_logged_at, outcome_observed_at — and every one of those is visible to Stefan in the trail, not just stored. |
| F3 | First sprint validates the time-to-decision baseline pre-platform and tracks it weekly post-platform. Latency growth in any persona triggers a workflow audit. |
| F4 | Confidence visual grammar (three-tier) is non-optional on every RAO. Low-confidence recommendations are visibly distinct and Kondukt spike 4 (BYOK 429 handling) covers the silent-degradation case. |

---

## Open questions

- **Modification UX as much as acceptance UX** — when Stefan modifies an RAO (changes the replacement unit, adjusts the effective_at time), what does that capture for the trust dataset? Modification rate is a signal in its own right; the surface needs to preserve the original recommendation alongside the modification.
- **Crisis vs. triage state transition** — the 70/30 triage default and the full-width crisis state are both Stefan's. What's the transition gesture? Modal? Layout shift? PRD has crisis modal pre-seeding the chat; UX scenario phase needs to lock the gesture.
- **Shift handover summary** — Stefan's outgoing shift hands over to Stefan's incoming peer. The handover surface inherits all open RAOs, all in-flight decisions, all unresolved outcomes. Defining "handover-complete" is a Phase 3 scenario.
