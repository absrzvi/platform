# 11: Roland's Live Board — Multi-Subsystem Diagnostics on the Moving Map

**Project:** ÖBB Landside Platform (harness model)
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** Sprint 3 cluster (Live Board + Kanban — Roland's two glanceable views)
**Persona:** Roland — Fleet Manager (Flottenmanagement / Team Telematik / Teil-PL DOSTO Neu + Enzo)

---

## Transaction (Q1)

**What this scenario covers:**
Roland opens the harness and scans his fleet on the Live Board — a glanceable moving map of every train (in-service + depot) overlaid with per-subsystem health. He needs the morning situational pulse: where are the trains, what subsystems are flagged, are there cross-subsystem patterns Kondukt has surfaced overnight. He pivots the view twice — first by subsystem ("show me all CCTV faults across the fleet"), then by train ("show me everything wrong with 4736-101") — without leaving the page. Kondukt has annotated a cluster he'd otherwise miss: three CCTV faults on the same hardware revision. He clicks the train at the top of that cluster to investigate; Live Board redirects him to that train's detail card in Kanban (the depth surface — scenario 12).

---

## Business Goal (Q2)

**Goal:** Multi-subsystem visibility no competitor produces — the harness's structural moat per Trigger Map Objective 1 (per-view acceptance) + Objective 6 (fundable event). Live Board is the surface where Roland *sees* the harness's USP every morning.
**Objective:** Trigger Map Objective 1 — Live Board annotation acceptance ≥70% per Roland's pre-pilot confirmation.

---

## User & Situation (Q3)

**Persona:** Roland — Fleet Manager (real-world: Roland Ruisz, DI(FH) — Flottenmanagement / System- und Betriebstechnik / Team Telematik / Teil-Projektleitung Telematik for Cityjet DOSTO Neu + Enzo). One role, two programme hats.

**Situation:** 06:30, Wien operations centre, start of shift. Roland has just authenticated via IdP (Keycloak in MVP). BYOK key already configured. The harness opens on the Live Board (default landing). Fleet today: 47 in-service trains spread across the rail network, 12 in depot across Wien-Hbf, Wien-Westbf, Floridsdorf, Simmering, Linz. Overnight there were three reservation-screen blackouts on DOSTO Neu units (he hasn't read the Daily Brief yet — that's scenario 13).

---

## Driving Forces (Q4)

**Hope:** See the fleet's overnight state in a single scan. No tab-switching. No re-assembling situational awareness from Power BI + Nomad Monitoring + vendor portals. The Live Board *is* the situational awareness.

**Worry:** Information density too high — the Live Board becomes wallpaper, Roland glazes over it; OR too low — important faults are buried under cosmetic surface noise. The view must surface decay-without-attention, not "everything alphabetically." Cross-subsystem patterns must be visible without him having to ask.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Desktop browser (Roland's ÖBB workstation). Single ≥1920×1080 monitor primary; dual-monitor common but not required. Modern Chromium-based browser (Chrome / Edge — Firefox is best-effort).

**Entry:** Roland authenticates → harness loads → Live Board is the default landing surface. Last-used pivot state restored per-user (FR3 — pivot state persisted). Today he opens on the subsystem-pivot view because that's where he left it last shift; the harness shows the morning state in that view.

---

## Best Outcome (Q7)

**User Success:** Roland reads his fleet's morning state in <60 seconds without expanding any train. He notices the Kondukt annotation surfacing a recurring CCTV pattern (3 of 8 active CCTV faults share hardware revision A7 — provenance attached). He clicks the top train in that cluster. Live Board redirects to that train's detail card in Kanban (scenario 12 entry). His next 5 minutes are spent on the right thing — the recurring failure — not on the surface noise.

**Business Success:** Live Board engagement ≥80% of shift starts (proxy: Roland's first action on session open is on Live Board, not on Kondukt or Kanban directly). Pivot-axis usage measured per-user — both subsystem-pivot and train-pivot used at least once per shift. Live-Board-to-Kanban click-through rate (the depth-on-demand redirect) is non-zero — operators are using Live Board as a triage surface, not a wallpaper.

---

## Shortest Path (Q8)

1. **Authenticate** — IdP redirect, return to harness. (Pre-condition; not a Live Board step.)
2. **Land on Live Board (default surface, pivot state restored).** Map shows all trains (in-service + depot), severity-banded per highest open fault. Per-subsystem health columns on the right edge (in subsystem-pivot view) OR overlaid on the train marker (in train-pivot view). `data_freshness_ts` indicator green; no stale-data warning.
3. **Glance.** Roland reads the map in <60 seconds. He sees: 47 in-service trains, 12 depot trains, severity bands distributed roughly evenly across CCTV (8 active), PIS (3 active), intercoms (2 active), AFZ (1 active), Wi-Fi (4 active — known dead-zone overlay accounts for 3, only 1 is hardware), TCMS (0). No CRITICAL severity.
4. **Notice Kondukt annotation.** Top-right of the CCTV subsystem column: a small Validation-Receipt-backed banner — *"Recurring pattern detected: 3 of 8 active CCTV faults share hardware revision A7. Confidence: HIGH. 14 days, provenance shown."* Roland clicks the banner to expand: shows the three affected trains (4736-101, 4736-103, 4736-108), the IntentPackets cited, the validator result.
5. **Pivot to train-pivot view.** Roland clicks "By train" toggle. View reorients: trains become primary; subsystems become column-overlays on each train marker. Severity bands now indicate per-train aggregate. The three CCTV-pattern trains are visible adjacent to each other on the map (they happen to be on similar routes).
6. **Click 4736-101.** Live Board does NOT expand inline. It redirects to that train's detail card in Kanban (scenario 12 entry). Scenario 11 ends; scenario 12 begins.

---

## Trigger Map Connections

**Persona:** Roland — Fleet Manager (Primary)

**Driving Forces Addressed:**
- ✅ **Want W3:** Live Board — multi-subsystem visibility no competitor produces
- ✅ **Want W4:** Decay-without-attention focus + depth on demand (glanceability + click-through redirect)
- ✅ **Want W6:** Cross-subsystem correlation surfaced by Kondukt (the recurring-CCTV annotation)
- ❌ **Fear F3:** The harness is another tab not THE tab — Live Board *is* the morning surface; no Power BI / Nomad Monitoring / vendor portal needed
- ❌ **Fear F1:** LLM-hallucinated provenance — Kondukt annotation carries Validation Receipt; provenance citation reachable inline

**Business Goal:** Trigger Map Objective 1 (per-view acceptance) + Objective 6 (fundable event — harness positioning USP)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 11.1 | `11.1-default-landing/` | Default Live Board landing — moving map with severity bands; per-subsystem health legible | Roland glances; reads fleet state |
| 11.2 | `11.2-subsystem-pivot/` | Subsystem-pivot view: trains arranged by subsystem health columns; CCTV / PIS / intercoms / AFZ / audio / Wi-Fi / switches / TCMS / Stadler diagnostic — each as a column or row aggregate | Roland sees per-subsystem severity at a glance |
| 11.3 | `11.3-train-pivot/` | Train-pivot view: trains as primary markers on map; subsystem state overlaid per-train marker | Roland sees per-train view; recognises the CCTV cluster |
| 11.4 | `11.4-kondukt-annotation/` | Kondukt cross-subsystem / recurring-failure annotation banner with provenance + Validation Receipt; expandable inline | Roland reads, agrees with the pattern detection |
| 11.5 | `11.5-click-through-to-kanban/` | Live Board → Kanban redirect on train selection; auto-expands that train's detail card | Roland lands on 4736-101's Kanban detail card (scenario 12 entry) |

---

## Open Questions (for Phase 4 page specs)

- **Map projection + tile source** — vector tile from a self-hosted OSM-based service is the working assumption. ÖBB rail network overlay is a per-deployment add-on. Phase 4 spec confirms tile provider + ÖBB rail-line GIS data source.
- **Subsystem-pivot view layout** — columns? Rows? Heatmap grid (subsystems × trains)? Trade-off: column layout shows subsystem aggregate well but train-by-train deltas are hard to read; heatmap-grid shows both but at higher visual density. Phase 4 spec confirms.
- **Severity band aggregation rule when train has multiple open faults across subsystems** — highest severity wins? Weighted by neglect-risk (Kondukt-computed)? Phase 4 spec — likely "highest severity wins" for first-cut.
- **Kondukt annotation density limit** — how many recurring-pattern annotations are visible on Live Board at once? If Kondukt detects 7 patterns simultaneously, the surface gets noisy. Working assumption: max 3 surfaced inline; "Show more (N)" affordance for the rest. Phase 4 spec confirms.
- **Cellular dead-zone overlay (FR7)** — the surface that disambiguates stale-telemetry-due-to-geography from stale-due-to-hardware. Where does it render? Phase 4 spec.
- **Default landing view persistence** — restored per-user (FR3). What about first-time users (no prior pivot state)? Default to subsystem-pivot or train-pivot? Working assumption: subsystem-pivot for first-run (showcases the multi-subsystem USP).
- **Train marker behaviour for trains in depot** — different visual treatment than in-service trains (depot trains are stationary; severity-band still applies but motion vector doesn't)? Phase 4 spec.
- **Click-through redirect target** — locked: redirects to Kanban detail card auto-expanded for that train. Open: does Live Board preserve its current view state when Roland navigates back from Kanban? Working assumption: yes, browser back button restores Live Board state including pivot. Phase 4 spec confirms.
- **Mobile/tablet view** — Live Board is desktop-primary. Tablet view is best-effort (read-only fleet glance); mobile is out of MVP. Phase 4 spec confirms.

---

_Scenario authored 2026-05-25 — harness-model MVP, Sprint 3 cluster._
