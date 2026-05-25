# IntentPacket Schema v1.0.0

**Date:** 2026-05-23
**Status:** Draft — pending decision on `journey_id` (optional vs required)
**Author:** Amelia (Engineering), ratified in party mode session

---

## Overview

The IntentPacket is the **single versioned contract** between the onboard brain (Hailo-10H LLM state machine) and all downstream consumers:
- Conductor App (PWA)
- Depot Briefing Interface
- Landside Control Centre
- Fleet Manager AI Interface

**Do not modify fields without bumping `schema_version` and opening a migration story.**

---

## MQTT Transport

```
oebb/{train_id}/intent        ← IntentPacket  (QoS 1, retain=True)
oebb/{train_id}/intent/ack    ← consumer acknowledgement  (QoS 1)
```

`retain=True` ensures a newly connected consumer (e.g. Conductor App reconnecting after a tunnel) receives current brain state immediately on subscribe — no polling, no waiting for the next state transition.

---

## State Machine States

| State | Trigger | LLM fired? |
|---|---|---|
| `NOMINAL` | No overlapping fault signals | No |
| `DEGRADED_DWELL_RISK` | Door IM fault + vision obstruction overlap | Yes |
| `HARDWARE_PORT_DISCONNECT` | Camera DEAD + switch port LINK_DOWN | Yes |
| `BLIND_SPOT_PRM` | Camera DEAD covering PRM zone | Yes |
| `VESTIBULE_CONGESTION` | Occupancy >85% sustained >60s | Yes |

---

## TTL per State (expires_at guidance)

| State | Suggested TTL |
|---|---|
| NOMINAL | 300s |
| DEGRADED_DWELL_RISK | 120s |
| VESTIBULE_CONGESTION | 90s |
| BLIND_SPOT_PRM | 60s |
| HARDWARE_PORT_DISCONNECT | 600s |

Consumers **must** discard display state after `expires_at`. Do not re-display stale CRITICAL alerts after reconnect.

---

## Pydantic Model

Target file: `shared/schemas/intent_packet.py`

```python
"""
shared/schemas/intent_packet.py

IntentPacket v1.0.0 — canonical contract between oebb-brain and all consumers.
MQTT topic: oebb/{train_id}/intent  (QoS 1, retain=True)
Do NOT modify fields without bumping schema_version and opening a migration story.
"""
from __future__ import annotations

import uuid
from datetime import datetime, timezone
from enum import Enum
from typing import Literal

from pydantic import BaseModel, Field, field_validator


class AgentState(str, Enum):
    NOMINAL = "NOMINAL"
    DEGRADED_DWELL_RISK = "DEGRADED_DWELL_RISK"
    HARDWARE_PORT_DISCONNECT = "HARDWARE_PORT_DISCONNECT"
    BLIND_SPOT_PRM = "BLIND_SPOT_PRM"
    VESTIBULE_CONGESTION = "VESTIBULE_CONGESTION"


class SignalSource(str, Enum):
    CCTV = "CCTV"
    APC = "APC"                   # door counts VLAN 8
    SNMP = "SNMP"                 # Stadler VLAN 7
    PIS = "PIS"                   # PIS/FIS VLAN 3
    RESERVATIONS = "RESERVATIONS" # VLAN 6
    ENERGY = "ENERGY"             # VLAN 12
    ZFR = "ZFR"                   # train control VLAN 2


class IntentPacket(BaseModel):
    schema_version: Literal["1.0.0"] = "1.0.0"
    packet_id: uuid.UUID = Field(default_factory=uuid.uuid4)
    train_id: str = Field(..., min_length=1, max_length=64)
    journey_id: str | None = Field(
        default=None,
        description="Aligns with shared/events/types.py journey_id scheme "
                    "({vehicle_id}_{trip_number}_{YYYYMMDD}). "
                    "None if journey not yet initialised at brain startup. "
                    "OPEN: confirm optional vs required before freezing schema.",
    )
    timestamp: datetime = Field(
        default_factory=lambda: datetime.now(timezone.utc),
        description="UTC, always timezone-aware",
    )
    expires_at: datetime = Field(
        ...,
        description="UTC. Consumers MUST discard display state after this time.",
    )
    agent_state: AgentState
    severity: Literal["INFO", "WARNING", "CRITICAL"]
    confidence_score: float = Field(
        ...,
        ge=0.0,
        le=1.0,
        description="LLM inference confidence [0.0, 1.0]. "
                    "Consumers should render low-confidence CRITICAL alerts differently "
                    "(e.g. amber border, 'Low confidence' label).",
    )
    incident_summary: str = Field(
        ...,
        max_length=300,
        description="Plain language. Max 300 chars. Written for crew, not engineers.",
    )
    location_ref: str | None = Field(
        default=None,
        description="Free-form spatial ref, e.g. 'Car 3 Door 2L', 'Vestibule B'. "
                    "Required when agent_state is VESTIBULE_CONGESTION or BLIND_SPOT_PRM.",
    )
    local_action_taken: str | None = Field(
        default=None,
        description="What the onboard system did automatically, if anything.",
    )
    downstream_impact: str | None = Field(
        default=None,
        description="Expected operational impact if not resolved.",
    )
    request_landside_tasking: bool = Field(
        default=False,
        description="True = landside fleet manager should be notified and act. "
                    "Surfaced as priority item in Fleet Manager AI Interface.",
    )
    source_signals: list[SignalSource] = Field(
        ...,
        min_length=1,
        description="Signal types that triggered this state transition. At least one required.",
    )

    @field_validator("timestamp", "expires_at", mode="before")
    @classmethod
    def require_utc(cls, v: datetime) -> datetime:
        if isinstance(v, datetime) and v.tzinfo is None:
            raise ValueError("datetime fields must be timezone-aware (UTC)")
        return v

    @field_validator("expires_at", mode="after")
    @classmethod
    def expires_after_timestamp(cls, v: datetime, info) -> datetime:
        ts = info.data.get("timestamp")
        if ts and v <= ts:
            raise ValueError("expires_at must be after timestamp")
        return v

    model_config = {
        "json_schema_extra": {
            "mqtt_topic_pattern": "oebb/{train_id}/intent",
            "mqtt_qos": 1,
            "mqtt_retain": True,
        }
    }
```

---

## What Was Added vs Original oebb-brain Spec

| Field | Reason |
|---|---|
| `packet_id` | Consumers need stable UUID for deduplication (SSE reconnect, Control Centre worker restart) |
| `confidence_score` | CRITICAL at 0.62 vs 0.98 look identical without this; consumers render differently |
| `expires_at` | Stale alerts must self-expire; consumers discard display state after this timestamp |
| `location_ref` | Spatial events (VESTIBULE_CONGESTION, BLIND_SPOT_PRM) need car/door reference |
| `journey_id` | Aligns with existing shared/events/types.py envelope; optional pending decision |
| `AgentState` enum | `agent_state` was bare `str` — compiler is now the spec |
| `SignalSource` enum | `source_signals` was `list[str]` — typed enum replaces Redis key names |

---

## Open Questions

1. **`journey_id` optional vs required?** Brain may initialise before journey data arrives (e.g. at depot). If optional, consumers must handle `None`. If required, brain must wait for journey initialisation before emitting any packet. Decision needed before schema is merged to `shared/`.

2. **`schema_version` migration strategy** — currently `Literal["1.0.0"]`. When v1.1.0 ships, add `Literal["1.0.0", "1.1.0"]` and a discriminated union. A migration story is required — do not bump inline without a story.

3. **`confidence_score` threshold for UI treatment** — what value triggers "low confidence" rendering in the Conductor App and Control Centre? Suggested: `< 0.75`. Needs UX decision.

---

## Consumer Responsibilities

| Consumer | Must do |
|---|---|
| Conductor App | Discard display state after `expires_at`; render low-confidence CRITICAL differently; show `location_ref` prominently for spatial events |
| Depot Briefing | Group packets by `agent_state` and `severity`; surface `request_landside_tasking=True` packets at top |
| Landside Control Centre | Deduplicate on `packet_id`; show `source_signals` as cited evidence; surface `confidence_score` |
| Fleet Manager AI | Store in PostgreSQL + pgvector (embed `incident_summary`); use `packet_id` as idempotency key on insert |
