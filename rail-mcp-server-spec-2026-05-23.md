# Rail MCP Server — Complete Specification
**Date:** 2026-05-23
**Status:** Frozen — ready for dev story creation
**Authors:** Winston (Architecture), Amelia (Engineering)

---

## Overview

The Rail MCP Server is a Nomad-built HTTP MCP endpoint exposing fleet data as typed tools for the Hermes Agent runtime (landside). Hermes calls these tools to answer Fleet Manager (Disponent) queries in natural language.

**Stack:** Python 3.11, FastAPI, Pydantic v2, SQLAlchemy async, PostgreSQL + pgvector
**Location in monorepo:** `rail-mcp-server/` (new subpackage, mirrors `cloud-backend/` patterns)
**MCP protocol:** HTTP endpoint, tools described via JSON Schema, responses are structured JSON

---

## Tool Set — 9 Tools (Frozen)

### Removed from original oebb-brain spec
- **`get_depot_schedule`** — REMOVED. Depot API schema unknown, BOOM descoped. Add when depot API is confirmed.
- **`schedule_report`** — REMOVED. Orchestration concern, not data retrieval. Recurring reports belong in a separate scheduling service with its own lifecycle, not an MCP tool Hermes calls ad-hoc.

---

### Tool 1 — `get_fleet_events`

**Purpose:** Primary event feed. Hermes uses for incident triage and timeline reconstruction.

**Parameters:**
```python
vehicle_id:  str | None        # ÖBB unit number; None = fleet-wide
journey_id:  str | None
severity:    Literal["INFO", "WARNING", "CRITICAL"] | None
event_type:  str | None
from_ts:     datetime | None   # UTC
to_ts:       datetime | None   # UTC; max span 72h enforced server-side
limit:       int = 50          # hard cap 200
```

**Response:**
```json
{
  "events": [
    {
      "event_id": "uuid",
      "vehicle_id": "str",
      "journey_id": "str | null",
      "timestamp": "ISO8601",
      "event_type": "str",
      "severity": "INFO|WARNING|CRITICAL",
      "source": "str",
      "payload": {}
    }
  ],
  "total": "int",
  "truncated": "bool"
}
```

**Data source:** `events` table, PostgreSQL landside
**Notes:** `truncated: true` signals Hermes to request a narrower time window. 72h max enforced server-side — no runaway queries.

---

### Tool 2 — `get_fleet_summary`

**Purpose:** Fleet health at a glance. Hermes uses for shift briefing or "how's the fleet right now."

**Parameters:**
```python
vehicle_ids: list[str] | None  # None = all active vehicles
```

**Response:**
```json
{
  "vehicles": [
    {
      "vehicle_id": "str",
      "active_journey_id": "str | null",
      "last_event_ts": "ISO8601 | null",
      "open_incident_count": "int",
      "severity_max": "INFO|WARNING|CRITICAL|null"
    }
  ],
  "generated_at": "ISO8601"
}
```

**Data source:** `events` table aggregate query

---

### Tool 3 — `get_hafas_timetable`

**Purpose:** Live schedule lookup. Hermes uses for delay questions and 48–72h forward planning window.

**Parameters:**
```python
journey_id:   str
date:         str              # YYYY-MM-DD pattern validated
horizon_h:    int = 48         # hours forward from now, max 72
```

**Response:**
```json
{
  "journey_id": "str",
  "route_name": "str",
  "stops": [
    {
      "station_name": "str",
      "scheduled_arrival": "ISO8601 | null",
      "scheduled_departure": "ISO8601 | null",
      "platform": "str | null",
      "delay_minutes": "int | null"
    }
  ],
  "data_freshness_ts": "ISO8601"
}
```

**Data source:** ÖBB HAFAS API (single call per invocation)
**Notes:** `data_freshness_ts` allows Hermes to caveat stale data. Unknown journey_id returns 200 with `stops: []`.

---

### Tool 4 — `search_fleet_events`

**Purpose:** Semantic retrieval. Hermes uses when query is conceptual — "any door sealing issues last week."

**Parameters:**
```python
query:       str               # natural language, min 3 chars, max 200
vehicle_id:  str | None        # optional pre-filter
from_ts:     datetime | None
to_ts:       datetime | None
limit:       int = 20          # max 50
```

**Response:**
```json
{
  "results": [
    {
      "event_id": "uuid",
      "vehicle_id": "str",
      "journey_id": "str | null",
      "timestamp": "ISO8601",
      "event_type": "str",
      "severity": "str",
      "source": "str",
      "payload": {}
    }
  ],
  "query": "str",
  "total_matches": "int"
}
```

**Data source:** pgvector embeddings over `IntentPacket.incident_summary` + pg_trgm fallback
**Notes:** Time range pre-filter prevents unbounded history returning irrelevant old matches.

---

### Tool 5 — `get_maintenance_cost_estimate` *(new)*

**Purpose:** Cost-impact one-liner for Disponent triage decisions.

**Parameters:**
```python
vehicle_id:        str
incident_summary:  str         # max 300 chars
severity:          Literal["INFO", "WARNING", "CRITICAL"]
```

**Response:**
```json
{
  "estimate": {
    "vehicle_id": "str",
    "severity": "str",
    "estimated_cost_eur": "float | null",
    "confidence": "float [0.0-1.0]",
    "cost_band": "low|medium|high|unknown",
    "rationale": "str"
  },
  "generated_at": "ISO8601"
}
```

**Data source:** `intent_packets` table (historical maintenance records) + static cost lookup table
**Notes:** Launch with `confidence: 0.0` and `cost_band: "unknown"` if cost seed data unavailable. Hermes must surface the confidence to Fleet Manager — never present low-confidence cost as authoritative.

**Open:** Who owns the cost lookup table seed data? Nomad ops must provide before this tool returns non-unknown values.

---

### Tool 6 — `get_shift_handover` *(new)*

**Purpose:** Shift continuity. Hermes uses when Disponent asks "what did last shift leave me."

**Parameters:**
```python
handover_id:  UUID | None      # specific handover; exactly one of the two required
vehicle_id:   str | None       # returns most recent handover for this vehicle
```

**Response:**
```json
{
  "handover": {
    "id": "uuid",
    "outgoing_user": "str",
    "incoming_user": "str",
    "handed_over_at": "ISO8601",
    "open_incidents": [
      {"event_id": "uuid", "vehicle_id": "str", "severity": "str", "title": "str"}
    ],
    "notes": "str | null"
  }
}
```

**Data source:** `shift_handovers` table
**Notes:** Unknown `handover_id` returns 404.

---

### Tool 7 — `get_ecm_audit_trail` *(new)*

**Purpose:** ECM compliance audit. Hermes uses when supervisor asks whether a maintenance action was signed off, or to surface unsigned actions.

**Parameters:**
```python
vehicle_id:  str
from_ts:     datetime | None
to_ts:       datetime | None
limit:       int = 20          # max 100
```

**Response:**
```json
{
  "vehicle_id": "str",
  "signoffs": [
    {
      "id": "uuid",
      "vehicle_id": "str",
      "technician_id": "str",
      "signed_off_at": "ISO8601",
      "vehicle_state": {},
      "directive_ref": "str | null",
      "supersedes_id": "uuid | null"
    }
  ],
  "total": "int"
}
```

**Data source:** `ecm_signoff_events` table (append-only, ordered by `signed_off_at DESC`)
**Notes:** Read-only. Hermes cannot write sign-offs. Corrections appear as new rows with `supersedes_id`.

---

### Tool 8 — `get_intent_packet` *(new)*

**Purpose:** Deep-dive on a specific incident. Hermes uses after `get_fleet_events` or `search_fleet_events` identifies a packet of interest.

**Parameters:**
```python
intent_packet_id: str          # UUID
```

**Response:**
```json
{
  "intent_packet_id": "uuid",
  "train_id": "str",
  "created_at": "ISO8601",
  "schema_version": "str",
  "incident_summary": "str",
  "agent_state": "str",
  "severity": "str",
  "confidence_score": "float",
  "location_ref": "str | null",
  "source_signals": ["str"],
  "local_action_taken": "str | null",
  "downstream_impact": "str | null",
  "request_landside_tasking": "bool",
  "expires_at": "ISO8601"
}
```

**Data source:** `intent_packets` table (IntentPacket v1.0.0 schema)
**Notes:** Closes the gap between lean event list rows and full IntentPacket detail. Without this, Hermes has no drill-down path.

---

### Tool 9 — `get_forward_planning` *(new)*

**Purpose:** 48–72h planning horizon for Disponent deployment decisions.

**Parameters:**
```python
vehicle_ids:    list[str] | None  # None = all active
horizon_hours:  int = 48          # max 72
```

**Response:**
```json
{
  "predictions": [
    {
      "vehicle_id": "str",
      "predicted_at": "ISO8601",
      "issue_type": "str",
      "probability": "float [0.0-1.0]",
      "horizon_hours": "int",
      "recommended_action": "str"
    }
  ],
  "generated_at": "ISO8601",
  "horizon_hours": "int"
}
```

**Data source:** Predictive model over historical `intent_packets` patterns (v1 can be rule-based — trains with recurring fault types flagged at configurable probability threshold)
**Notes:** Predictions are not events — cannot be expressed as `get_fleet_events` with a future time window.

---

## Tool Summary Table

| # | Tool | Status | Source |
|---|---|---|---|
| 1 | `get_fleet_events` | Kept — 72h cap + truncated flag | PostgreSQL events |
| 2 | `get_fleet_summary` | Kept — unchanged | PostgreSQL aggregate |
| 3 | `get_hafas_timetable` | Kept — horizon_h + data_freshness | HAFAS API |
| 4 | `search_fleet_events` | Kept — time_range pre-filter added | pgvector |
| 5 | `get_maintenance_cost_estimate` | **New** | intent_packets + cost table |
| 6 | `get_shift_handover` | **New** | shift_handovers |
| 7 | `get_ecm_audit_trail` | **New** | ecm_signoff_events |
| 8 | `get_intent_packet` | **New** | intent_packets |
| 9 | `get_forward_planning` | **New** | Predictive model over intent_packets |
| — | `get_depot_schedule` | **Removed** — depot API unknown, BOOM descoped | — |
| — | `schedule_report` | **Removed** — orchestration, not data retrieval | — |

---

## File Structure

```
rail-mcp-server/
├── pyproject.toml
├── .python-version              # 3.11
├── ruff.toml
├── mypy.ini
├── src/
│   └── rail_mcp_server/
│       ├── __init__.py
│       ├── main.py              # FastAPI app + MCP manifest endpoint
│       ├── config.py            # Settings (pydantic-settings)
│       ├── db.py                # SQLAlchemy async engine
│       ├── models/
│       │   ├── __init__.py
│       │   ├── requests.py
│       │   └── responses.py
│       ├── tools/
│       │   ├── __init__.py      # exports TOOL_REGISTRY: dict[str, MCPTool]
│       │   ├── _base.py         # MCPTool ABC: name, description, input_schema, execute()
│       │   ├── fleet_events.py  # get_fleet_events + search_fleet_events
│       │   ├── fleet_summary.py
│       │   ├── timetable.py     # get_hafas_timetable
│       │   ├── maintenance.py   # get_maintenance_cost_estimate
│       │   ├── handover.py      # get_shift_handover
│       │   ├── ecm.py           # get_ecm_audit_trail
│       │   ├── intent_packet.py # get_intent_packet
│       │   └── forward_planning.py
│       └── routers/
│           ├── __init__.py
│           ├── manifest.py      # GET /mcp/manifest
│           └── invoke.py        # POST /mcp/tools/{tool_name}
└── tests/
    ├── conftest.py              # async test client, DB fixtures (real Postgres, no mocks)
    ├── unit/
    │   ├── test_models.py
    │   └── test_tool_registry.py
    └── integration/
        ├── test_manifest.py
        ├── test_fleet_events.py
        ├── test_fleet_summary.py
        ├── test_timetable.py
        ├── test_maintenance.py
        ├── test_handover.py
        ├── test_ecm.py
        ├── test_intent_packet.py
        └── test_forward_planning.py
```

**Routing in `invoke.py`:**
- Look up tool in `TOOL_REGISTRY`
- Unknown tool → 404
- Parse request body against tool's input model
- Validation failure → 422 with `detail`
- DB unavailable → 503, no stack trace in response body
- Call `execute()`, return validated response → 200

---

## Acceptance Criteria

```
AC-MCP-01  GET /health returns 200 {"status": "ok"} with no DB dependency.

AC-MCP-02  GET /mcp/manifest returns 200 with a JSON array of exactly 9 tool
           descriptors, each containing: name, description, input_schema,
           output_schema. Every input_schema is a valid JSON Schema
           (jsonschema.Draft7Validator.check_schema passes).

AC-MCP-03  POST /mcp/tools/{tool_name} with valid body returns 200 and a
           Pydantic-validated response matching the tool's output_schema.
           Applies to all 9 tools.

AC-MCP-04  POST /mcp/tools/{tool_name} with invalid body returns 422
           with "detail" in response body.

AC-MCP-05  POST /mcp/tools/nonexistent returns 404.

AC-MCP-06  get_fleet_events with severity="CRITICAL" returns only events where
           severity = 'CRITICAL'. Verified against seeded test data (min 3 rows
           across severities). response.truncated == False when results < limit.

AC-MCP-07  search_fleet_events with query="door fault" returns events matching
           the query. response.total_matches == len(response.results).

AC-MCP-08  get_fleet_summary with vehicle_ids=None returns a VehicleSummary
           entry for every vehicle_id present in the events table.

AC-MCP-09  get_hafas_timetable returns response with len(stops) >= 1 for a
           known journey_id. Unknown journey_id returns 200 with stops=[].

AC-MCP-10  get_maintenance_cost_estimate returns confidence in [0.0, 1.0] and
           cost_band as one of: low, medium, high, unknown.
           confidence=0.0 + cost_band="unknown" is valid at launch.

AC-MCP-11  get_shift_handover with unknown handover_id returns 404.
           With valid vehicle_id returns most recent shift_handovers row.

AC-MCP-12  get_ecm_audit_trail returns signoffs ordered by signed_off_at DESC.
           supersedes_id references must exist in the table (DB constraint,
           not tool responsibility).

AC-MCP-13  get_intent_packet with unknown id returns 404.
           With valid id returns full IntentPacket v1.0.0 fields.

AC-MCP-14  get_forward_planning with horizon_hours=72 returns only predictions
           where prediction.horizon_hours <= 72. Empty list is valid.

AC-MCP-15  No tool issues more than one outbound HTTP call per invocation.
           Verified by mocking httpx.AsyncClient and asserting call_count <= 1.

AC-MCP-16  DB unavailable (monkeypatched OperationalError): any tool returns
           503 with no stack trace in response body.

AC-MCP-17  mypy --strict passes with zero errors.

AC-MCP-18  ruff check passes with zero errors.

AC-MCP-19  All integration tests run against real PostgreSQL. No mock DB.

AC-MCP-20  Coverage gate: pytest-cov --cov-fail-under=80.
```

---

## Hermes Spike Tests — Pass/Fail

**Setup:** Hermes runtime at `http://localhost:8765`. Rail MCP Server running. Test DB seeded via `tests/conftest.py`.

---

### SPIKE-01 — Tool Discovery

```
GIVEN  rail-mcp-server running
WHEN   Hermes client calls GET /mcp/manifest
THEN   status_code == 200
AND    len(body) == 9
AND    {t["name"] for t in body} == {
           "get_fleet_events", "get_fleet_summary", "get_hafas_timetable",
           "search_fleet_events", "get_maintenance_cost_estimate",
           "get_shift_handover", "get_ecm_audit_trail",
           "get_intent_packet", "get_forward_planning"
       }
AND    every descriptor has: name, description, input_schema, output_schema
AND    jsonschema.Draft7Validator.check_schema(t["input_schema"]) raises no error

PASS   All assertions hold.
FAIL   Any tool missing, extra present, or schema invalid.
```

---

### SPIKE-02 — IntentPacket Round-Trip

```
GIVEN  DB seeded: 5 events for vehicle_id="4711" — 2 CRITICAL, 3 INFO
WHEN   POST /mcp/tools/get_fleet_events
       {"vehicle_id": "4711", "severity": "CRITICAL", "limit": 50}
THEN   status_code == 200
AND    len(body["events"]) == 2
AND    all(e["severity"] == "CRITICAL" for e in body["events"])
AND    all(e["vehicle_id"] == "4711" for e in body["events"])
AND    body["truncated"] == False
AND    GetFleetEventsResponse(**body) — no ValidationError

PASS   All assertions hold.
FAIL   Wrong count, severity leak, or Pydantic rejects response.
```

---

### SPIKE-03 — Schema Conformance (all 9 tools)

```
GIVEN  manifest loaded; each tool's output_schema extracted
WHEN   each tool invoked with minimal valid request:
         get_fleet_events:              {}
         get_fleet_summary:             {}
         get_hafas_timetable:           {"journey_id": "J001", "date": "2026-05-23"}
         search_fleet_events:           {"query": "door"}
         get_maintenance_cost_estimate: {"vehicle_id": "4711",
                                         "incident_summary": "door fault",
                                         "severity": "WARNING"}
         get_shift_handover:            {"vehicle_id": "4711"}
         get_ecm_audit_trail:           {"vehicle_id": "4711"}
         get_intent_packet:             {"intent_packet_id": "<seeded-uuid>"}
         get_forward_planning:          {}
THEN   for each tool:
         status_code == 200
         jsonschema.validate(response.json(), tool["output_schema"]) — no error

PASS   All 9 tools return schema-valid responses.
FAIL   Any tool non-200 or response fails schema validation.
```

---

### SPIKE-04 — Error Boundary

```
CASE A — unknown tool
  POST /mcp/tools/does_not_exist  {}
  → 404

CASE B — missing required field
  POST /mcp/tools/get_hafas_timetable  {"journey_id": "J001"}
  (date omitted)
  → 422, "detail" in body

CASE C — wrong type
  POST /mcp/tools/get_fleet_events  {"limit": "not_an_int"}
  → 422

CASE D — DB unavailable (monkeypatched OperationalError)
  POST /mcp/tools/get_fleet_events  {}
  → 503, no stack trace in body

PASS   All four cases return specified status code; Case D leaks no traceback.
FAIL   Any wrong status code, or Case D exposes internal detail.
```

---

## Confidence Score Thresholds (from UX decisions)

These thresholds apply to `confidence_score` on IntentPacket and to any confidence field returned by Rail MCP tools:

| Value | State | UI Treatment |
|---|---|---|
| ≥ 0.80 | Full confidence | Normal rendering |
| 0.60 – 0.79 | Low confidence | "Low confidence" label, amber border |
| < 0.60 | Unverified | "Unverified — verify manually", no recommended action shown |
| CRITICAL + < 0.60 | Always shown | Maximally muted, "Sensor data insufficient — verify manually" |
