# Party Mode — Critical Assessment
**Date:** 2026-05-23  
**Topic:** Unified platform scope, use case prioritisation, and Phase 2 risks  
**Participants:** Winston (Architect), Mary (Business Analyst), Freya (UX Designer), Amelia (Engineer)

---

## Context

Abbas is merging two repos into one unified Phase 2 product:
- **oebb-agent** — Phase 1 built: Hailo-8 vision pipeline, Control Centre Dashboard (React/FastAPI/PostgreSQL/SQLite), 5 epics complete, CI green
- **oebb-brain** — Designed but not built: Hailo-10H LLM layer (1–2B model, structured pipeline), Hermes BYOK harness, Fleet Manager UI, Rail MCP Server, intent packet JSON schema as onboard↔landside contract

**Hardware confirmed:** Two M.2 slots — Hailo-8 stays for vision, Hailo-10H added for LLM.

**Phase 2 scope (4 interfaces):**
1. Conductor App — greenfield Android tablet, offline-first, active alerts + "Is it safe to continue?" LLM safety query + departure clearance + coach occupancy overview
2. Depot Briefing Interface — mobile-first, BOOM work order integration, ECM Manager + electrical/mechanical/IT technicians, pre-arrival fault brief
3. Landside Control Centre — extend existing React dashboard with intent packet display + LLM diagnosis + platform health monitoring
4. Fleet Manager AI Interface — Hermes BYOK, 70/30 layout, natural language queries over fleet data

**Key USP:** LLM inference runs fully offline onboard.  
**Priority stack:** Onboard crew first → Depot maintenance second → Landside reporting third.

---

## 🏗️ Winston — Architectural Assessment

### The offline-first claim is underspecified
"Offline-first" means one of three very different things: (a) works fully without any network, (b) works with degraded functionality without network, or (c) syncs when reconnected. Each has a completely different data architecture. Before any UX story begins for the Conductor App, you need a written answer to: what data is pre-loaded, what is generated locally, and what explicitly fails when the train has no uplink?

The LLM safety query is the dangerous one. If the Hailo-10H answers from local inference, fine. But if it needs fleet-wide context (sister train incidents, track conditions) that lives landside, you have a latency/availability dependency baked into a safety-critical query. That dependency must be explicit, not assumed away.

### The intent packet schema is doing too much structural work with too little definition
"JSON schema" is not a contract — it's a syntax. What you need before UX stories begin: who writes intent packets, at what cadence, what happens to a malformed one, what's the versioning strategy when Hailo-10H firmware changes the output format, and what does a consumer do when a packet is stale or missing? Define the semantics, not just the structure.

### Two M.2 slots, two AI accelerators — no arbitration model
If the vision pipeline detects an anomaly and the LLM simultaneously processes a conductor's "is it safe?" query using a slightly stale occupancy frame — which output takes precedence? This is an architectural gap. You need a defined arbitration layer, even if simple: vision events are authoritative, LLM provides diagnosis and recommendation only, never command. Write that down before UX starts designing alert dismissal flows, because dismissal UX encodes assumptions about authority.

### BOOM integration has no fallback story
What happens when BOOM is unreachable? What subset of fault data is available from the onboard event store without BOOM, and is that sufficient for a minimum viable briefing? Design to that baseline first.

### Hermes BYOK has an implicit trust boundary problem
BYOK means customer-supplied API keys hitting an LLM that has access to fleet telemetry. Threat model: prompt injection from the LLM response path back into fleet data display, and credential exposure if the key is ever logged or cached. Before UX stories spec the query interface: where does the API key live at runtime, does it ever touch the inference code path, and what sanitisation sits between LLM output and the UI render layer?

### What's missing before UX design stories begin
1. Offline capability matrix per interface (feature × connectivity state × behaviour)
2. Intent packet semantics document (lifecycle, versioning, staleness policy)
3. Safety authority chain document (who can raise, dismiss, which system is authoritative)
4. BOOM fallback design (minimum viable briefing without external integration)
5. Hermes trust boundary spec (key storage, output sanitisation, audit trail)

---

## 📊 Mary — Business Assessment

### On prioritisation: mostly right, one dangerous gap
ECM Manager as primary persona — correct. The commercial logic is airtight: Roland Ruiz and Martin Lerch are the rollout gatekeepers, and they care about maintenance cost reduction. The fault detection ≤60s / false positive <5% criteria are measurable.

But: **who owns the Conductor App decision?** "Is it safe to continue?" is safety-adjacent on a moving train — it touches ETCS, TSI, and ÖBB's own operational safety framework. If no named person has said "yes, an LLM can display a safety recommendation to a conductor," this feature is commercially unvalidated regardless of how well it's built. This is the single biggest risk in the scope.

### On scope realism: it's too wide
Four interfaces in Phase 2 — two greenfield, one extension, one platform play. The Hailo-10H LLM layer isn't built. The intent packet schema is untested. The Rail MCP Server is untested. Every interface depends on that stack being production-stable first.

**Recommendation: sequence it.**
- Depot Briefing Interface first — ECM Manager use case, commercial bet, smallest LLM surface area
- Conductor App second — only after safety sign-off question is answered
- Fleet Manager third — landside, loses nothing by waiting

### Underestimated stakeholder risks

1. **BOOM integration** — Who owns the BOOM API? Is there a data-sharing agreement? ÖBB IT governance for third-party integrations can add 3–6 months. This is procurement and legal, not technical.
2. **The >50% dispatcher adoption metric is vanity** — Adoption of what? If the Control Centre just passively displays intent packets, a glance counts. Define the adoption interaction precisely before UX begins.
3. **Data residency for Fleet Manager BYOK** — Fleet telemetry entering a customer-supplied LLM endpoint. Has ÖBB's data protection officer reviewed what's permissible? NIS2 and ÖBB's critical infrastructure classification probably applies.

### What's missing before UX begins — non-negotiable
1. A signed safety statement for the Conductor App LLM feature (even one sentence from an ÖBB safety authority)
2. Intent packet schema ratified and version-locked
3. BOOM API discovery — scope as "BOOM-ready" not "BOOM-integrated" until confirmed API access
4. Defined offline degradation behaviour for the Conductor App

---

## 🎨 Freya — UX Assessment

### The conductor's mental model is not "query an AI" — it's "is this my call or not?"
At 120km/h, a conductor isn't composing a query. They're looking for permission or a stop signal. The UX must resolve to one of two states: **proceed** or **hold**. Anything in between — nuanced LLM prose, confidence percentages, sensor summaries — is a liability.

What happens when the answer is ambiguous? What's the UI for "I'm not sure"? That scenario must be designed before any story is written — it's the hardest one and most likely during sensor degradation.

### You're designing for clear-sky conditions
Four flows for the Conductor App assume good data. What's the conductor's experience when Hailo-8 has a degraded feed, or VLAN 5 drops mid-journey, or inference arrives 40 seconds late? "Offline-first" means works without cloud — it also needs to mean **works when onboard sensors are partially blind.** That's a different design problem and it's not resolved.

### The Depot Briefing Interface is trying to serve three roles with one layout
Each technician has a different primary concern. An IT tech doesn't care about brake pad wear. An electrical tech doesn't need firmware versions first. "Role-differentiated" cannot mean a tab strip or filter toggle — it needs to mean three genuinely different information hierarchies. Also: who confirms a generated work order? Who owns errors in it? That accountability flow is invisible.

### The Fleet Manager's 70/30 layout is a solution looking for a problem
70/30 is a layout prescription. Before that's locked: what is the Fleet Manager *actually doing* when they open this interface? Triaging in real time? Reviewing trends? Responding to escalations? Each is a different primary task with a different information priority. Defining the layout before defining the task model is backwards.

The progressive disclosure pattern (Widget → Discovery → Conversational) assumes the Fleet Manager has time to explore. In crisis mode, progressive disclosure is friction. The mode needs to be demand-sensitive, not just progressive.

### The LLM being "not a chatbot" is correct — but it's not communicated in the UX framing
Users will try to interact with it like a chatbot because that's what AI interfaces have trained them to expect. You need explicit UX decisions about how to signal that this system answers *one* structured question, not a conversation. What does the interface do when a user tries to ask a follow-up? Silence is not a design.

### What needs to be in UX scenarios before stories
1. The degraded-data scenario for the conductor — sensor failure mid-journey, partial inference, late response
2. The ambiguous safety answer — LLM returns non-binary outcome, how does the UI handle it without panic or false confidence?
3. Role authentication for Depot Interface — how does the system know which technician is viewing, and what changes?
4. Fleet Manager crisis mode vs. routine mode — same entry point or different?
5. The "follow-up question" anti-pattern — what happens when a conductor or fleet manager tries to push back on an LLM answer?

---

## 💻 Amelia — Engineering Assessment

### BLOCKER-1: IntentPacket contract undefined
No schema file exists in either repo. Every downstream consumer builds against air.

Must exist before any story goes to dev:
- `shared/events/types.py` — `IntentPacket` as a versioned StrEnum event type (append-only rule applies)
- `shared/schemas/intent_packet.py` — Pydantic v2 model, field-level validation, version field
- MQTT topic structure (`oebb/train/{unit_id}/intent`) documented and frozen

Failure mode: Conductor App caches IntentPacket shape X. Brain emits shape Y. Cache deserialization fails silently on reconnect. Offline-first guarantee broken.

### BLOCKER-2: 4 Hermes spikes unrun

| Spike | Story failure if skipped |
|---|---|
| MCP startup error handling | Rail MCP Server story ships, Fleet Manager gets unhandled 500 on tool init |
| SQLite concurrency under 10 sessions | Offline store corrupts under multi-tab or background sync |
| Skills async model | Hermes blocks event loop; SSE to Control Centre drops |
| BYOK 429 handling | Fleet Manager hangs indefinitely on rate limit; no retry budget |

Run all 4 before any Phase 2 story creation.

### BLOCKER-3: BOOM API is unknown
Auth mechanism, query contract, SLA, availability — all zero-information. Cannot dev-story until confirmed. If BOOM is landside-only, the depot offline story doesn't exist. If it requires VPN, container placement changes.

### BLOCKER-4: Conductor App transport decision unresolved
"React Native or PWA TBD" determines:
- `event-store/` WebSocket handler binary framing requirements
- Offline store: `expo-sqlite` vs IndexedDB vs custom sync engine
- CI pipeline: Expo EAS vs Vite build
- Test stack: Detox vs Playwright

Every story written before this decision is wrong by definition. Spike: 1 day, decide, file ADR in `docs/adr/`, freeze.

### RISK-1: Redis Streams consumer — no idempotency spec
If brain/ crashes mid-correlation and replays from last ack, does the landside PostgreSQL writer deduplicate? On what key? Must specify before brain/ container story: consumer group name, ack strategy, dedup key, dead letter policy.

### RISK-2: pgvector migration is not zero-downtime
`CREATE EXTENSION vector` requires superuser or `pg_extension_owner` role. On the current Alembic flow this blocks concurrent reads during migration. Confirm PostgreSQL user has extension creation rights. Write migration with `IF NOT EXISTS` guard. Document rollback: `DROP EXTENSION vector CASCADE` drops all vector columns — irreversible if data exists.

### RISK-3: 17-event StrEnum is append-only — IntentPacket adds at minimum 1
Gate in CI: grep the StrEnum count in `shared/events/types.py`, fail if a story branch adds events without a `shared/` change.

### RISK-4: Rail MCP Server — no tool contract
Six tools described, no signatures, no error codes. Fleet Manager stories will be written against invented contracts and break on integration.

Minimum required before Fleet Manager story — freeze these signatures:
```
get_train_status(unit_id: str) -> TrainStatusResponse
query_events(unit_id: str, event_type: EventType, limit: int) -> list[Event]
get_intent_summary(unit_id: str) -> IntentPacket | None
search_events(query: str, k: int) -> list[Event]  # pgvector
get_fleet_overview() -> FleetSummary
submit_work_order(unit_id: str, payload: WorkOrderRequest) -> WorkOrderResponse
```

### Gate criteria before any Phase 2 story creation

| # | Gate | Type |
|---|---|---|
| 1 | `IntentPacket` Pydantic schema merged to `shared/` | Hard blocker |
| 2 | All 4 Hermes spikes run and findings documented | Hard blocker |
| 3 | BOOM API contract obtained (or story descoped to "BOOM unavailable" state) | Hard blocker |
| 4 | Conductor App transport ADR filed and merged to `docs/adr/` | Hard blocker |
| 5 | Rail MCP Server tool signatures frozen | Hard blocker |
| 6 | Redis Streams idempotency spec written | Sprint-entry criteria |
| 7 | pgvector migration rights confirmed with ops | Sprint-entry criteria |

---

## Key Convergence Points

All four agents independently flagged the same three issues:

1. **IntentPacket schema** — foundational blocker for every interface. Must be defined and frozen before any story begins.
2. **Conductor App safety sign-off** — both a commercial gate (Mary) and a UX design gate (Freya). No named ÖBB authority has approved LLM safety recommendations to crew.
3. **Conductor App transport decision** — React Native vs PWA must be resolved and filed as an ADR before any story is written.

**Sequencing recommendation (Mary):** Depot Briefing first → Conductor App second → Fleet Manager third. Don't design all four simultaneously against an unvalidated LLM stack.
