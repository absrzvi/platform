# 12: Roland's Kanban Triage — Neglect-Risk Lane Discipline + Progressive Disclosure

**Project:** ÖBB Landside Platform (harness model)
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** Sprint 3 cluster (Live Board + Kanban — Roland's two glanceable views; Kanban is also the depth surface)
**Persona:** Roland — Fleet Manager (Flottenmanagement / Team Telematik / Teil-PL DOSTO Neu + Enzo)

---

## Transaction (Q1)

**What this scenario covers:**
Roland arrives at the Kanban via Live Board click-through (scenario 11.5). Train 4736-101's detail card is auto-expanded at the primary-disclosure level. He reads the per-component diagnostic depth: each subsystem name + VLAN + IP + MAC + switch port + firmware + time-since-fault. He recognises the CCTV component as Überw. Kamera A7 — IP 172.18.192.138 — the one in the Kondukt-flagged hardware-revision-A7 pattern. He clicks the component row to reveal secondary disclosure: MTTR (harness-computed from landside-observed Stadler alarm timestamps) + recurring-failure history + last 24h packet trace + vendor portal deep-link. He sees this camera has failed 4 times in the last 90 days, MTTR has been trending up. He decides this card is worth promoting to the Investigating lane. Kondukt has already proposed exactly that move with a HIGH neglect-risk score; Roland accepts the move with a single click. The Kanban card relocates to Investigating; his action is recorded.

---

## Business Goal (Q2)

**Goal:** Neglect-risk lane discipline — Roland's continuous all-day surface for "which incident am I ignoring that will bite me by Friday." Trigger Map Objective 1 (per-view acceptance ≥70% on Kanban lane moves). Also: the **depth surface** for the harness — primary + progressive disclosure on the train detail card replaces VLAN/IP/MAC lookups across vendor portals + Power BI + spreadsheets.

**Objective:** Per-view acceptance rate ≥70% on Kondukt's lane-move proposals; secondary-disclosure click-through rate (proxy for "operators are using Kanban as the diagnostic depth surface") tracked as engagement signal.

---

## User & Situation (Q3)

**Persona:** Roland — Fleet Manager (real-world: Roland Ruisz, DI(FH) — Flottenmanagement / Team Telematik / Teil-PL DOSTO Neu + Enzo).

**Situation:** Roland just clicked train 4736-101 on Live Board (scenario 11.5 redirect). Kanban view loads with the 4736-101 card located in the New lane, auto-expanded at primary disclosure. There are 18 open faults across the fleet today, distributed across the four lanes. Kondukt has proposed lane-placement scores per card with provenance.

---

## Driving Forces (Q4)

**Hope:** See the full per-component diagnostic depth in one expanded card — VLAN, IP, MAC, switch port, firmware version. Click to reveal MTTR + recurring-failure history + 24h packet trace + vendor portal deep-link. Replace the vendor-portal-lookup + Power-BI-historical reconciliation friction with progressive disclosure on one card.

**Worry:** Information density on the expanded card too high — Roland can't find the right component in a multi-subsystem train; OR not enough — he has to navigate elsewhere for diagnostic depth that *should* be on the card. The progressive disclosure split is the key UX choice — primary visible by default, secondary on demand.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Desktop browser. ≥1920×1080 primary; 1280–1920 functional. Modern Chromium-based browser.

**Entry:** Roland clicked train 4736-101 on Live Board (scenario 11.5). Kanban opens with that train's card located in its current lane (New) and auto-expanded at primary disclosure. The page scrolls to position the card centre-of-view. Card has a subtle highlight for 2 seconds confirming "this is the one you clicked."

---

## Best Outcome (Q7)

**User Success:** Roland reads the primary-disclosure view in <15 seconds; identifies the Überw. Kamera A7 component as the Kondukt-flagged one; clicks to reveal secondary disclosure; sees the recurring-failure history (4 incidents / 90 days); accepts Kondukt's proposed lane move (New → Investigating) with a single click. Total time from Live Board click to Investigating-lane commit: <90 seconds.

**Business Success:**
- Per-view acceptance rate on Kanban lane-move proposals ≥70% within first 30 pilot mandays (FR63 criterion 1)
- Secondary-disclosure click-through rate non-zero — operators are using Kanban as the diagnostic depth surface, not as a queue
- Card-expansion → action time tracked per Roland (proxy for "harness is faster than current practice")

---

## Shortest Path (Q8)

1. **Entry from Live Board click-through (scenario 11.5).** Kanban renders, train 4736-101 card is in the New lane with auto-expansion at primary disclosure. Page scroll positions the card centre-of-view. 2-second highlight confirms.
2. **Roland reads primary disclosure.** Per-component rows: each shows subsystem name (operator-friendly) + VLAN + IP + MAC + switch port + firmware + time-since-fault. Among the rows: CCTV — Überw. Kamera A7 — VLAN 5 — IP 172.18.192.138 — port e1-0 — firmware v2.3.1 — fault-active for 4h 23m.
3. **Roland clicks the CCTV row to reveal secondary disclosure.** Inline reveal: MTTR (harness-computed: 3.2h average across recurring incidents on this component) + recurring-failure history (4 incidents in last 90 days, each visible with timestamps) + last 24h packet trace (compact log lines) + vendor portal deep-link (Stadler portal URL pre-filled with this MAC).
4. **Roland reads Kondukt's lane-move proposal.** On the card, a small Validation-Receipt-backed banner: *"Move to Investigating — neglect-risk HIGH. Source: recurring-failure pattern + 4-incident history + 47-day firmware age. Confidence: HIGH."* Validator pass visible.
5. **Roland accepts the move.** Single click on "Accept move" → card relocates from New to Investigating lane with a brief animation. Operator action recorded as `operator_action` with `action_type='kanban_lane_move'`, `recommendation_id` of the Kondukt proposal, IdP-bound identity, timestamp.
6. **Returns to broader Kanban view.** Page scroll restores; Roland sees the rest of his Kanban. Next card in New lane has its own Kondukt proposal he can scan and act on.

---

## Trigger Map Connections

**Persona:** Roland — Fleet Manager (Primary)

**Driving Forces Addressed:**
- ✅ **Want W2:** Neglect-Risk Kanban surfaces decay-without-attention (not queue order)
- ✅ **Want W4:** Decay-without-attention + depth on demand — primary disclosure on expansion; secondary on click
- ✅ **Want W6:** Cross-subsystem correlation surfaced by Kondukt (the lane-move proposal carries the pattern's neglect-risk score)
- ✅ **Want W8:** MTTR computed by harness from landside-observable timestamps — surfaced on secondary disclosure
- ❌ **Fear F1:** LLM-hallucinated provenance — Kondukt's lane-move proposal carries Validation Receipt
- ❌ **Fear F2:** Reconstructability gap — every operator action recorded against IdP identity with timestamp + recommendation_id
- ❌ **Fear F4:** LOW-confidence rubber-stamping — LOW-confidence proposals' Accept is disabled; Modify is the only path with cursor pre-placed in justification (sub-page 12.5)

**Business Goal:** Trigger Map Objective 1 (per-view acceptance) + Objective 3 (Provenance Replay archive integrity — every lane-move proposal produces a triplet)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 12.1 | `12.1-lanes-view/` | Default Kanban view — four lanes (New / Investigating / Recommendation Pending / Cleared); neglect-risk sort within lane; Kondukt-proposed lane moves visible per card; filters (train / subsystem / lane / neglect-risk band) | Roland scans lanes; identifies the card he just clicked from Live Board |
| 12.2 | `12.2-lane-move-acceptance/` | Roland accepts a Kondukt-proposed lane move with a single click; card relocates with brief animation; action recorded | Move committed; operator interaction logged |
| 12.3 | `12.3-train-detail-primary/` | Auto-expanded train detail card at primary disclosure — subsystem name + VLAN + IP + MAC + switch port + firmware + time-since-fault per component | Roland identifies the component to investigate; clicks it for secondary disclosure |
| 12.4 | `12.4-train-detail-secondary/` | Secondary disclosure on component click — MTTR (harness-computed) + recurring-failure history + 24h packet trace + vendor portal deep-link | Roland reads diagnostic depth; decides on action |
| 12.5 | `12.5-low-confidence-on-card/` | LOW-confidence variant: Kondukt's lane-move proposal is LOW confidence; Accept button is non-interactive; Modify is the only path with cursor pre-placed in justification field; modification_distance captured on commit | Roland modifies the proposed move or rejects with reason |

---

## Open Questions (for Phase 4 page specs)

- **Lane layout direction** — horizontal lanes (left-to-right: New → Investigating → Recommendation Pending → Cleared) is the working assumption. Vertical lanes were considered. Horizontal is the dominant Kanban convention. Phase 4 confirms.
- **Card sort within lane** — primary: neglect-risk score (highest first). Secondary: time-since-fault (oldest first as tiebreaker). Tertiary: train ID alphabetical. Phase 4 confirms tiebreaker semantics.
- **Manual sort override** — Roland can override Kondukt-computed neglect-risk sort with a manual sort toggle (per FR8). Manual sort options: by train ID, by time-since-fault, by subsystem. Phase 4 confirms.
- **Drag-and-drop vs button-only lane moves** — both supported per FR9 ("drag-and-drop or keyboard"). Phase 4 confirms drag-and-drop interaction details + accessibility.
- **Card expansion state preservation** — when Roland navigates away from Kanban (to Live Board or Kondukt) and back, does his expanded card stay expanded? Working assumption: yes, expansion state persists for the session. Phase 4 confirms.
- **Multi-card-expansion** — can Roland have multiple cards expanded simultaneously? Working assumption: yes, no exclusivity. Each card's secondary disclosure is independent.
- **Secondary-disclosure component-row exclusivity** — within a card, if Roland clicks one component row to reveal secondary disclosure and then clicks another, does the first collapse? Working assumption: independent — both reveal simultaneously.
- **Recommendation Pending lane semantics** — what marks a card as "Recommendation Pending"? Working assumption: Kondukt has emitted a recommendation that requires operator action (e.g. depot scheduling, parts procurement) that hasn't been logged yet. Phase 4 confirms — may need explicit `lane_signal_type` enum.
- **Card visual treatment when secondary disclosure is fully revealed across all components** — does the card expand vertically beyond the viewport? Working assumption: yes; card uses internal scroll if necessary. Phase 4 confirms.
- **Filters: persistence** — filter state per-user across sessions? Working assumption: filters persist per-user, like pivot state on Live Board (FR3 generalised). Phase 4 confirms.

---

_Scenario authored 2026-05-25 — harness-model MVP, Sprint 3 cluster._
