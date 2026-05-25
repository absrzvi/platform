# 13: Kondukt — Daily Brief Skill on Session Open

**Project:** ÖBB Landside Platform (harness model)
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** Sprint 4 cluster (Kondukt skill registry — Roland's LLM-native surface)
**Persona:** Roland — Fleet Manager (Flottenmanagement / Team Telematik / Teil-PL DOSTO Neu + Enzo)

---

## Transaction (Q1)

**What this scenario covers:**
Roland opens the harness at 06:30 start of shift. The Live Board is the default landing (scenario 11.1) but he doesn't dwell there — his morning routine is to run the *Daily Brief* skill in Kondukt to get a structured overnight situation report. He clicks into the Kondukt view, invokes the *Daily Brief* skill from the skill registry. Kondukt runs the skill: pulls overnight IntentPackets via the Rail MCP server, runs recurring-failure detection across the last 24h, validates the LLM's draft output through the constitution-as-contract layer (per-skill schema + provenance + advisory-language linter), and emits the brief into the canvas. Each finding cites its IntentPacket source; each claim has a confidence pill (HIGH / MEDIUM / LOW). Roland reads, agrees with the prioritisation, makes his first-kill decision (which of last night's anomalies actually changes today's plan). The brief is logged as a recommendation triplet — Envelope + Provenance Bundle + Validation Receipt — in the Recommendation Provenance Replay archive.

Variant: on validation failure (e.g. citation-trace check fails because the LLM cited a non-existent IntentPacket), the emission is **suppressed**, not regenerated. Roland sees a suppression banner with the reason. He can re-invoke the skill manually.

---

## Business Goal (Q2)

**Goal:** Demonstrate the constitution-as-contract enforcement layer in the most-frequent operator interaction — the Daily Brief is the morning ritual. Every shift starts with this skill invocation. The brief is the first artifact Renate's outside auditor sees in the Replay CLI demo at manday 20–25. Trigger Map Objective 3 (Provenance Replay archive integrity ≥99% replay-verify clean) depends on this skill running cleanly thousands of times across the pilot.

**Objective:** Daily Brief skill invocation ≥80% of Roland's shift starts (FR63 per-view acceptance proxy); per-claim confidence distribution ≥80% HIGH+MEDIUM (FR63 criterion 4); recommendation triplet integrity 100% (FR63 criterion 3).

---

## User & Situation (Q3)

**Persona:** Roland — Fleet Manager.

**Situation:** 06:30, start of shift. Roland has authenticated (IdP). BYOK key is configured. The harness has just rendered Live Board as the default landing (scenario 11). His morning routine: glance Live Board for ~30 seconds, then click into Kondukt and run the *Daily Brief* skill. He prefers reading the skill output to scanning Live Board because the Daily Brief synthesises overnight changes into a prioritised structure.

---

## Driving Forces (Q4)

**Hope:** Get a validated, provenance-attached overnight situation report in <30 seconds from skill invocation. Each finding cites its source IntentPacket. Each claim has a confidence pill. The validator has already checked schema + provenance + advisory-only language — Roland reads with confidence that the harness owned correctness before letting Kondukt speak.

**Worry:** A hallucinated finding reaches the operator. A citation pointing to an IntentPacket that doesn't exist. A claim phrased in imperative-action language ("dispatch X") rather than advisory-only. The constitution-as-contract enforcement layer is the architectural answer — failed validations suppress; *silence is safe*. Roland never sees a hallucinated recommendation.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Desktop browser. Same as scenarios 11 + 12. The skill invocation is part of the Kondukt view.

**Entry:** Roland has authenticated, glanced Live Board, clicked into Kondukt. The view loads with chat input + skill registry shortcuts visible. Kondukt view is empty (no prior session content) since this is the start of his shift.

---

## Best Outcome (Q7)

**User Success:** Roland invokes the *Daily Brief* skill. Within 20 seconds, the brief renders in the canvas: 3 findings (overnight bus-controller pattern on DOSTO Neu, 2 CCTV firmware mismatches on Enzo, 1 PIS display unreachable for 14h). Each finding has its confidence pill (2 HIGH, 1 MEDIUM in this case). Each citation links to its source IntentPacket. Roland reads in <60 seconds, identifies bus-controller as today's priority, marks the brief as "read" (lightweight action recorded), and navigates to Kanban to act on the bus-controller pattern. First-kill decision: 4 minutes from invocation to action.

**Business Success:**
- Skill invocation count tracked per shift (FR55 server-side action log)
- Recommendation triplet stored in the archive (FR29 — Envelope + Provenance Bundle + Validation Receipt)
- Per-claim confidence distribution feeds FR63 criterion 4 (≥80% HIGH+MEDIUM)
- Validator passes: ≥99% replay-verify clean rate across pilot (FR63 criterion 3)

---

## Shortest Path (Q8)

1. **Enter Kondukt view.** Roland clicks "Kondukt" in top chrome. View loads with chat input field, skill registry shortcut bar above it, and empty canvas area on the right (or below, per layout choice).
2. **Invoke Daily Brief skill.** Roland clicks the *Daily Brief* skill button (or types `/daily-brief` in chat input, or uses keyboard shortcut). Visual confirmation: skill button highlights briefly; status indicator shows "Kondukt: running Daily Brief…"
3. **Skill runs (≤20s typical).** Kondukt invokes Rail MCP tools: `get_fleet_telemetry` (last 24h), `get_recurring_failure_pattern` (cross-fleet analysis), `search_fault_history` (semantic similarity to past briefs for tone calibration). The customer's BYOK LLM produces a draft. The harness validator runs: schema check + per-claim citation trace + advisory-only language linter. **All three pass.**
4. **Emission rendered in canvas.** The brief appears in the canvas area — three findings, each with severity badge + confidence pill + IntentPacket citation chips. Recommended focus highlighted at the top of the brief.
5. **Roland reads.** ~60 seconds of reading time. He hovers a citation chip — tooltip shows the IntentPacket ID + a one-line excerpt from the packet. He clicks the bus-controller finding's "View provenance" link → opens the Replay archive triplet detail in a new tab (cross-link to scenario 05.7 surface).
6. **Roland acts.** Decides bus-controller is today's priority. Clicks "Acknowledge — go to Kanban" affordance at the bottom of the brief. Operator action recorded: `kondukt_skill_ack_and_navigate`. Navigates to Kanban (scenario 12) with the bus-controller pattern filter pre-applied. Scenario 13 ends; Kanban triage continues.

---

## Trigger Map Connections

**Persona:** Roland — Fleet Manager (Primary)

**Driving Forces Addressed:**
- ✅ **Want W1:** Daily Brief skill produces a validated overnight situation report (first-kill decision in 4 minutes)
- ✅ **Want W5:** Kondukt skill registry — Daily Brief is a named skill, codified, versioned in-product
- ✅ **Want W6:** Cross-subsystem correlation surfaced by Kondukt (the bus-controller pattern as one of the brief's findings)
- ❌ **Fear F1:** LLM-hallucinated provenance — every claim citation-traced; validator suppresses unsupported claims
- ❌ **Fear F4:** LOW-confidence rubber-stamping — LOW-confidence findings visually distinct per design system DISABLE rule (carries through from Kanban variant)
- ❌ **Fear F9:** Constitution-as-theatre — every brief produces a triplet (Envelope + Provenance Bundle + Validation Receipt) that the Replay CLI verifies deterministically; the brief Roland reads at manday 1 is replay-verifiable at manday 180

**Business Goal:** Trigger Map Objective 3 (Provenance Replay archive integrity — the Daily Brief is the highest-frequency skill invocation; if this skill's triplets are clean, the archive is clean)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 13.1 | `13.1-skill-invocation/` | Kondukt view; skill registry shortcut bar; chat input; invoke *Daily Brief* — button click, keyboard shortcut, or `/daily-brief` chat command | Skill running indicator visible |
| 13.2 | `13.2-validated-emission/` | Brief renders in canvas after validator passes — three findings, each with citation chips, severity badges, recommended focus highlighted | Roland reads |
| 13.3 | `13.3-confidence-pill-provenance/` | Confidence pill rendering (HIGH / MEDIUM / LOW per FR27); IntentPacket citation chips clickable to hover-popover; "View provenance" link to Replay archive | Roland hovers / clicks citation |
| 13.4 | `13.4-suppression-banner/` | Variant: validator fails (e.g. citation-trace check fails). Skill is **suppressed**, NOT regenerated (FR25). Banner: "Kondukt suppressed a recommendation — reason: {reason}. Run again or simplify your query." | Roland re-invokes skill manually or moves on |

---

## Open Questions (for Phase 4 page specs)

- **Skill invocation latency target** — working assumption: ≤20s typical (BYOK LLM + 2 MCP calls + validator + render). Slow runs (>30s) need a progress indicator beyond the static "running" status. Phase 4 confirms.
- **Brief structure (findings hierarchy)** — working assumption: top section = "Recommended focus today" (1–3 findings); middle = "Overnight changes" (full list); bottom = "Awaiting your input" (in-flight items from previous shift). Phase 4 confirms with Roland-test.
- **Findings count** — typical Daily Brief expected to have 3–8 findings. Edge: a quiet night with 0 findings ("No significant overnight changes. Fleet healthy."). Edge: a noisy night with >15 findings (truncation + "view all" affordance). Phase 4 confirms.
- **Re-invocation cadence** — Roland may run Daily Brief multiple times per shift (e.g. after a coffee break). Each invocation produces a fresh triplet. Does the harness cache the previous brief for comparison ("diff from previous brief")? Phase 4 — possibly a Phase 1.5 capability.
- **Canvas layout** — chat input on left + canvas on right (split-pane)? Or chat input below skill output (vertical stack)? Working assumption: split-pane horizontally on ≥1440 width, vertical stack below; chat persistent. Phase 4 confirms.
- **Acknowledge / navigate affordance** — bottom of brief has "Acknowledge — go to Kanban" or "Acknowledge — go to Live Board" or just "Acknowledge"? Working assumption: contextual based on the dominant finding (if findings reference specific trains → suggest Kanban; if findings are fleet-wide patterns → Live Board; default: Kanban). Phase 4 confirms.
- **Skill failure modes** — validation failure (suppression — 13.4); BYOK provider 429 (scenario 10 harness-degraded); MCP tool timeout (scenario 10 suppression variant); LLM produces malformed output (scenario 10 suppression). Phase 4 confirms whether 13.4 covers all or each has its own variant.

---

_Scenario authored 2026-05-25 — harness-model MVP, Sprint 4 cluster._
