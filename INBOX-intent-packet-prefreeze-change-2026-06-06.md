# ⚠️ Coordination: IntentPacket schema changed pre-freeze (from oebb-brain)

**From:** oebb-brain workspace
**Date:** 2026-06-06 (rev. 2 — **expanded**: now carries the FULL D1–D14 + evidentiary-integrity changeset, not just the early evidence-layer additions)
**Action needed:** Review + ack before the schema is re-frozen. Your local
`intent-packet-schema-v1.0.0.md` is **out of date** — sync it from `oebb-brain/`.

---

## Why this changed

In the last ÖBB TS meeting, ÖBB TS disclosed they are **not capturing ECM 2019/779
maintenance-delivery evidence** today (BOOM CMMS unused — adoption failure). We are
repositioning the brain from "advisory alerting" to the **automatic evidence layer**
for the ECM F4 record — without claiming release authority (decision stays human).
See `oebb-brain/ADR-002-ecm-system-of-record.md`.

Since rev. 1 of this notice, the brain architecture completed (steps 1–8, decisions D1–D14;
`oebb-brain/_bmad-output/planning-artifacts/architecture.md`), the **Tier-2 `control-centre/`
internal architecture landed (ADR-006)**, and a **23-finding adversarial review** of it surfaced
cross-tier evidentiary-integrity carriage the contract was missing. All of that is now folded into
the schema as ONE coordinated, **additive-only, one-way-door** amendment.

Schema was amended **in place, not via a migration story** — nothing is built or deployed on either
side, so there is no contract to migrate. But the IntentPacket is **co-owned**, so you need the full
diff and must ack before re-freeze. **Version stays 1.0.0** (no shipped predecessor).

---

## The diff — Part 1: already notified in rev. 1 (recap)

1. **`record_type: Literal["EVIDENCE"]`** on IntentPacket — machine-checkable guardrail; the brain
   emits EVIDENCE, never a clearance (no `CLEARANCE` value exists). Additive, defaulted.
2. **`AckRecord` model** — permanent, immutable human acknowledgement on `oebb/{train_id}/intent/ack`.
   *(Significantly expanded since rev. 1 — see Part 2.)*
3. **`journey_id` → OPTIONAL** on the packet. **Consumers MUST handle `journey_id == None`.**
4. **Two-lifecycle clarification** — `expires_at` governs ephemeral *display* only; the permanent
   evidence is the persisted packet + AckRecord. An expired display ≠ expired evidence.
5. **Fault states** `CABLE_FAULT` / `SWITCH_FAILURE` / `REDUNDANCY_LOST` + **`affected_devices`** (fan-out).

## The diff — Part 2: NEW since rev. 1 (the D1–D14 + integrity changeset — **needs your ack**)

> Each item tagged with the brain-side decision / finding id. All additive; the `source_signals`
> reshape is the one required-field shape change (hence pre-freeze).

**A. Provenance trust class — `source_signals` reshaped (D2 / COH-3B).**
`source_signals: list[SignalSource]` → `list[{source, trust_class}]`, new `TrustClass` enum
{`AUTHORITATIVE`, `INFERRED`}. AUTHORITATIVE anchors a fault; INFERRED corroborates.
*Consumer impact:* **shape change** — read `entry.source` + `entry.trust_class`; your validator must
reject a packet missing `trust_class` on any signal. Segregate INFERRED-only counts in any
"how often / pattern" aggregation (don't silently fold vision false-positives into authoritative counts).

**B. Ordering / epoch keys (D3c / D11 / COH-1B / COH-10B).** New on **both** models:
`record_id` (server-assigned edge-writer rowid at insert; **store-local**, monotonic+gapless per store,
NOT a client UUID), `attestation_epoch`, `device_id`.
*Consumer impact:* **`record_id` is the ordering/causality authority — wall-clock is descriptive only.**
The true dedup/citation key is **`(train_id, attestation_epoch, record_id)`**, NEVER `record_id` alone
(rowids reset on a board swap).

**C. Cross-tier integrity carriage (EI-1 / EI-2 / D14).** New on both models:
`content_hash`, `merkle_batch_id`, `merkle_inclusion` (sibling-hash path).
*Consumer impact:* a record with no `merkle_inclusion` yet is **UNANCHORED** (tamper-detectable, NOT
yet tamper-evident) — the declared residual window. **Do not render UNANCHORED rows as tamper-evident.**
Full tamper-evidence requires the inclusion proof + the landside-held signed Merkle root / anti-rollback
counter / epoch device pubkey.

**D. `time_trust` enum {`ntp_synced`, `board_clock_unsynced`} (D11 / COH-5A).** Both models.
*Consumer impact:* **never present a `board_clock_unsynced` UTC as trusted time**, and never order on it.

**E. AckRecord — discriminated `ack_kind` union (D3b / COH-4A / COH-2B).**
`ack_kind` ∈ {`HUMAN_ACK`, `SYSTEM_SLA_BREACH`}. `HUMAN_ACK` requires `acknowledged_by` + `action_taken`
(permits `sop_referenced`, `resolution_mode`). `SYSTEM_SLA_BREACH` (brain-minted: SLA lapsed without ack —
*silence recorded as a signed fact*) requires a trusted-time + TPM-device-rooted system actor and
**forbids** `acknowledged_by` / `action_taken` / `sop_referenced` / `resolution_mode`.
*Consumer impact:* the `/intent/ack` topic now carries **two** record kinds — discriminate on `ack_kind`.
The IntentPacket⋈AckRecord join is **1:N** (provisional → countersign → SLA-breach).

**F. AckRecord authority + identity frozen enums.**
`ack_tier: Literal[2]` (D13 — `control-centre/` is the **sole** ack authority; **platform/ cannot mint an
ack**, even under a control-centre/ outage; domain reserves `3`). `identity_assurance_level` ∈
{`ATTRIBUTED_SSO` (default), `COUNTERSIGNED` (reserved)} (ADR-004 §4 / COH-6B).
*Consumer impact:* **platform/ (Tier 3) is downstream + read-only on acks** — please confirm you will not
emit AckRecords. The reserved `COUNTERSIGNED` value lets a future production upgrade be a value, not a
shape, change (no evidence migration).

**G. AckRecord provenance / SOP-resolution fields (D4 / D5 / COH-6A / COH-7A; F10 = no defaults).**
`resolution_mode` ∈ {`live`,`cached`,`unavailable`}, `corpus_artifact_version`, `field_provenance` map.
Invariants: non-null `sop_referenced` ⇒ `resolution_mode` present; + mode ≠ `unavailable` ⇒
`corpus_artifact_version` present.
*Consumer impact:* never treat a `sop_referenced` id as a stable handle without its `corpus_artifact_version`.

**H. GDPR crypto-shred envelope (D10 / COH-2A).** Named-person fields (`acknowledged_by`, `personnel`)
are now an `EncryptedEnvelope` {`subject_key_id`, `ciphertext`, `tombstoned`}, NOT bare plaintext columns.
Erasure = per-subject key destruction → tombstone; the per-row hash/Merkle leaf is over **ciphertext** so
the chain stays intact.
*Consumer impact:* render a `tombstoned` / key-destroyed envelope as a **tombstone**, never plaintext;
compute any hash over the ciphertext bytes.

**I. Lifted JSON-Schema wire invariants (COH-4B).** Datetime fields carry a tightened `pattern` (trailing
`Z`/offset); `expires_at > timestamp` and the `ack_kind` per-subtype required/forbidden rules are lifted to
JSON-Schema if/then/else (Python `field_validator`s alone do not reach your Ajv/Zod).
*Consumer impact:* your generated validator now enforces these — packets the brain rejects will be rejected
landside too (previously they'd pass silently).

**J. `ack_id` minting authorship by ORIGIN (ACK-1, conductorless correction).** The client that captures the
**deciding human's** confirm gesture mints `ack_id` (primary case = `control-centre/` dispatcher UI, since
KISS NV runs conductorless; on-train `onboard-portal/` when boarded). Still a stable idempotency key; landside
UPSERTs on it.

---

## What we need from you

- [ ] **Sync** your local `intent-packet-schema-v1.0.0.md` from `oebb-brain/` (full file — many fields changed).
- [ ] **Confirm** the `source_signals` shape change (`{source, trust_class}`) is acceptable to your consumers.
- [ ] **Confirm** platform/ (Tier 3) is **read-only on acks** and will not mint AckRecords (`ack_tier` has no
      value for you; D13).
- [ ] **Confirm** your consumers will: handle `journey_id == None`; discriminate `ack_kind`; render UNANCHORED
      as not-yet-evident; never trust a `board_clock_unsynced` clock; render crypto-shred tombstones; dedup on
      `(train_id, attestation_epoch, record_id)` not `record_id` alone.
- [ ] **Forwarding shape (separate, see `INBOX-control-centre-forwarding-shape-2026-06-06.md`):** the Tier-2→
      Tier-3 forward is provisionally **raw via CDC tail**; note the crypto-shred fields cannot be losslessly
      rehydrated to plaintext — confirm what you expect to receive for erased named-person fields.
- [ ] **Ack back** so oebb-brain can **re-freeze v1.0.0**. (Version stays 1.0.0 — no shipped predecessor.)

> Cross-row invariants (`ack_id` UNIQUE, 1:N cardinality) are **not** expressible in JSON Schema — they need
> DB constraints + a **bilateral contract test** on both sides. The committed JSON Schema is necessary but not
> sufficient for D3b conformance; the bilateral test is the other half.
