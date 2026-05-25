# Design Log — ÖBB Landside Platform

> Log of design decisions, current focus, and backlog across WDS phases.

---

## Current

- **Phase 1 Product Brief — COMPLETE** (2026-05-25, Suggest mode)
  - Output: `_bmad-output/A-Product-Brief/project-brief.md` (simplified)
  - Synthesized from: `prd.md`, `commercial-model-2026-05-23.md`, `oebb-role-research-2026-05-23.md`, `go-to-market-notes-2026-05-23.md`, `ux-decisions-2026-05-23.md`, `roadmap-to-fundraise-2026-05-23.md`
- **Phase 2 Trigger Map — COMPLETE** (2026-05-25, Suggest mode)
  - Outputs:
    - `_bmad-output/B-Trigger-Map/trigger-map.md` (poster + Mermaid visualization)
    - `_bmad-output/B-Trigger-Map/personas/01–05` (Stefan, Klaus, Elena, Lukas, Renate)
    - `_bmad-output/B-Trigger-Map/feature-impact-analysis.md` (capability tiers + risk watchlist + sequencing recommendation)
- **Phase 3 UX Scenarios — COMPLETE** (2026-05-25, Suggest mode)
  - Outputs:
    - `_bmad-output/C-UX-Scenarios/00-ux-scenarios.md` (index + coverage matrix + driver matrix)
    - `_bmad-output/C-UX-Scenarios/01–10` (10 scenarios sequenced by sprint plan, Stefan ×4, Klaus ×1, Elena ×1, Renate ×1, Lukas ×2, multi-persona ROI ×1)
  - Naming: all references to "Hermes" → "Kondukt" applied across Phase 1–3 outputs
- **Phase 4 UX Design — IN PROGRESS** (2026-05-25, Suggest mode)
  - **Sprint 1 cluster COMPLETE** — 12 page specs across scenarios 01 + 02 (Stefan crisis + cost-aware triage); 5 Roland-tagged questions closed via party mode
  - **Sprint 2 cluster COMPLETE** — 9 page specs: scenario 05 in full (Renate audit-export review, 6 pages) + scenario 03 tail (Klaus hold-to-record gesture, 3 pages). Establishes the audit-reconciliation grammar Renate signs off at manday 15.
  - **Sprint 3 cluster COMPLETE** — 12 page specs: scenario 03 head (4 pages: push notification, brief, two card details) + scenario 04 in full (Elena ACK + DEFER on shared device, 8 pages). Establishes the depot PWA grammar — role-share-without-identity-share, per-action IdP auth, brake-wear absence, DEFER reason taxonomy LOCKED v0.
  - Design System seed: `_bmad-output/D-Design-System/00-design-system.md` (spacing, type, severity bands, confidence pills, operator-verb rules LOCKED, hold-to-record gesture spec)
  - Outputs (12 specs):
    - `01.1-triage-view` (foundational; defines the contextual panel + footer grammar)
    - `01.2-crisis-modal`
    - `01.3-crisis-fullwidth` (defines chat container grammar)
    - `01.4-tool-call-render` (defines Kondukt tool-call render grammar)
    - `01.5-rao-card` (defines RAO card grammar — *the* defining component)
    - `01.6-mark-executed` (defines hold-to-record gesture entry — shared with Klaus 03.6)
    - `01.7-execution-log` (defines execution-log contextual surface)
    - `02.1-triage-view` (inherits 01.1; documents WARNING vs CRITICAL cost-impact framing)
    - `02.2-card-expand`
    - `02.3-rao-card` (inherits 01.5; equal-weight Modify + Mark Executed buttons)
    - `02.4-modify-panel` (immutable-original contract; reason-for-modification required)
    - `02.5-execution-log-with-modification` (modification-rate signal manifests visibly)

---

## Design Loop Status (per-page status, latest row per page = current state)

| Scenario | Page | Status | Date |
|---|---|---|---|
| 01 | 01.1-triage-view | spec drafted | 2026-05-25 |
| 01 | 01.2-crisis-modal | spec drafted | 2026-05-25 |
| 01 | 01.3-crisis-fullwidth | spec drafted | 2026-05-25 |
| 01 | 01.4-tool-call-render | spec drafted | 2026-05-25 |
| 01 | 01.5-rao-card | spec drafted | 2026-05-25 |
| 01 | 01.6-mark-executed | spec drafted | 2026-05-25 |
| 01 | 01.7-execution-log | spec drafted | 2026-05-25 |
| 02 | 02.1-triage-view | spec drafted (inherits 01.1) | 2026-05-25 |
| 02 | 02.2-card-expand | spec drafted | 2026-05-25 |
| 02 | 02.3-rao-card | spec drafted (inherits 01.5) | 2026-05-25 |
| 02 | 02.4-modify-panel | spec drafted | 2026-05-25 |
| 02 | 02.5-execution-log-with-modification | spec drafted (inherits 01.7) | 2026-05-25 |
| 05 | 05.1-audit-export-view | spec drafted (audit grammar foundation) | 2026-05-25 |
| 05 | 05.2-compare-with-paper | spec drafted | 2026-05-25 |
| 05 | 05.3-supersedes-example | spec drafted (correction model demo) | 2026-05-25 |
| 05 | 05.4-supersedes-detail | spec drafted (field-level diff) | 2026-05-25 |
| 05 | 05.5-reconciliation-queue | spec drafted (Phase 2 threshold surface) | 2026-05-25 |
| 05 | 05.6-divergence-detail | spec drafted (single-page audit artefact) | 2026-05-25 |
| 03 | 03.5-return-record-entry | spec drafted (advisory framing LOCKED) | 2026-05-25 |
| 03 | 03.6-hold-to-record | spec drafted (gesture spec mirrors design system + Stefan 01.6) | 2026-05-25 |
| 03 | 03.7-signoff-success | spec drafted | 2026-05-25 |
| 03 | 03.1-push-notification | spec drafted (Web Push payload, source=push tracking) | 2026-05-25 |
| 03 | 03.2-prearrival-brief | spec drafted (ECM Manager lens, role-filtered cards) | 2026-05-25 |
| 03 | 03.3-card-detail-critical | spec drafted (resolution chain timeline) | 2026-05-25 |
| 03 | 03.4-card-detail-warning | spec drafted (DEFERRAL DETAIL section, links to Elena's 04.7) | 2026-05-25 |
| 04 | 04.1-role-selector | spec drafted (4-hour TTL, role-share-no-identity-share LOCKED) | 2026-05-25 |
| 04 | 04.2-electrical-vehicle-list | spec drafted (role-filtered, pre-IdP) | 2026-05-25 |
| 04 | 04.3-vehicle-card-stack-electrical | spec drafted (brake-wear absent, DEFER-blocked-on-CRITICAL) | 2026-05-25 |
| 04 | 04.4-card-detail-with-ack | spec drafted (ACK preview before IdP commit) | 2026-05-25 |
| 04 | 04.5-idp-auth-per-action | spec drafted (WebAuthn primary + PIN fallback, per-action absolute) | 2026-05-25 |
| 04 | 04.6-ack-confirmation | spec drafted (identity confirmation line non-removable) | 2026-05-25 |
| 04 | 04.7-card-detail-with-defer-reason | spec drafted (DEFER reason taxonomy LOCKED v0, 7 items) | 2026-05-25 |
| 04 | 04.8-defer-confirmation | spec drafted (scenario-completion indicator, undo-within-30s) | 2026-05-25 |

**Sprint 4–5 (scenarios 06–10) — not yet started.** 33 pages remaining (7 in scenario 06 + 6 in scenario 07 + 6 in scenario 08 + 7 in scenario 09 + 7 in scenario 10).

---

## Backlog

- **Phase 4 Sprint 4 cluster** — scenarios 06 (Lukas vendor warranty dispute, 7 pages) + 07 (Lukas OTA canary + drift, 6 pages). Establishes the three-zone Telematics Control Board grammar. 13 pages total.
- **Phase 4 Sprint 5 cluster** — scenarios 08 (ROI conversation, 6 pages) + 09 (Stefan handover, 7 pages) + 10 (Kondukt degraded, 7 pages). 20 pages.
- **Phase 4 Sprint 4 cluster** — scenarios 06 (Lukas warranty) and 07 (Lukas OTA)
- **Phase 4 Sprint 5 cluster** — scenarios 08 (ROI conversation), 09 (Stefan handover), 10 (Kondukt degraded)
- **Validation of Sprint 1 with Roland Ruisz** — scenarios 01 + 02 are the RAO grammar locking specs; Roland review is the unblocker for Sprint 2 commit
- **Validation: Roland Ruisz** — primary-persona driver list + scenarios 01, 02, 06, 07, 09 (Stefan + Lukas dual-hat) confirmed by real-world equivalent
- **Validation: Renate Fischer** — gatekeeper driver list + scenario 05 + audit-export surface in 08
- **Validation: ECM Manager peer** — scenario 03 (Klaus archetype) walkthrough
- **Validation: ECM 4 Electrical technician peer** — scenario 04 (Elena archetype) walkthrough
- **Pre-pilot blocker: `role_relevance` taxonomy** — open decision Q1 in `ux-decisions-2026-05-23.md` — needs ECM Manager sign-off pre-MVP UX lock (affects scenarios 03, 04)
- **Pre-pilot blocker: Probebetrieb cycle overlap** — confirm with Lukas's programme team before pilot manday 1 (affects scenarios 06, 07 evidentiary status)
- **Pre-pilot blocker: ÖBB SSO IdP federation live by pilot manday 1** — affects every scenario (login pre-condition) + scenario 05 (Renate review)
- **Pre-pilot blocker: DEFER reason taxonomy** — ECM Manager sign-off needed (affects scenario 04)
- **Roland queue: F3 scope question** — does Renate's F3 (Phase 2 pre-emption concern) cover the ECM-4 maintenance contractor chain, or is it purely about internal-ÖBB pre-emption? Mary flagged 8–15 subcontractor QA manuals as a missed stakeholder. The invariant-noun label ("Audit record — advisory/authoritative mode") mitigates this regardless, but the answer determines whether the Phase 2 cutover runbook needs contractor-specific comms artefacts. Not blocking — confirm at next Roland touchpoint.
- **Phase 2 cutover runbook stub** — needed in planning-artifacts before Phase 2 trust-criteria sign-off event. Covers Schienen-Control letter, ECM audit committee briefing memo, contractor handover-packet template update, doctrine paragraph on "authoritative from date X" semantics. Tracked in planning-artifacts, not design-artifacts. Owner: TBD post-MVP launch.
- **Phase 4 reference reading list** — see decision-log entries for hermes-workspace and hermes-agent patterns to lift

---

## Decision Log

### 2026-05-25 — Shadow record label → invariant-noun label + enum schema (party-mode pass)
- **Context:** Prior lock in `05.1-audit-export-view` (commit 29f58b3) made "Shadow record (companion to paper)" non-removable in MVP, with Phase 2 transition framed as a "single config flag" flip to "Authoritative record." Checkpoint review (party mode) stress-tested this with Saga, Winston, Mary, Mimir.
- **Diagnosis converged across 4 voices:**
  - Saga: "Shadow record" leaks into committee minutes, procurement language, contractor habit during MVP — relabelling becomes a contract-amendment cycle, not a flag flip
  - Winston: "single config flag" was an architecturally false slogan — hash chain means historical rows must keep their original label forever; Phase 2 is "authoritative from date X forward," not "platform becomes authoritative"
  - Mary: ECM-4 maintenance contractor chain (8–15 subcontractor QA manuals) was the missed stakeholder — full vocabulary relabel = 2–4 months of Renate's attention
  - Mimir: schema must be enum-in-canonical + string-in-renderer (option c); enum bound by HMAC, string lives in `labelFor(record_type, locale)`. Option (b) — full string in HMAC bytes — is operationally impossible without breaking the chain
- **Choice (option 1 from convergence):**
  - **Label changed:** "Audit record — advisory mode (paper sign-off is authoritative)" → "Audit record — authoritative mode." Noun invariant across modes; only the `— mode` sub-line changes at Phase 2.
  - **Schema locked (Mimir option c):** `record_type` enum on the canonical row, values `"advisory"` / `"authoritative"`. Enum in HMAC bytes; human-readable string lives in renderer lookup. Per-row enum immutable.
  - **Phase 2 flip:** write-side default change for *new* rows only. Historical advisory rows render as advisory forever, by chain integrity.
  - **Phase 2 cutover runbook:** added to backlog as planning-artifacts work — Schienen-Control letter, audit-committee briefing, contractor handover-packet update, doctrine paragraph. Not code; not in design-artifacts.
- **Confirmed prior to lock:** "Shadow record (companion to paper)" had not yet appeared in any external written communication — cheap-fix window still open. Saga's mandatory migration plan (overlap window + co-signed regulator memo) downgrades to optional given the invariant-noun label.
- **Open question for Roland:** does F3 (Phase 2 pre-emption concern) cover the ECM-4 contractor chain, or is it purely internal-ÖBB? Affects whether the Phase 2 cutover runbook needs contractor-specific artefacts. Not blocking; queued for next Roland touchpoint.
- **Reversibility:** Schema lock is a one-way door once Sprint 4 commits (changing `record_type` semantics after audit data accumulates invalidates the dataset). Label string is reversible (renderer lookup, one line). Lock the schema before Sprint 4.
- **Revisit trigger:** Phase 2 trust-criteria sign-off event — verify cutover runbook is complete and the invariant-noun label is doing the F3 protection work in practice (operator habit, contractor manuals, audit-committee speech all still use "Audit record" as the noun).

### 2026-05-25 — Simplified Brief over Complete Brief
- **Context:** Project has rich existing PRD + research; Complete brief would re-litigate settled decisions
- **Choice:** Simplified brief (one-page North Star) over Complete (36-step + Content + Visual + Platform docs)
- **Reversibility:** Two-way door — can expand to Complete later if Phase 3/4 surfaces gaps
- **Revisit trigger:** If Freya in Phase 3 finds the simplified brief insufficient for scenario authoring

### 2026-05-25 — Suggest mode over Workshop
- **Context:** Documentation is rich and pre-validated through PRD adversarial review (commits b34b679, 0fcafb0)
- **Choice:** Suggest mode (Saga drafts from existing docs, user redlines) over Workshop (one question at a time)
- **Reversibility:** Two-way door
- **Revisit trigger:** If draft accuracy is materially wrong on any persona — switch to Workshop for that persona

### 2026-05-25 — Renate Fischer included as Trigger Map persona
- **Context:** Renate is not a daily user but is one of five Phase 2 trust criteria
- **Choice:** Include as "Commercial Gatekeeper" persona with distinct driver set focused on audit-trail integrity
- **Reversibility:** Two-way door — can demote to "stakeholder" if downstream phases don't need her surfaces
- **Revisit trigger:** Phase 3 — if Renate's review surface (audit export + reconciliation queue) doesn't generate scenarios distinct from Klaus's

### 2026-05-25 — Reject `outsourc-e/hermes-workspace` as frontend base
- **Context:** Evaluated `outsourc-e/hermes-workspace` (MIT, 4.8k stars, React 19 + TanStack + Vite + Tailwind, mature Electron app) as candidate base for the landside platform frontend wrapper
- **Choice:** Reject as base. Build frontend fresh on equivalent stack (React 19 + TanStack Router + Vite + Tailwind 4 + Zustand). Lift specific patterns as reference reading: chat SSE + tool-call rendering, MCP catalog UX, auth middleware + path-traversal guard, TanStack file-based routing, `vt-capital` domain-state-with-guardian pattern (model for Lukas's Telematics Control Board)
- **Reasons:**
  1. **Wrong persona** — workspace cockpit aesthetic (chat-first, files, terminal, memory, skills marketplace, 2,000-skill catalog, swarm/conductor/crew) is for the operator-of-the-agent, not the Disponent/ECM Manager/Technician/Telematics PM/ECM Lead
  2. **Wrong regulatory posture** — PTY terminal, full Monaco file editing, swarm-dispatched autonomous agents are all surfaces that NIS2 supplier-chain review + EU AI Act conformity assessment would flag. Renate's audit committee would ask "what does this system *do* if a flag flips?" and the answer would be too long.
  3. **Wrong commercial framing** — Three.js + 3D scene + Rapier physics + Hermes World violates Abbas's principle (`feedback_no_over_designed_systems`). Brand collision with our internal "Hermes" agent. Solo-maintainer bus factor (Eric / outsourc-e, 107/165 commits).
- **Reversibility:** Two-way door for the build; one-way door for the brand decision (rename our internal Hermes — see next entry)
- **Revisit trigger:** Post-MVP if any specific surface (Lukas's Telematics control board, audit export viewer) is being built and the corresponding hermes-workspace screen is genuinely close enough to fork piecemeal

### 2026-05-25 — Reject `NousResearch/hermes-agent` as platform backend; Option A for LLM runtime
- **Context:** Evaluated `NousResearch/hermes-agent` (MIT, 166k stars, Python 3.11+, mature general agent platform from Nous Research) as candidate platform backend
- **Choice:** Reject as platform backend wholesale. Build platform backend fresh. For the LLM agent runtime (the BYOK piece that produces RAOs), use **Option A — direct Anthropic/OpenAI SDK call with MCP-client + structured output**, with the Rail MCP Server as the *only* tool surface
- **Reasons:**
  1. **Vast tool surface hostile to advisory-only posture** — 78 built-in tools include terminal, process, browser (12 tools, CDP + Camofox), file ops, execute_code, delegate_task, cronjob. NIS2 + EU AI Act review would require structural disablement of each, not just a config flag.
  2. **22 messaging-platform adapters** (Telegram/Discord/Slack/WhatsApp/Signal/Email/SMS/Matrix/Mattermost/Teams/DingTalk/WeCom/Weixin/Feishu/QQBot/Bluebubbles/Yuanbao/Home Assistant/LINE/SimpleX/webhook/api_server) — every one is supplier-chain audit code, ÖBB Disponent gets RAOs in the platform, not on Telegram
  3. **Single-tenant personal agent by design** — their SECURITY.md states this explicitly. Misaligned with per-operator-deployment platform service identity
  4. **Codebase scale** — `cli.py` is ~22,800 lines; v0.14.0 alone was 165k LoC inserted. Maintaining a fork is untenable; vendoring inherits the surface
  5. **Brand collision** — Nous owns "Hermes Agent" as project name in the market
- **What's good and worth lifting:**
  - SECURITY.md trust-model framing (use as structural template for ours — name the single boundary, mark heuristics as heuristics, document supported postures)
  - MCP-client integration pattern (reference reading for how to call Rail MCP Server from agent loop)
  - Provider-switching via OpenAI-compatible proxy (conceptual reference for BYOK)
  - agentskills.io standard (if we publish rail-domain skills in Phase 2)
- **Reversibility:** Two-way door — can re-evaluate at Phase 2 when agent surface complexity grows. Renaming our Hermes is one-way.
- **Revisit trigger:** Phase 2 transition planning — when platform-mediated dispatch comes online, agent surface complexity may justify a more substantial runtime. Re-evaluate then.

### 2026-05-25 — Sprint 1 Roland-question locks (party mode resolution)
- **Context:** 5 Sprint 1 UX questions tagged for Roland sign-off. Two party-mode rounds (Winston, Sally, Mary, Mimir, then John, Dr. Quinn, Victor, Saga, Caravaggio + Sally-v2 + Mimir-v2). Goal: answer what we can with our expertise, keep it simple, defer only the genuinely operator-domain ones.
- **Choices:**
  - **Q1 — Low-confidence RAO behaviour:** **DISABLE** "Mark Executed" on LOW; force Modify. Three independent paths converged: Saga (Renate's audit committee reads two row types cleaner than three), John (Modify is the job on LOW), Victor (DISABLE produces Phase 2 evidence dataset). Sally retracted her DISABLE-for-phantom-protection argument after Winston pointed out Mark Executed is a *post-hoc receipt* of a HAFAS-executed action, not a pre-commit. Mimir proposed WITNESS (relabel button), but Saga showed it introduces a third row type that Renate's audit committee will question.
  - **Q2 — Two equal-weight primary buttons (02.3):** LOCKED — keep equal prominence; Winston refinement = outlined Modify + solid Mark Executed (same size, different fill); Mimir = instrument distinct `data-action` attributes for modification-rate analytics.
  - **Q3 — Modification reason taxonomy (02.4):** **6 of 7 items LOCKED** — Mary's seed list grounded in Disponent role + rail-disruption literature. Item 6 (commercial priority) genuinely needs Roland — ÖBB-operator-specific hierarchy. Taxonomy real job: distinguish RAO context gap (1,3,4) vs data freshness (2) vs guardrail working (5) vs RAO quality signal (7).
  - **Q4 — Triage card sort order (01.1):** LOCKED — severity-banded (HIGH → MEDIUM → LOW), newest-first within band, LOW band capped at 5 with "Show more (N)". Sally + Mary converged independently. Evidence: Endsley/Vicente situation-awareness research on long-shift operators.
  - **Q5 — RAO visual treatment (01.5):** SPINE LOCKED — left-rail 3px in confidence-tier colour. Two variants flagged for Roland-test: (a) subtle background tint (Winston + Mimir favour, Sally argues against); (b) "Recommendation" badge (Mimir favours, Sally argues against). Three-way split on decoration; spine all three voices agreed on.
  - **Bonus pickup (Caravaggio):** "Log What You Did" as label alternative for "Mark executed" — plain-language, scans in <1s, possibly stronger than the current label across ALL RAO surfaces. Worth a Roland side-by-side test before locking permanently.
- **Reversibility:** All five LOCKS are reversible (Mimir confirmed the LOW-confidence enum/policy is a one-line flip; visual treatment is a few utility classes; taxonomy is a config). The locks are the *current decision*, not architectural one-way doors. **Caveat (added 2026-05-25, pre-mortem pass):** the `modification_distance` audit-schema field is a one-way door once Sprint 4 build commits — adding it after Phase 2 audit data has accumulated invalidates the dataset. Lock the field before Sprint 4 begins.
- **Revisit trigger (revised 2026-05-25, pre-mortem pass):** **Day-21 OR LOW-rate threshold trip (>15% over 72h), whichever comes first** — not Day-5. Day-5 only measures whether Roland *recognises* the LOW-DISABLE behaviour; it does not measure whether the behaviour holds up at operational frequency. The pre-mortem identified classifier drift (8% → 30%+ LOW) as the highest-probability failure mode, and Day-5 is too early to see it. Day-5 walkthrough with Roland is still on the calendar as a *recognition check*, not as the revisit decision.

### 2026-05-25 — Rename internal "Hermes" agent → Kondukt (resolved)
- **Context:** Our internal LLM co-pilot was called "Hermes" throughout PRD, commercial model, UX decisions, and Rail MCP Server spec. Nous Research owns "Hermes Agent" as a public project name (`NousResearch/hermes-agent`, 166k stars). Brand confusion was a concern for Roland Ruisz / Martin Lerch conversations and an anticipated IP question from Stadler/Siemens procurement ("what's the proprietary value beyond wrapping an OSS tool?")
- **Candidates evaluated:** Kondukt, Telos, Pacer, Vidar, no-name-at-all. Telos rejected (multiple active AI projects: Dan Miessler TELOS, Ramen VR Telos, autonomous-builder Telos). Pacer rejected (Pacer International is a major rail-logistics company — direct domain collision). Vidar rejected (also a known infostealer malware family — bad secondary association in any ÖBB security review). No-name option strong but Abbas chose the proper noun.
- **Choice:** **Kondukt** — German "Konduktor" root → conductor/guide semantics, fits rail co-pilot framing, German-native (Disponent reads it correctly first try), short, novel spelling
- **Namesake check:** `kondukt.app` exists but is low-profile (single Google result, no description, no major OSS or AI project). No GitHub collision. Acceptable.
- **Scope of rewrite (executed 2026-05-25):** `prd.md`, `ux-decisions-update-1-2026-05-23.md`, `oebb-role-research-2026-05-23.md`, `roadmap-to-fundraise-2026-05-23.md`, `company-dashboard-2026-05-23.md`, `rail-mcp-server-spec-2026-05-23.md`, `_archive/party-mode-critical-assessment-2026-05-23.md`, `CLAUDE.md` workspace-scope blurb. **Pending Abbas confirmation:** 11 additional `_bmad-output/` files (UX scenarios, trigger map, personas, project brief) still reference "Hermes".
- **Reversibility:** One-way door — locked in now; further rename gets more expensive as Phase 4 specs commit. Phase 2 dispatch-capability release is the natural moment to revisit if needed.
- **Revisit trigger:** Only if `kondukt.app` later becomes a major collision OR if German-speaking ÖBB users find the spelling awkward in practice (test at first wds-0 alignment session)

---

## Open Backlog Items (added 2026-05-25)

- ~~**Hermes rename decision**~~ — RESOLVED 2026-05-25: renamed to Kondukt. See decisions log above.
- **Reference reading list for Phase 4 implementation:**
  - `outsourc-e/hermes-workspace` — `src/screens/chat/` (SSE + tool-call rendering), `src/screens/mcp/` (MCP catalog), `src/screens/vt-capital/` (domain-state-with-guardian pattern for Lukas's Zone 1/2/3)
  - `NousResearch/hermes-agent` — `mcp_serve.py` and MCP client code (Rail MCP Server integration reference), SECURITY.md (structural template for ours)
