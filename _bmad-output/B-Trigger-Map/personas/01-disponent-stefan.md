# Persona Detail: Roland the Fleet Manager

> **File rename pending.** This file is still named `01-disponent-stefan.md` on disk to preserve cross-references in scenario files and the design log; rename to `01-fleet-manager-roland.md` is deferred to a deliberate filename sweep. The persona's name is now **Roland** (real first name, no longer pseudonymous as "Stefan"). The "Disponent" framing in the filename is also outdated — Roland is Flottenmanagement / Team Telematik / Teil-Projektleitung Telematik, not formally a Disponent (dispatcher). See [_progress/00-design-log.md](../../_progress/00-design-log.md) for the rename + harness-reframe entries.

**Priority:** Primary — daily user, per-view acceptance owner, harness USP carrier
**Real-world equivalent:** Roland Ruisz, DI(FH) — Flottenmanagement / System- und Betriebstechnik / Team Telematik, with Teil-Projektleitung Telematik for Cityjet DOSTO Neu + Cityjet Enzo. **One role, two programme hats** — not two roles. (The previous separate "Lukas the Teil-Projektleitung Telematik" persona is retired 2026-05-25; see `04-telematik-lukas.archived.md`.)
**Surface:** The harness (Live Board / Kanban / Kondukt — three MVP views) + the shadow audit trail he writes into via recommendation acceptances.

---

## Profile

Roland runs a 12-hour rotating shift across operations + new-fleet commissioning — day/night/weekend rotation, weekend availability required. He's solution-focused, stress-resilient, accountable for fleet availability + the technical telematics layer health across two new-fleet programmes. He's the Fleet Manager: technical fleet management for the IP-networked subsystems across the DOSTO Neu and Enzo fleets, plus sub-project leadership for the telematics workstream of each programme's commissioning.

His day combines:
- **Live fleet-health monitoring** — the operational view: what's broken now, what's degrading, what's in depot, what's coming back
- **New-fleet commissioning telematics work** — the programme view: which units are passing Probebetrieb, which are showing drift, which need vendor coordination
- **Multi-subsystem diagnostic responsibility** — CCTV, PIS, intercoms, AFZ passenger counters, audio, Wi-Fi, switches, TCMS, Stadler diagnostic plane, Firewall RCU — every IP-addressed subsystem is in his scope
- **Cross-vendor coordination** — Stadler, Siemens, Knorr-Bremse, Nomad — when something breaks, he's the landside point of contact

Today he works monitor-heavy with multiple systems open — Nomad Monitoring (IoB only — no other subsystem visibility), vendor diagnostic portals (one per vendor), Power BI for historical, ServiceNow for tickets, MS Project + Excel for programme tracking, paper for ECM correlation. Switching between them mid-incident is where decisions get slow and traceability gets thin. Recurring failures across vendor silos stay invisible. MTTR is not surfaced anywhere. Daily situational awareness is reassembled from scratch each shift.

**Roland works alone, but his screens are read by Team Telematik peers** — every view must read cold by a peer, not just by Roland.

## Behavioural anchors

- **Cross-fleet view** — he watches DOSTO Neu + Enzo as primary, with full fleet visibility (both in-service and depot)
- **Subsystem-pivot AND train-pivot** — switches between "all my CCTV faults across the fleet" and "everything wrong with 4736-101"
- **Live + retrospective combined** — live moving map for situational awareness; historical recurring-failure intelligence for engineering decisions
- **Vendor-system as source of truth** — MTTR is sourced from Stadler's diagnostic system; the harness reads it, does not recompute it
- **Operator authority is the safety architecture** — Roland decides; the harness recommends. Advisory-only is not a limitation — it's the posture that lets a one-person Einzelunternehmen supply this on a NIS2-scope system.

---

## Positive Drivers (Wants)

### W1. Daily Brief skill produces a validated overnight situation report
Roland opens the harness in the morning, runs the *Daily Brief* skill in Kondukt. Kondukt has already prepared (or prepares on invocation) a structured brief: fleet overnight state, changes since last shift, recommended focus, top incidents needing triage — every claim citing its IntentPacket source, every claim with a confidence pill. Failed validations would have suppressed; the brief he sees has passed the constitution-as-contract layer. First-kill decision: which of last night's anomalies actually changes today's plan, made in 4 minutes.

### W2. Neglect-Risk Kanban surfaces decay-without-attention
Open faults sorted by neglect-risk score (Kondukt-computed, provenance-attached), not by chronological queue order. Lanes (New / Investigating / Recommendation Pending / Cleared) carry context across the shift. The detail card, when expanded, gives him primary per-component detail (subsystem, VLAN, IP, MAC, switch port, firmware, time-since-fault) on first click; secondary detail (MTTR, recurring-failure history, 24h packet trace, vendor portal deep-link) on second click. Progressive disclosure: heavy diagnostic depth surfaces only when he asks for it.

### W3. Live Board — multi-subsystem visibility no competitor produces
Fleet-wide moving map showing all in-service trains + all depot trains. Per-subsystem health legible at a glance — CCTV, PIS, intercoms, AFZ, audio, Wi-Fi, switches, TCMS, Stadler diagnostic — all in one context, all correlated. **Group-pivot both ways**: by subsystem ("all CCTV faults") OR by train ("everything wrong with 4736-101"). The Nomad Monitoring System sees only IoB. Stadler sees only their plane. The harness sees everything. Live Board stays glanceable (operator-friendly names, no VLAN IDs); click a train → redirects to Kanban depth surface.

### W4. Decay-without-attention focus + depth on demand
The harness is built around the discipline of *not* drowning Roland in queue noise. Kanban lane-sort by neglect-risk forces attention to the right card. Live Board glanceability with click-through redirect prevents two-depth surfaces. Progressive disclosure on Kanban detail keeps the expanded card readable until Roland needs the deep technical view. Every UI choice optimises for *which card matters now*, not *how many cards exist*.

### W5. Kondukt skill registry + free-form chat — the agentic-OS surface
Skills codify the routines: *Daily Brief*, *Recurring-failure pattern report*, *Vendor meeting prep doc*, *Fleet health summary*. Each skill produces a validated artifact in the canvas. Free-form chat is the catch-all for anything not codified. Kondukt has access to Rail MCP tools (telemetry, depot state, MTTR from Stadler, recurring-failure detection, semantic search, neglect-risk inputs, xlsx/docx artifact builders) + **allowlisted internet egress** (Stadler / Siemens / Knorr-Bremse vendor docs, CENELEC, ERA, EU regulatory texts; no open web). Roland can ask for a vendor-facing report and download it as `.docx` to take to a Friday meeting.

### W6. Cross-subsystem correlation surfaced by Kondukt
Intercom drops + CCTV drops + Wi-Fi AP drops on the same train within the same window = network-path issue, not three independent subsystem failures. Recurring-failure detection at fleet scale: *"3 of 8 active CCTV faults share hardware revision A7, mean-time-since-firmware-update is 47 days. Confidence: HIGH."* Kondukt annotates Live Board with patterns it detects; provenance attached; suppressed if validation fails. This is the structural moat — cloud-only competitors don't have the multi-subsystem feed to correlate against.

### W7. xlsx + docx artifact production from operational evidence
Roland needs to produce vendor-facing reports for Stadler escalations, Lastenheft procurement evidence, internal fleet health summaries for his boss. Kondukt's *Vendor meeting prep* skill assembles the report in the canvas — cover page, pattern summary, fleet table, vendor firmware timeline overlay, plain-language conclusion with provenance citations. Roland reviews, asks for revisions in free-form chat, clicks Download. The artifact lands on his desktop.

### W8. MTTR computed by the harness from landside-observable timestamps
Component alarm raised in Stadler diagnostic system → arrives landside via brain MQTT forward = **t_open**. Same component's "alarm cleared" update arrives landside via brain MQTT forward = **t_close**. **MTTR = t_close − t_open**, computed by the harness. The harness does NOT read Stadler's internal MTTR field. Reasons: (a) the operator's experienced MTTR is the landside-observable window, which is what matters; (b) brain→landside forwarding latency is included on both ends, which is the honest number; (c) the harness owns this metric and can audit it without trusting Stadler's internal accounting. Surfaced on the Kanban detail card (secondary disclosure layer). Time-since-fault is on the primary layer; MTTR is on the secondary.

---

## Negative Drivers (Fears)

### F1. LLM-hallucinated provenance reaches the operator
Without the constitution-as-contract enforcement layer, Roland would be reading Kondukt-produced briefs and recommendations that might cite IntentPackets that don't exist, MCP queries that didn't happen, vendor portal pages that weren't fetched. The validator (schema + provenance check + advisory-language linter) is the architectural answer — failed validations suppress at emission time. *Silence is safe.* Roland sees a suppression banner with reason; he never sees a hallucinated recommendation.

### F2. Reconstructability gap
If an incident is escalated and Roland can't reconstruct who recommended what, when, what action was taken, and what the outcome was — his liability widens. The Recommendation Provenance Replay archive (Envelope + Provenance Bundle + Validation Receipt) for every Kondukt emission, plus the recommendation→action→outcome chain in the shadow audit trail, plus the 1:1 reconciliation between the two — this is the architectural answer. Six months later, an auditor with the Replay CLI can verify any emission deterministically.

### F3. The harness is another tab, not THE tab
If Roland still has Nomad Monitoring + Power BI + Stadler portal + ServiceNow + MS Project + Excel open alongside the harness, adoption fails — the harness has just added a sixth tab without removing the friction of the other five. The four MVP views must be sufficient for Roland's daily work; cross-subsystem visibility must replace Nomad's IoB-only view; Chat+Canvas + allowlisted internet egress must replace ad-hoc vendor portal browsing for research; the Kanban detail card must replace per-vendor diagnostic portal lookups for the common cases. The harness is THE tab or it's nothing.

### F4. LOW-confidence rubber-stamping
The LOW-confidence DISABLE rule (party mode 2026-05-25 lock) is the design-system answer — Modify is the only execution path on LOW emissions, cursor pre-placed in the justification field to force operator attention. But the rate of LOW emissions itself is the calibration signal: LOW-rate sustained over 15% across any 72h window = validator is wrong (or the customer's LLM is wrong, or the data layer is missing context). Trip-wire monitored.

### F5. BYOK token exhaustion mid-shift
The customer's Anthropic / OpenAI / etc. API key has a daily token budget — the harness has no control over it. If Roland exhausts his key mid-shift, Kondukt skill invocations + free-form chat are suppressed. Live Board + Kanban remain functional (telemetry-driven, no LLM). Degraded-mode banner explicit about provider status. Token usage transparent throughout the day so Roland sees depletion coming.

### F6. Train-to-landside-DB mapping divergence
A recommendation cites "fault on train 4736-101" but the underlying IntentPacket actually came from train 4736-103 — silent misrouting destroys the trust premise. Weekly automated reconciliation between IntentPacket `train_id` and MQTT topic + component MAC/IP registry validation. Mapping divergence is a pilot-kill trigger: halt harness data writes for the affected train pending root-cause within 24h. Without this, the entire Provenance Replay archive is suspect.

### F7. Phantom recommendation in early pilot mandays
Even with the constitution-as-contract enforcement, a single bad recommendation that reached Roland and was acted on could poison acceptance for the next 30 mandays. The tabletop exercise (mandays 20–25) explicitly tests this: Kondukt recommends X, Roland acts, Stadler diagnostic state diverges from what Kondukt cited — who's accountable, what does the Replay CLI confirm, where does the audit trail live? Four-way ambiguity (LLM provider / harness / operator / IntentPacket layer).

### F8. A "co-pilot" that turns out to be a slower way to do what he already does in the vendor portals
If Kondukt skill invocations add cycle time without removing other-system cycle time, adoption collapses inside the pilot and per-view acceptance never crosses 70%. Skills must be *additive* — the artifact Roland was going to produce anyway, but now with provenance and downloadable in 60 seconds instead of 30 minutes.

### F9. Constitution-as-theatre — validator passes at emission, breaks on replay six months later
If the validator is non-deterministic, or the Provenance Bundle isn't stored verbatim, or the content-addressed blob store loses bytes, then the Replay CLI gives a different verdict than the original Validation Receipt. This is the existential bug for Renate's sign-off + the outside-auditor tabletop verdict. Deterministic validators (no LLM in the validation loop), content-addressed blob storage, periodic Replay-against-archive smoke tests through pilot.

---

## Must-address design implications

| Driver | Design implication |
|---|---|
| W1 | *Daily Brief* skill ships in Kondukt v1 skill registry; emission validated end-to-end (schema + provenance + advisory-language linter) before reaching canvas; suppression on validation failure logged with reason. |
| W2 | Kanban lane-sort by neglect-risk score, not queue order. Train detail card uses progressive disclosure: primary = VLAN/IP/MAC/firmware/switch port/time-since-fault; secondary = MTTR/recurring-failure history/24h packet trace/vendor portal deep-link. Operator-friendly names on card summary; technical depth on second click. |
| W3 | Live Board renders per-subsystem health for every monitored subsystem by operator-friendly name only — no VLAN IDs surface on Live Board. Group-pivot both ways (by subsystem / by train) with state persisted per-user. Click-through redirect to Kanban detail card. |
| W4 | Decay-without-attention is the architectural principle. Lane-sort, glanceability, progressive disclosure all serve this. Surface that surfaces *more* (chronological queue, expanded detail by default) violates the principle. |
| W5 | Kondukt skill registry versioned in-product, inspectable (constitution + schemas + validator rules visible to operator). MVP starting set: Daily Brief / Recurring-failure pattern / Vendor meeting prep / Fleet health summary. New skills via versioned ADR. Free-form chat is the catch-all; every chat turn validated. xlsx/docx artifact builders accessible to skills + chat. |
| W6 | Cross-subsystem correlation is a Kondukt-emitted annotation on Live Board, with provenance to the IntentPackets that informed it. Validator rejects unsupported correlations. Recurring-failure detection is a named skill in the registry. |
| W7 | xlsx + docx export from canvas; operator-triggered download; chat transcripts retained as Provenance Bundle content. Downloaded artifacts are working artifacts (not in the audit trail). |
| W8 | MTTR computed by the harness from landside-observable timestamps: `t_open` = arrival timestamp of the Stadler alarm-raised IntentPacket, `t_close` = arrival timestamp of the Stadler alarm-cleared IntentPacket. `MTTR = t_close − t_open`. Stadler's own internal MTTR field is not read. Surfaced on Kanban detail card secondary disclosure. |
| F1 | Constitution-as-contract enforcement layer: per-skill output schemas + provenance validators + advisory-language linter on every emission. Deterministic post-processing, no LLM in the validation loop. Failed validations suppress; suppression banner shows reason. No regeneration loop in MVP. |
| F2 | Every recommendation produces a triplet at emission (Envelope + Provenance Bundle + Validation Receipt) stored in `recommendation` Postgres table (append-only) + content-addressed blob store. Recommendation→action→outcome chain in shadow audit trail. 1:1 reconciliation. Replay CLI runs deterministically on auditor laptop. |
| F3 | The four MVP views must be sufficient for Roland's full work. Cross-subsystem visibility on Live Board replaces Nomad's IoB-only view. Vendor portal deep-link on Kanban detail replaces per-vendor portal lookups for the common case. Kondukt allowlisted-egress fetches replace ad-hoc browser tabs for vendor doc research. Test: at pilot manday 30, is the harness the *only* landside surface Roland has open? |
| F4 | LOW-confidence DISABLE rule (locked 2026-05-25): on LOW-confidence recommendations, the Accept interaction is non-interactive (`aria-disabled`); Modify is the only execution path with cursor pre-placed in the justification field; `modification_distance` captured on commit. LOW-rate >15% sustained over 72h = trip-wire, calibration review. (Note: deliberate-confirmation gestures like the old "Mark-executed hold" or hold-to-record are descoped from MVP 2026-05-25; the DISABLE rule applies to the simple-click Accept on the recommendation card.) |
| F5 | BYOK key vault encrypted at rest, per-user, never logged. Token usage transparent in every Kondukt session (cumulative + per-skill). Degraded-mode banner (FR20) on provider unreachability — Live Board + Kanban remain functional. |
| F6 | Weekly automated train-to-landside-DB mapping verifier (FR41). Mapping divergence is a pilot-kill trigger. `mapping_health` metric on harness ops dashboard. |
| F7 | Tabletop exercise at manday 20–25 explicitly tests phantom-recommendation scenario with Replay CLI demonstrated live on outside auditor's laptop. |
| F8 | Per-skill cycle-time measurement during pilot. If a skill adds time-on-task without removing time-on-other-systems, skill is retired or revised. |
| F9 | Deterministic validators (functional purity, no LLM, no network calls inside the validation loop). Content-addressed blob storage (SHA-256 keys). Periodic Replay-against-archive smoke tests through pilot. Replay CLI runs on auditor laptop with no harness infrastructure. |

---

## Open questions

- **Per-view acceptance threshold per hat** — ≥70% per-view is the working assumption. Roland's three programme contexts (operations / DOSTO Neu commissioning / Enzo commissioning) may have different acceptance profiles. Per-hat tracking may need finer granularity than per-view; pre-pilot baseline conversation with Roland is the moment to confirm.
- **Modification UX vs acceptance UX** — when Roland modifies a Kondukt recommendation (changes the proposed action, adjusts the citation set), what does the modification_distance signal mean in practice? Modification rate is a trust signal in its own right; the surface needs to preserve the original alongside the modification with the diff visible.
- **Live Board → Kanban click-through gesture** — single click? Double click? Open in new tab? Keyboard shortcut? The redirect is locked; the gesture isn't.
- **Kondukt skill registry evolution during pilot** — additions via ADR are locked. But what's the cadence? Weekly ADR review with Roland? Monthly? Or on-demand when Roland names a skill gap?
- **Shift handover summary** — Roland's outgoing shift hands over to Roland's incoming peer (or his own next shift). The handover surface inherits all open Kanban cards by lane, all in-flight Kondukt skill invocations, all suppressed-emission events, all unverified-outcome states. Defining "handover-complete" is a Phase 3 scenario open question (task #3).
- **Colleague-readability sample** — Roland's screens are read by Team Telematik peers. Sample what they read first: Live Board scan at standup? Kanban triage at shift change? Pilot manday baseline conversation with Roland's team.

---

## What's merged from the retired Lukas persona

These drivers were originally split off as "Lukas the Teil-Projektleitung Telematik." On 2026-05-25 they were merged here:

| Lukas driver (retired) | Where it landed |
|---|---|
| W1 (single-session vendor warranty win) | Vision tier — warranty recovery workflow descoped. Detection capability lives in Roland W6 (cross-subsystem correlation) + Kondukt *Vendor meeting prep* skill. |
| W2 (cross-domain visibility — onboard + landside in one view) | Roland W3 (Live Board multi-subsystem visibility) + W6 (cross-subsystem correlation). The structural moat survived; the framing reframed. |
| W3 (OTA rollouts with canary + drift) | Vision tier — OTA management out of MVP. Drift moved to ECM module. |
| W4 (Lastenheft procurement export) | Vision tier — unchanged. Kondukt skill registry can grow into this post-pilot. |
| F1 (vendor lawyer picks apart timestamps) | Roland F9 (Constitution-as-theatre / Replay non-determinism). Same structural concern, different artifact context. |
| F2 (Kondukt wrong, Lukas wears it) | Roland F1 (LLM-hallucinated provenance). Same fear, answered by the constitution-as-contract enforcement layer. |
| F3 (no Probebetrieb overlap during pilot) | De-scoped — warranty itself out of MVP. |
| F4 (lose access to vendor diagnostic portals) | Roland W5 (Kondukt allowlisted internet egress to vendor portals) + Kanban detail card vendor portal deep-link. |

---

_Revised 2026-05-25 — harness-model reframe + Lukas merger + Stefan→Roland rename._
