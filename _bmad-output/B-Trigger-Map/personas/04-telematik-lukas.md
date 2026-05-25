# Persona Detail: Lukas the Teil-Projektleitung Telematik

**Priority:** Primary (commercial wedge) — owns vendor warranty leverage; non-obvious-to-competitors persona
**Real-world equivalent:** Roland Ruisz wearing his second hat (same person as Disponent persona, distinct role context); programme-side counterpart is Lukas in the Cityjet DOSTO Neu / Cityjet Enzo programmes
**Surface:** Telematics Unified Control Panel (three-zone: macro situational awareness + micro train topology + Kondukt RAO console)

---

## Profile

Lukas manages a sub-project inside the larger Cityjet DOSTO Neu programme (and, on rollout, Cityjet Enzo). Before each new fleet enters service he runs Probebetrieb — a commissioning gauntlet where a single hardware telematics failure can freeze formal sign-off. Vendors push back; disputes get bureaucratic.

Today he spends two days per dispute pulling logs from vendor diagnostic portals, cross-referencing against ÖBB's PostgreSQL reservation service uptime in Power BI, exporting timestamps to Excel, writing a Change Request rebuttal email. Likely outcome: vendor holds the line, dispute escalates to procurement.

He is invisible to most "fleet management" pitches because he's not in operations — he's in delivery. His user-facing surface needs are entirely different from Stefan's: macro situational awareness over the whole new-fleet rollout, micro topology drill-down per vehicle, and a co-pilot that helps him build vendor-defensible evidence packets.

## Behavioural anchors

- **Programme-scale view** — he's not looking at one train, he's looking at a fleet's worth of vehicles at the same commissioning stage
- **Vendor-facing** — his outputs end up in adversarial settings (warranty claims, CR rebuttals) — they must survive a vendor lawyer's read
- **Cross-domain debugging** — most disputes are about *whose layer failed*; he needs to see onboard memory utilisation, landside service P99 latency, and network-path topology together
- **Project-managerial** — owns OTA rollouts, configuration drift, digital-twin validation, multi-vehicle procurement Lastenheft — all surfaces where competitor cloud dashboards have nothing to offer

---

## Positive Drivers (Wants)

### W1. Resolve a vendor dispute in a single session with timestamp-correlated evidence the platform packaged for him
Lukas opens the Telematics control board. Zone 1 (macro): fleet compatibility matrix shows three affected units flagged. Zone 2 (micro): SVG train topology with reservation screen node selected, inline drawer surfaces MAC + firmware + 24h packet trace. Zone 3 (Kondukt RAO): "Reservation service P99 was 187ms during all three failure windows. Bus controller memory utilisation crossed 94% in the same window on all three units. Pattern consistent with onboard memory leak, not landside latency." He approves a Recommended Action Object: "Package evidence dump for Stadler ticket #SR-2026-0517." PDF download. Attached. Two days saved per dispute.

### W2. Cross-domain visibility (onboard ↔ landside) that competitor dashboards can't produce
This is the platform's structural moat for Lukas's use case. Cloud-only competitors don't have the IntentPacket feed from the brain. They can show vendor telemetry or landside metrics — not both, correlated, in one view.

### W3. OTA rollouts with canary + drift detection, not Excel
Lukas runs phased OTA software rollouts. Currently he tracks them in MS Project + Excel + vendor portal status pages. The platform surfaces canary cohorts and configuration/firmware drift detection across the fleet at the commissioning stage.

### W4. Multi-vehicle Lastenheft procurement-spec export generated from operational evidence
Today's Lastenheft (procurement specification document) is written from scratch each time, lifting clauses from previous documents and hoping they still apply. Tomorrow's is generated from the cumulative operational evidence the platform has collected — clauses with traceable evidentiary backing.

---

## Negative Drivers (Fears)

### F1. A warranty-evidence package the vendor's lawyer can pick apart on a timestamp ambiguity
If the PDF Kondukt generates has timestamps in mixed zones (UTC vs. Europe/Vienna), or correlates two events that are causally unrelated, Stadler's response is "your evidence doesn't establish causation" and the dispute escalates. The structured warranty claim document has to be defensible.

### F2. A platform that surfaces "Kondukt thinks it's the bus controller" and is wrong — Lukas loses credibility, not the platform
Lukas's reputation is the currency. If he presents an evidence packet that turns out to be wrong, vendors discount his next packet. Confidence visual grammar matters here as much as for Stefan, and low-confidence RAOs in Lukas's context should *not* be packaged for vendor delivery without explicit override.

### F3. A Probebetrieb window that ends before the platform has been validated against it (Journey 6 unsupported during pilot)
The Probebetrieb cycle dependency is a pre-pilot gate (PRD Journey 6 NEEDS DECISION). If no Probebetrieb cycle overlaps the pilot window, Lukas can't evidence the warranty value during the pilot, and the commercial wedge is a post-pilot story.

### F4. Loss of access to vendor-raw diagnostic portals — Lukas trusts those, the platform is layered on top, not a replacement
If the platform's narrative becomes "you don't need the vendor portal any more," Lukas backs away. He needs both, and the platform must visibly *augment* — citing the underlying portal data, deep-linking where possible.

---

## Must-address design implications

| Driver | Design implication |
|---|---|
| W1 | Three-zone Telematics control panel: Zone 1 = macro (fleet compatibility matrix + geospatial telemetry overlay); Zone 2 = micro (SVG train topology + inline metadata drawers per hardware node); Zone 3 = Kondukt co-pilot console (chat + Proactive Recommendation Cards). Interaction binding: Zone 1 selection → Zone 2 topology; Zone 2 fault → Zone 3 RAO. |
| W2 | IntentPacket fields visible at the topology-node level include both onboard-sourced and landside-sourced data, time-correlated. |
| W3 | OTA rollout view: canary cohort + production cohort + drift count per firmware version + per-unit-version status. Surface drift before vendor flags it. |
| W4 | Lastenheft export action: takes a fleet selection + a time window + a clause taxonomy, produces a structured document with evidentiary citations. Format and consumer schema descoped from MVP (vision-tier capability) — design pattern should not preclude. |
| F1 | Kondukt RAO for warranty evidence: timestamps explicit timezone-stamped, causal-chain explicit, every claim backed by a packet/log reference, confidence on every claim. PDF export includes the evidentiary trail, not just the conclusion. |
| F2 | Confidence visual grammar (three-tier) on every RAO Lukas sees. Vendor-delivery RAOs (warranty packets, CR rebuttals) require explicit Lukas confirmation before export — Kondukt can propose, Lukas owns the send. |
| F3 | Pre-pilot gate: confirm Probebetrieb cycle overlap with Lukas's programme team. If no overlap, Telematics surfaces ship, warranty triage is a documented post-pilot story, FR41 capability is present-but-unevidenced-in-pilot. |
| F4 | Every topology-node drawer includes a "view in vendor portal" link where the vendor portal is reachable. Platform's value framing: cross-correlation, not replacement. |

---

## Open questions

- **Probebetrieb cycle overlap with pilot window** — pre-pilot blocker. Confirm with Lukas's programme team before pilot manday 1 is set.
- **SAP asset registry integration timing** — Journey 6 dependency. MVP uses local registry; SAP integration is post-MVP. Topology drawer metadata depth is bounded by this.
- **Cross-vehicle pattern detection** — Kondukt's "memory leak on three units" finding requires fleet-level pattern search, not single-vehicle. MCP tool design (pgvector + structured filter) — is this a single tool or multiple? Spike pre-Telematics stories.
- **Three-zone responsive behaviour** — desktop only, or does it degrade for tablet / dual-screen? Lukas's work context is desktop-heavy but spec needs to lock.
