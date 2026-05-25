# 10: Kondukt Degraded — Stefan Keeps Working

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** 5
**Persona:** Stefan the Disponent (Primary)

---

## Transaction (Q1)

**What this scenario covers:**
Stefan is on shift. The BYOK Kondukt endpoint starts returning 429s (rate limit) sustained beyond 24 hours. The platform degrades gracefully: Kondukt chat panel shows an explicit degraded state; triage and crisis views remain fully functional; RAO generation is disabled until the endpoint recovers. Stefan works through a fleet incident using the platform's non-Kondukt capabilities — triage cost-impact one-liner, Rail MCP tool results in chat (without Kondukt summarising), pgvector semantic search over incident summaries.

---

## Business Goal (Q2)

**Goal:** Operator continuity — platform remains useful even when the LLM is down; this is the resilience evidence Renate's audit committee will ask about
**Objective:** Trigger Map Objective 5 (Phase 2 readiness — resilience criterion within trust composite) + supporting Objective 1

---

## User & Situation (Q3)

**Persona:** Stefan the Disponent (Primary)
**Situation:** Stefan is on shift, 11:32. The platform's degraded banner has been visible since 09:08 — sustained 429 from the BYOK Kondukt endpoint. A disruption fires: Unit 4905 — door motor fault.

---

## Driving Forces (Q4)

**Hope:** Get the same operational support without Kondukt — cost visibility, structured data lookups, search — and have the platform be honest about what is and isn't available.

**Worry:** A platform that fails subtly — Kondukt silently degrades but the UI says nothing, and Stefan acts on stale or absent recommendations.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Desktop (Stefan's workstation)
**Entry:** Stefan is in default triage view. A persistent degraded banner sits in the chrome: "Fleet intelligence temporarily unavailable — RAO generation disabled. Triage, search, and direct tool calls remain available. Last successful Kondukt response: 09:08."

---

## Best Outcome (Q7)

**User Success:**
Stefan handles the door motor incident in 4 minutes using triage + direct Rail MCP tool calls + pgvector semantic search — no RAO. He logs his action via "Mark executed" tied to a manual decision (not to an RAO). The platform records: action_logged_at, IdP, decision_basis = "Manual (Kondukt degraded)".

**Business Success:**
One operator-action recorded during degraded state with clear `decision_basis` tag. Phase 2 readiness composite has resilience evidence: under sustained Kondukt outage, decision rate did not drop to zero. This is the rebuttal to "what if the LLM is down for a day?"

---

## Shortest Path (Q8)

1. **Triage view with degraded banner** — Default state with persistent banner: "Fleet intelligence temporarily unavailable — RAO generation disabled. Triage, search, and direct tool calls remain available. Last successful Kondukt response: 09:08." Banner is non-dismissable.
2. **Disruption fires — crisis modal** — "Unit 4905 — Door Motor Fault, Car 2. Active unresolved incident. View?" Stefan taps yes.
3. **Crisis full-width (degraded variant)** — Layout transitions to full-width but Kondukt chat panel is replaced with: "Kondukt is unavailable. Tool calls (without Kondukt summary) and search are available." The chat composer is replaced with two affordances: "Direct tool call" and "Search incident history".
4. **Direct tool call — get_maintenance_cost_estimate** — Stefan clicks "Direct tool call". A picker shows the 9 Rail MCP tools. He picks `get_maintenance_cost_estimate` and fills the form (unit_id, replacement_unit_id). The tool result renders raw — no LLM summary, no RAO. Just the JSON-style structured response: `{maintenance_overhead_eur: 2400, confidence: medium, rationale: "..."}`.
5. **Search — incident history** — Stefan clicks "Search incident history". pgvector semantic search query input. He types: "door motor fault, replacement unit, similar resolution." Search returns 5 ranked incident summaries with one-click "Open" affordance per result. He reads two — both reinforce his judgment.
6. **Manual decision + Mark executed** — Stefan opens HAFAS in his other tab, executes the rotation update manually (same as scenario 01 but without the RAO scaffolding). He returns to the platform. A "Mark manual decision" affordance is visible in the degraded crisis view. He taps it. A short form: action description (free-text), affected unit, affected rotation, executed-at timestamp. He fills it. Submit.
7. **Confirmation + return to triage** — The platform records: action_logged_at, IdP, decision_basis = "Manual (Kondukt degraded since 09:08)", action description. No RAO row is created (no recommendation existed). Background outcome polling against HAFAS read endpoint kicks off the same way. Stefan returns to triage. The action is in his execution log this shift, distinct visual styling vs. RAO-driven actions. ✓

---

## Trigger Map Connections

**Persona:** Stefan the Disponent (Primary)

**Driving Forces Addressed:**
- ✅ **Want W1:** Cost consequence visible before confirming — direct tool call surfaces the cost number without RAO scaffolding
- ✅ **Want W2:** Structured proposals — degraded state offers structured data (tool result + search), no raw telemetry
- ❌ **Fear F1:** Autopilot accusation — "Manual decision" is unambiguously Stefan's action; decision_basis tag is explicit
- ❌ **Fear F4:** Phantom recommendations — RAO generation is *disabled* (not silently failing). UI is honest.

**Business Goal:** Trigger Map Objective 5 (Phase 2 readiness composite — resilience criterion); Objective 1 (continuity of prevented-delay-minutes work even under degradation)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 10.1 | `10.1-triage-degraded-banner/` | Persistent non-dismissable degraded banner | Disruption fires |
| 10.2 | `10.2-crisis-modal-degraded/` | Same crisis modal, same entry behaviour | Stefan opens crisis |
| 10.3 | `10.3-crisis-fullwidth-degraded/` | Kondukt chat replaced with direct tool + search affordances | Stefan picks direct tool call |
| 10.4 | `10.4-direct-tool-call-form/` | 9-tool picker; form per tool; raw structured result | Stefan moves to search |
| 10.5 | `10.5-search-incident-history/` | pgvector semantic search; ranked results | Stefan reads results, executes externally |
| 10.6 | `10.6-mark-manual-decision/` | Manual-decision form; decision_basis tagging | Stefan submits |
| 10.7 | `10.7-degraded-execution-log/` | Triage with action logged; distinct styling vs RAO-driven | ✓ |

---

## Open Questions (for Phase 4)

- **Banner copy** — exact phrasing tested with Stefan/Roland — "Fleet intelligence temporarily unavailable" vs "Kondukt unavailable" vs "AI co-pilot unavailable"? Affects how operators perceive the degradation. Phase 4 spec.
- **Direct-tool-call form complexity** — each of the 9 tools has different parameter shapes. Form-per-tool is straightforward but adds 9 spec surfaces. Phase 4 spec — likely just 1-2 specs (the most common tools), the rest generated from MCP schema.
- **Manual-decision action description structure** — free-text only, or structured fields (rotation_id, replacement_unit, effective_at)? Structured aligns with RAO chain; free-text is faster. Phase 4 spec.
- **Degraded-state visual styling for execution log** — how distinct? A different background colour? A "manual" badge? Phase 4 spec — affects how Renate's audit-committee artefact differentiates.
- **Resilience threshold** — how long sustained 429s before degraded UI activates? PRD pilot-kill says ">24h sustained 429/5xx" — but degraded UI should activate sooner (e.g., 5 min sustained 429). Phase 4 spec.
- **Pgvector fallback within search** — if pgvector itself is degraded (PRD pilot-kill trigger), search shows "Keyword search active" label. Combined-degradation scenario (both Kondukt AND pgvector down) — separate scenario or sub-case of 10? Phase 4 spec.
