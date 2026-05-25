# 06: Lukas's Vendor Warranty Dispute

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** 4
**Persona:** Lukas the Teil-Projektleitung Telematik (Primary — commercial wedge)

---

## Transaction (Q1)

**What this scenario covers:**
Three Cityjet DOSTO Neu units have shown reservation-screen blackouts during Probebetrieb. Stadler's first response blames ÖBB's landside reservation database. Lukas opens the Telematics Control Board, uses Zone 1 to identify the affected units, drills into Zone 2 train topology to inspect the reservation-screen hardware, reads the Kondukt finding in Zone 3, approves a Recommended Action Object to package the warranty evidence, downloads the structured warranty claim document, and attaches it to the Stadler ticket himself.

---

## Business Goal (Q2)

**Goal:** Vendor warranty leverage — one Probebetrieb dispute resolved in a single session
**Objective:** Trigger Map Objective 4

---

## User & Situation (Q3)

**Persona:** Lukas the Teil-Projektleitung Telematik (Primary — commercial wedge)
**Situation:** Lukas's office at ÖBB programme management, mid-afternoon. He has Stadler Service Request #SR-2026-0517 open in another tab — Stadler's response blames ÖBB. The Probebetrieb cycle for the affected units is active (this scenario assumes overlap; if no overlap, this is a post-pilot story per the pre-pilot blocker).

---

## Driving Forces (Q4)

**Hope:** Build a single-session, timestamp-correlated evidence packet that survives the vendor lawyer's read and gets Stadler to authorise the bus controller swap under warranty.

**Worry:** That an ambiguity in Kondukt's timestamps or causal chain becomes the vendor's pick-apart point.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Desktop (Lukas's workstation, dual-screen — the Telematics Control Board is desktop-heavy)
**Entry:** Lukas opens the Telematics Control Board. Three-zone layout is live. He filters Zone 1 to the affected fleet.

---

## Best Outcome (Q7)

**User Success:**
Lukas resolves the dispute in a single session — under 90 minutes (vs the previous 2-day baseline). The Kondukt-generated warranty claim PDF includes explicit timezone stamping, the causal chain, every claim backed by an underlying packet/log reference, and a confidence indicator per claim. Stadler authorises the bus controller swap within two days.

**Business Success:**
One Probebetrieb dispute resolved during the pilot — Trigger Map Objective 4 satisfied. Commercial wedge evidenced for fleet GTM. Lukas's credibility intact.

---

## Shortest Path (Q8)

1. **Zone 1 — fleet compatibility matrix (filtered)** — Lukas opens the Control Board. Zone 1 (top-left, dominant) shows the Cityjet DOSTO Neu fleet compatibility matrix with geospatial telemetry overlay. He filters to "active Probebetrieb units with reservation-screen alerts in last 7 days." Three units highlighted with red severity bands.
2. **Zone 1 selection → Zone 2 binding** — Lukas clicks one of the three units (Unit DN-014). Zone 2 (top-right) opens the SVG train topology for DN-014 — coaches arranged left-to-right with hardware nodes as labelled points. The reservation-screen nodes (4 per coach) are visible.
3. **Zone 2 — node inspection** — Lukas clicks one of the reservation-screen nodes (Car 3, Position 2L). Inline metadata drawer opens: MAC address, firmware version, last 24h packet trace from VLAN 6 (reservations), and — critically — the corresponding landside reservation-service P99 latencies during the same 24h window, time-aligned.
4. **Zone 3 — Kondukt Proactive Recommendation Card** — Lukas glances at Zone 3 (bottom). Kondukt has already surfaced a Proactive Recommendation Card: "Reservation service response P99 was 187ms during all three failure windows. Bus controller memory utilisation crossed 94% in the same window on all three units. Pattern consistent with onboard memory leak, not landside latency. Confidence: HIGH (3 of 3 units exhibit the same pattern; landside service P99 within SLA bounds across all failure windows)." Each claim has an inline reference to the underlying data.
5. **Approve RAO — package warranty evidence** — Lukas reads the card. Cross-correlates with Zone 2's drawer. He approves: "Package evidence dump for Stadler ticket #SR-2026-0517." A confirmation surface explicitly states "Vendor-delivery RAOs require operator confirmation — this generates a warranty claim packet that will be exported to a PDF you can attach yourself. Confirm?" Lukas confirms.
6. **Warranty packet preview** — Kondukt assembles the packet. Preview surface shows: cover page (Stadler ticket reference, affected units, claim summary), timestamp-correlated logs (UTC explicit, alongside Europe/Vienna for human readability), causal chain (with each link traceable to the underlying packet), and a confidence column per claim. Lukas reads through. Looks defensible.
7. **Download PDF + manual attachment** — Lukas clicks "Download PDF". The structured warranty claim document downloads to his desktop. He uploads it to the Stadler service portal himself (the platform does not file warranty claims automatically — PRD constraint).
8. **Control Board — return** — Lukas returns to the Control Board. The RAO is logged in his execution history. Status: "Evidence packet generated, manually delivered to Stadler. Awaiting vendor response." ✓

---

## Trigger Map Connections

**Persona:** Lukas the Teil-Projektleitung Telematik (Primary — commercial wedge)

**Driving Forces Addressed:**
- ✅ **Want W1:** Single-session warranty dispute resolution
- ✅ **Want W2:** Cross-domain visibility (onboard memory + landside P99 latency in one view)
- ❌ **Fear F1:** Vendor picks apart timestamps — explicit UTC + Europe/Vienna timezone stamping; causal chain explicit
- ❌ **Fear F2:** Kondukt wrong, Lukas wears it — confidence indicator per claim; vendor-delivery RAO requires explicit confirmation; underlying-data references inline
- ❌ **Fear F4:** Loss of vendor portal access — platform layers on top, not replaces. (Phase 4 spec for "view in vendor portal" deep links)

**Business Goal:** Trigger Map Objective 4 (vendor warranty leverage as commercial wedge)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 06.1 | `06.1-zone-1-fleet-matrix/` | Filtered compatibility matrix + geospatial overlay; alerts highlighted | Lukas selects a unit |
| 06.2 | `06.2-zone-1-to-2-binding/` | Zone 1 selection opens Zone 2 topology | Lukas clicks node |
| 06.3 | `06.3-zone-2-topology-drawer/` | SVG topology + inline metadata drawer with time-aligned cross-domain data | Lukas reads Zone 3 |
| 06.4 | `06.4-zone-3-proactive-rao/` | Pre-surfaced Proactive Recommendation Card with confidence + references | Lukas approves the RAO |
| 06.5 | `06.5-vendor-delivery-confirmation/` | Explicit operator confirmation for vendor-delivery RAOs | Lukas confirms → packet preview |
| 06.6 | `06.6-warranty-packet-preview/` | Cover page + timestamp-correlated logs + causal chain + confidence | Lukas downloads PDF |
| 06.7 | `06.7-control-board-return/` | RAO logged in execution history; manual-delivery status | ✓ |

---

## Open Questions (for Phase 4)

- **Three-zone layout on dual-screen vs single-screen** — Lukas's persona detail open question: desktop-heavy, but does the layout degrade gracefully if he's on a single screen? Phase 4 spec.
- **Zone interaction binding direction** — Zone 1 → Zone 2 is in this scenario. Reverse direction (Zone 2 selection updates Zone 1 highlight)? Phase 4 spec.
- **Confidence threshold for vendor-delivery RAOs** — should low-confidence RAOs block the vendor-delivery action (or just warn)? Phase 4 spec — Lukas's persona detail F2.
- **Probebetrieb cycle overlap with pilot window** — pre-pilot blocker. If no overlap, this scenario is a post-pilot story. Confirm with Lukas's programme team pre pilot manday 1.
- **PDF structure standardisation** — does Stadler / other vendors have a preferred warranty claim document format? Best-effort schema or standards-bound? Phase 4 spec.
