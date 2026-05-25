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

### 2026-05-25 — Scenario cleanup (Option B): selective retire + revise in place + new 11+ scenarios

- **Context:** With strategic layer (brief / PRD / trigger map / FIA / persona) fully aligned to harness model, the existing 10 UX scenarios were assessed against the new product. Scenarios 01/02 (Stefan crisis + cost-aware triage) were built around retired RAO card grammar + Mark-executed gesture; scenarios 06/07 (Lukas warranty + OTA) describe retired persona + descoped features. Scenarios 03/04/05/08/09 substantially survive with revisions; scenario 10 needs rewrite for three-surface harness-degraded model. Plus 5 new MVP-harness scenarios needed (Live Board, Kanban, Kondukt-Daily-Brief, Kondukt-Vendor-meeting-prep, BYOK setup).
- **Decision:** Option B — selective retire + revise in place + new scenarios at 11+:
  - **Retired to `_archive-pre-harness-2026-05-25/`:** 01 (Stefan crisis redeployment), 02 (Stefan cost-aware triage), 06 (Lukas vendor warranty), 07 (Lukas OTA). Archive README authored with per-scenario retirement rationale + reversibility note.
  - **Revise in place (keep existing sub-page specs):** 03 Klaus (drop 03.6 hold-to-record gesture; simplify 03.7 to single-click); 04 Elena (RAO refs update; persona unchanged); 05 Renate (add new sub-pages 05.7 Replay CLI demo + 05.8 Suppression log inspection); 08 Roland-Martin ROI (adoption scorecard + 7-criteria composite + Replay archive integrity row); 09 Roland handover (handover summary scope expanded — Kanban-lane-state + Kondukt-skill-invocation-history + suppressed-emission-log + unverified-outcome states).
  - **Rewrite in place:** 10 (three-surface harness-degraded per PRD Journey 8 — stale telemetry / MCP timeout / BYOK unreachable).
  - **New harness-MVP scenarios at 11+:** 11 Live Board, 12 Kanban, 13 Kondukt Daily Brief skill, 14 Kondukt Vendor meeting prep skill, 15 BYOK setup.
- **Number gaps preserved as retirement signal.** No 01/02/06/07 in active scenarios — gap signals retirement. Readers find the archive folder for trail.
- **Sub-page granularity decision:** keep existing sub-page specs (03.1–3.7, 04.1–4.8, 05.1–5.6, etc.). Sub-pages with retired content (e.g. 03.6 hold-to-record) get retired or replaced; sub-pages whose content survives get revised in place. New scenarios at 11+ follow the same sub-page pattern where the surface earns it (11.x, 12.x with multiple sub-pages; 13/14/15 may be simpler).
- **Stefan→Roland rename in scenario contents** applied as part of revision pass (deferred from the earlier rename round which was strategic-docs-only). Folder names (e.g. `09-stefan-shift-handover/`) and filenames retained pending a later deliberate filename sweep.
- **Index doc (00-ux-scenarios.md) rewritten** with the new active-vs-archived split, new Page Coverage Matrix (Live Board / Kanban / Kondukt + Replay CLI demo + Suppression log + BYOK setup; retired pages listed at the bottom), new Driver Coverage Matrix aligned to revised trigger map drivers.
- **Reversibility:** Archived scenarios preserved on disk. If any retired feature returns to MVP scope (e.g. warranty recovery becomes a Phase 1.5 commit), the archive is a starting reference — not a spec, because the surfaces those scenarios described (RAO card grammar, three-zone Telematics Control Panel) are also retired.

---

### 2026-05-25 — MTTR sourcing reframed + deliberate-confirmation gestures descoped from MVP

Two operator corrections applied across the strategic layer:

**Correction 1 — MTTR computed by harness, not read from Stadler.**
- Previous framing: "MTTR sourced from Stadler diagnostic system (alert created → cleared); read by harness, not recomputed."
- Corrected framing: The harness computes MTTR from **landside-observable timestamps** — `t_open` = arrival timestamp of the Stadler "alarm raised" IntentPacket on the landside; `t_close` = arrival timestamp of the corresponding Stadler "alarm cleared" IntentPacket; `MTTR = t_close − t_open`. The harness does NOT read Stadler's internal MTTR field.
- **Why this matters:** (a) the landside-observed window is the operator-experienced MTTR — what an ÖBB operator actually had visibility into; (b) brain→landside forwarding latency is included on both ends — the honest number; (c) the harness owns this metric without trusting Stadler's internal accounting; (d) the Stadler API contract surface shrinks — we don't need MTTR fields from them, just the open and close alarm events which are normal IntentPacket payload.
- **Where applied:** Roland persona file (W8 reworded, design-implications table row updated), trigger map (driver list + design focus statement), feature impact analysis (Tier 2 capability row), product brief, PRD §3 Executive Summary + Live Board view spec + Kanban detail card secondary disclosure spec + FR4 + Rail MCP toolset descriptions throughout + Live Board capability row.

**Correction 2 — Deliberate-confirmation gestures (Mark-executed + hold-to-record) descoped from MVP.**
- Previous framing: Mark-executed gesture on recommendations (Roland-side) + hold-to-record gesture (Klaus-side, 3s hold + 5s countdown + atomic write at expiry) preserved across MVP shadow and Phase 2 authoritative writes. Trust-building muscle memory for Phase 2.
- Corrected framing: **No deliberate-confirmation gestures in MVP.** Roland's recommendation interactions are simple-click — accept / modify / reject / lightweight mark-seen. Klaus's ECM shadow sign-off is a simple click in MVP. The hold-to-record gesture spec is preserved in `design-artifacts/ecm-signoff-interaction-spec.md` as a **Phase 2 candidate** — when the platform's sign-off becomes authoritative under EU 2019/779, the deliberate-confirmation gesture is reconsidered.
- **Why this matters:** the shadow trail in MVP is additional to (not replacement for) the paper sign-off that remains regulatorily authoritative. The deliberate-confirmation friction is appropriate where the click *is* the regulatory authority; in MVP, paper carries that weight. Simple click + IdP-bound identity at moment of click + append-only DB-level immutability is sufficient for the shadow trail. The "muscle memory for Phase 2 dispatch" justification is removed.
- **Roland's W9** (muscle memory for gestures that become authoritative in Phase 2) **retired.** Roland persona file F4 row reworded so the LOW-confidence DISABLE rule still applies (to the simple-click Accept on recommendation cards) without referencing the descoped Mark-executed gesture.
- **Glove-gesture acceptance criteria walkthrough** (ECM 4 Technician + ÖBB winter glove + ≥20 sessions across two lighting conditions) was a pre-MVP-pilot gate. Re-scoped as a **pre-Phase-2 gate** if Phase 2 readiness verdict includes promoting the sign-off to authoritative.
- **Where applied:** Roland persona (W9 retired, F4 reworded, design-implications table updated), trigger map (driver list + Klaus-Roland tension paragraph), feature impact analysis (Tier 1 capability row reworded), product brief (Live Board MTTR line had been updated; ECM PWA description unchanged from this round), PRD §UX Constraints section (full restructure to Phase 2 candidate), PRD FR45 + FR56 (simple-click language; deliberate-confirmation flagged as Phase 2 candidate), PRD §3 product scope ECM PWA description, PRD permission table for ECM Manager, PRD Capabilities Revealed footers in Journey 1, PRD Journey 1 Klaus moment ("3-second hold...5-second countdown" → "single click in MVP"), PRD NFR Accessibility step-ceiling row.

**Reversibility:** Both corrections are reversible — the hold-to-record spec is preserved in the design-system reference and can be re-promoted to MVP if needed. The MTTR-sourcing change is a one-way door at the data-layer claim (the harness now owns the metric) but reversible via Stadler-MTTR-read FR addition.

---

### 2026-05-25 — Persona pseudonym retired: "Stefan" → "Roland"

- **Context:** Throughout the strategic and design-system work, the Fleet Manager persona was pseudonymous as "Stefan" — Roland Ruisz's real name was used only in the persona file's `real-world equivalent` field. Operator decision (2026-05-25): retire the pseudonym, use Roland's real first name directly across the strategic docs.
- **Scope of rename:** Product brief, PRD, trigger map, feature impact analysis, Roland's persona file (formerly `01-disponent-stefan.md`), this design log going forward. **Out of scope for this pass:** scenario file contents (1, 2, 9, 10) — those scenarios are queued for harness-model rewrites anyway and the rename happens then. Scenario folder names + filenames (`01-stefan-crisis-redeployment/`, `01-disponent-stefan.md`) untouched until a deliberate sweep — too much cross-reference breakage risk to combine with content updates.
- **Historical design-log entries (entries written before 2026-05-25 today): "Stefan" retained as the contemporaneous pseudonym** — log is a chronological trail; rewriting history destroys traceability. Today's reframe + recompute entries refer to Roland directly.
- **Why:** Pseudonyms made sense when the persona doc was the only place his name lived; now that all four strategic docs use his work context heavily, the pseudonym distance is more confusing than protective. Operational reality is one person (Roland) — calling him Roland is clearer.
- **Reversibility:** Reversible at any moment via the same find/replace. Roland is also Roland-the-real-person in the design log + memory; ambiguity is low (when a doc says "validate with Roland Ruisz" it always meant Roland).

---

### 2026-05-25 — Strategic recompute: Trigger Map + Feature Impact Analysis (harness-model alignment)

- **Context:** Strategic-layer documents needed alignment with the harness reframe + party-mode locks + operator corrections. Trigger Map and Feature Impact Analysis are downstream of the rewritten product brief + PRD.
- **Trigger Map changes:**
  - **Vision rewritten** around harness-model framing (BYOK + constitution-as-contract + Provenance Replay + three views)
  - **Business objectives recomputed** — six objectives now: per-view acceptance, ECM audit credibility, Provenance Replay integrity (new), confidence + mapping floor (new), seven-criteria Phase 2 readiness (was five), fundable event with harness positioning as wedge
  - **Objective 4 retired** — vendor warranty leverage as commercial wedge (operator correction post-party-mode; warranty descoped from MVP)
  - **Lukas persona retired** as separate persona; drivers merged into Stefan (one role, two programme hats)
  - **Stefan's driver list expanded** to absorb harness USP responsibilities: Daily Brief skill, Live Board multi-subsystem visibility, Kondukt skill registry, cross-subsystem correlation, MTTR-from-Stadler, BYOK token budget concern, LOW-confidence calibration, train-mapping integrity
  - **Renate's driver list expanded** to include Replay CLI demonstration + suppression log inspection + constitution-as-theatre fear
  - **Mermaid diagram regenerated** — removed Lukas node, added Provenance Replay + confidence/mapping objective nodes
  - **Design Focus Statement updated** — four signals now (per-view acceptance + Replay integrity + shadow-match + Renate sign-off)
- **Feature Impact Analysis changes:**
  - **Persona multiplier table** — Lukas-2 dissolves into Stefan-3
  - **Tier 1 expanded** — Constitution-as-contract enforcement, Recommendation Provenance Replay archive, Live Board, Kanban-with-progressive-disclosure, Kondukt-with-skill-registry, Confidence visual grammar with LOW-DISABLE, BYOK key vault, Train-mapping integrity verifier, Allowlisted egress proxy added as Tier 1 must-ship
  - **Tier 2 expanded** — Cross-subsystem correlation, MTTR-from-Stadler, Live-Board-to-Kanban redirect, Kondukt skill registry inspectability, Suppression log inspection, Phase 2 Readiness dashboard (now 7-criteria)
  - **Tier 4 expanded** — Warranty recovery workflow, OTA management, Drift Visibility as daily view, Probebetrieb workflow as dedicated surface, Regeneration loop, ÖBB SSO IdP all retired from MVP
  - **Risk watchlist updated** — validator over-strict, validator-vs-runtime drift, BYOK provider-agnostic guarantee, BYOK token exhaustion, train-mapping divergence, LOW-confidence rate >15% trip-wire
  - **Persona Coverage Matrix recomputed** — Lukas column removed; Renate's coverage expanded to include Replay CLI + Suppression log inspection
  - **Design Sequencing recommendation aligned** with PRD §15 Sprint Scope (7-day Sprint 1, 5-day Sprint 2 + policy-day)
- **What this completes:** The strategic-layer (brief + PRD + trigger map + FIA) is now fully consistent with the harness model. Next: persona revision (task #7), then scenario authoring.
- **Reversibility:** Trigger Map and FIA are reversible documents (recomputed from PRD + persona files). Strategic locks are not in these docs — they're in the product brief and the PRD.

---

### 2026-05-25 — OPERATOR CORRECTION on party-mode crystallisations (3 of 4 reverted or modified)

- **Context:** Party mode produced 4 crystallisations (entry below). On review, Abbas rejected 3 of the 4 as either operator-overrideable or premature for pre-pilot. Revisions applied to brief same day.
- **What was kept from party mode:**
  - **Winston's Recommendation Provenance Replay** as the headline pilot test artifact — load-bearing, retained intact
  - **Constitution-as-contract** (schemas + provenance validators, regeneration loop deferred) — retained
  - **The 1:1 reconciliation between Provenance Replay archive and shadow audit trail** — retained
- **What was overridden by operator:**
  - **5 → 2 view collapse REJECTED.** Live Board and Chat+Canvas are the USP, not optional surfaces. The party-mode demotion treated them as decorative; the operator named them as constitutive of the product. **Final MVP scope: 4 views** (Daily Brief + Neglect-Risk Kanban + Live Board + Chat+Canvas). Drift Visibility stays moved to ECM module (the one party-mode demotion that holds — quarterly cadence is not daily-harness material).
  - **Euro-denominated scorecard REVERTED to adoption-dominated.** Mary's prescription was right in spirit but premature: we don't have baseline euro figures yet. Committing to euro thresholds pre-pilot would be making numbers up. The pilot is the *instrument* that produces those numbers. Pre-pilot, the honest scorecard is acceptance rate ≥70% (per-hat, not aggregated, per Saga's caution) + qualitative trust signals + the five composite trust criteria.
  - **Capability admission paragraph REMOVED.** Victor's "warranty cut for capacity reasons" framing is internally honest but not buyer-facing material. Logged in design log; out of the brief. Brief no longer telegraphs founding-team capacity limits.
  - **Renewal-conversation script REWRITTEN.** Mary's three-yes-with-euros version required figures we don't have. Replaced with "sign-offs as gate, trust criteria as texture": 3/3 sign-offs (Roland + Renate + outside-auditor tabletop verdict on the Provenance Replay archive) = extend; anything less = don't.
- **Why this matters:** Party mode was useful — Winston's contract architecture is now load-bearing — but three of the four crystallisations conflated *agent confidence* with *operator-validated truth*. The operator's product knowledge overrode the agents' strategic reasoning on USP composition (Live Board + Chat+Canvas), pre-pilot evidence honesty (euro figures), and buyer-facing posture (capability admission). This is the right outcome of the convergence-stress-test pattern — agents flag risks; operator decides which to act on.
- **Reversibility:** All three operator-correction reversions are reversible at brief level until PRD writes architecture. Provenance Replay + contract enforcement remain one-way doors at the engineering layer (per Winston's pre-mortem).
- **Revisit triggers:**
  - If Live Board / Chat+Canvas adoption is <30% per-hat at manday 30, revisit whether they earned MVP scope
  - If the four-view scope generates a measurement-aggregation problem at the pilot scorecard (Saga's caution lands), revisit per-hat tracking discipline
  - If pre-pilot baseline conversations with Roland produce euro figures that close the "we don't have numbers" gap, the scorecard can re-denominate to a hybrid (adoption + euros) before manday 1

---

### 2026-05-25 — PARTY MODE on harness brief (3 rounds → 4 crystallised brief revisions)

- **Context:** After the harness-model reframe (above), per Abbas's stated preference (memory entry: party-mode + pre-mortem is the preferred stress-test for already-locked decisions before one-way-door commitment, ESPECIALLY when the lock came from convergence), opened a multi-agent party-mode pass on the rewritten brief.
- **Roster:** Saga (WDS analyst), John (PM), Winston (architect), Mary (analyst), Victor (innovation strategist). Strategic-first composition to catch positioning gaps, MVP scope risks, architecture viability, persona/driver misses.
- **Round 1 (independent first impressions):**
  - **Saga:** Persona collapse was role-name convergence, not JTBD convergence. Roland's three hats (Disponent / Telematik / Teil-PL) have three different time horizons and tool ergonomics. Failure mode: harness tunes for one hat, others migrate back to Excel + email, pilot reads as success while real work is invisible.
  - **John:** "Recommendation" is undefined; five views imply three different products (synthesis tool / triage signal / drafting tool). Failure mode: 73% acceptance, "ja, nett, aber ich treffe die gleichen Entscheidungen wie vorher." Engagement-without-displacement. Prescription: name the first-kill decision in one sentence.
  - **Winston:** Harness has no defined behavioral contract with the BYOK LLM. Constitution as PROMPT (cheap, fragile) vs CONTRACT (per-skill schemas + provenance validators + regeneration loop, real engineering weeks not budgeted). Brief sells the BYOK commercial pitch and is silent on the safety pitch (harness owns correctness, not the LLM). The contract is the spine — needs spec before personas, before scenarios.
  - **Mary:** Reframe moved from outcome-denominated value (delay-minutes, warranty euros) to architecture-denominated value (BYOK, constitution, MCP). Category mistake — buyers don't have a budget line for "agentic OS." Failure mode at renewal: Roland says "I can't justify a second 6 months — I'm not sure what problem you solve that my existing process doesn't already solve well enough." Prescription: produce a Roland sentence of form "I would pay €X for Y because I lose €Z to its absence."
  - **Victor:** "Why can't ÖBB build this themselves in six months with their own engineers and an Anthropic contract?" The previous wedge (warranty leverage) was a BUSINESS asset, not a software asset. BYOK is anti-moat. Constitution+skills+MCP = commodity, assemblable by a competent team in a quarter. Reframe moves UP the disruption curve toward incumbents. The wedge has been deleted and not replaced.
- **Round 2 (cross-examination — Mary↔Victor, Saga↔John, Winston cross-examined by all four):**
  - **Mary + Victor triangulated, not converged.** Same wound from supply vs demand sides — Mary on buyer P&L, Victor on build-vs-buy defensibility. Victor's chess move: warranty was cut as "capability admission, not strategic pivot" — restoring warranty wedge would solve both. If team cannot stand up warranty wedge in MVP for capability reasons, brief should call itself "foothold consulting engagement that hopes to compound into a product" — honest posture.
  - **Saga + John converged hard on the SAME wound.** Both: "the brief refuses to pick." Joint prescription: MVP = Daily Brief (Disponent hat) + reframed Kanban (neglect-risk triage, NOT queue order). Chat+Canvas = Phase 1.5 (Teil-PL hat). Live Board = backlog ("TV in the breakroom"). Drift = move to ECM module. Five views collapse to two.
  - **Winston's contract argument unified all four.** Saga: contract IS the persona work. John: contract IS the operational definition of "recommendation." Mary: § 1313a ABGB Erfüllungsgehilfe means founder is responsible regardless of LLM supplier — no contract, no wedge. Victor: contract-enforcement layer IS the missing moat. All four asked Winston: what is the single pilot test that proves the contract holds?
- **Round 3 (Winston solo answers the unified question):**
  - **The Recommendation Provenance Replay** — every recommendation stored as triplet at emission (Envelope + Provenance Bundle + Validation Receipt). Auditor-runnable CLI replays any recommendation 6 months later from artifacts alone, no LLM session needed. Aggregate pass: ≥99% replay-verify cleanly, zero recommendations without triplet, advisory-only linter zero breaches.
  - **Collapse 3 components into 2.** Schemas + provenance validators are inseparable (treated as one engineering item). Regeneration loop is the deferrable one. MVP failure mode = suppression, not regeneration. "Silence is safe. The Disponent does not get a recommendation; they fall back to current practice. You lose availability, not safety."
  - Boring stack: Postgres append-only Recommendation table + content-addressed Provenance blob store + pure-function Validator module + Replay CLI. ~2000 LOC. Operable by one person.
- **Four crystallised brief revisions (applied 2026-05-25, post-party-mode):**
  1. **Five views → two.** MVP = Daily Brief + Neglect-Risk Kanban (Disponent hat only). Chat+Canvas deferred. Live Board + Drift Visibility out.
  2. **Recommendation Provenance Replay** as headline pilot test artifact + auditor-runnable CLI + reconciliation with shadow audit trail.
  3. **Pilot scorecard re-denominated in euros, not adoption.** Four buyer-currency numbers (delay-minutes attributable / warranty-recoverable detections / ECM audit hours saved / dispatch-decision concurrence). Acceptance rate retained as sanity check, NOT pass criterion. Renewal-conversation script pre-committed.
  4. **Capability admission named honestly.** Warranty recovery workflow descoped partly for capacity reasons (no indemnity capital / § 1313a opinion / insurance backing). Warranty DETECTION restored as pilot-scorecard line item. Brief refuses the "Software 3.0 agentic OS" slogan as moat substitute; commits to "harness narrowly tuned to Disponent hat first-kill, with provenance archive as commercial asset."
- **Reversibility:** All four revisions are reversible at brief level until PRD writes the architecture statement. Once the Recommendation Provenance Replay spec lands in code, it's a one-way door at the engineering layer (one of Winston's pre-mortem warnings).
- **Revisit triggers:**
  - If pilot scorecard's 4 euro numbers are all <10% of their hypothesised baselines at manday 30, revisit the harness-model reframe itself
  - If Roland accepts the "Disponent-hat-only MVP" narrowing without pushback, validates the persona+JTBD compression
  - If Renate Fischer's tabletop-test reaction to the Recommendation Provenance Replay is "this is what I would have asked for," validates Winston's contract spec
  - If any pilot LLM hallucination escapes the validators, the entire contract layer is suspect — full architectural review

---

### 2026-05-25 — HARNESS-MODEL REFRAME (one-way door)

- **Context:** During WDS Phase 4 spec-writing for scenarios 06–10, Abbas surfaced that the product is fundamentally mis-framed. Existing strategic docs treated the platform as "three surfaces (Fleet Manager AI Interface + Depot PWA + Telematics Control Board) with Kondukt as an autonomous RAO-producing agent." The actual product Abbas is building is **a harness for a customer-provided LLM** — constitution + skills + MCP server access + tools + allowlisted internet egress — exposing landside fleet telemetry. The customer brings the API key (BYOK); the harness brings the scaffolding. MVP is recommendation-only; no autonomous action.

- **Five corrections triggered by this reframe:**
  1. **Role merge:** "Disponent" (Stefan) and "Teil-Projektleitung Telematik" (Lukas) are the same role, not two roles. Roland Ruisz is one person with one role (Flottenmanagement / System- und Betriebstechnik / Team Telematik) and two programme hats (Cityjet DOSTO Neu + Cityjet Enzo). Lukas-as-separate-persona retired.
  2. **Surface collapse:** Three-surfaces framing → one harness with five views (Daily Brief / Kanban Triage / Live Board / Chat+Canvas / Drift Visibility). Depot PWA remains as a recommendation-consumer surface; Telematics-as-separate-product retired.
  3. **MVP descope:** Vendor warranty dispute (was scenario 06), OTA rollout management (was scenario 07), HAFAS validation against onboard GPS — all cut from MVP. Replaced by drift visibility (much smaller surface).
  4. **TCMS data path corrected:** TCMS lives on onboard VLAN 7 and is polled via SNMP by the brain, then forwarded landside. Landside sees TCMS through the brain, not through Stadler's RDC.
  5. **Kondukt brand reframed:** Kondukt is the LLM-persona identity any customer-provided LLM adopts inside the harness (constitution-defined). One brand, model-agnostic. Customer can swap LLM providers without losing the Kondukt voice.

- **Architecture decisions locked:**
  - **BYOK** — operator brings their LLM API key; harness never stores plaintext keys; per-user scope; never logged
  - **Allowlisted egress** — LLM internet access is restricted to a curated approved-domains list (vendor portals, standards bodies, EU regulatory texts). NIS2-defensible. No open web.
  - **Audit scope** — recommendations + operator actions only. Daily briefs and chat conversations are *working artifacts*, not audit records. Smallest defensible audit surface.

- **Personas:**
  - One primary: Fleet Manager (Stefan = pseudonym for Roland Ruisz)
  - Two recommendation-consumer roles: ECM Manager, ECM 4 Technician
  - Renate Fischer (ECM Lead) retained for audit-export sign-off
  - Retired: Teil-Projektleitung Telematik as a separate persona

- **Reversibility:** **ONE-WAY DOOR** on multiple axes:
  - Product positioning (harness vs. embedded-agent SaaS) — locked at this brief revision
  - MVP scope cuts (warranty, OTA, HAFAS) — reversal means adding scope back, which inflates the pilot
  - Persona merge — reversal means re-authoring the retired Lukas persona
  - Allowlisted egress — opening it later is reversible; closing an open egress after operators have built habits around it is not

- **Revisit triggers:**
  - **Architectural:** if BYOK proves operationally hostile during pilot (key management friction blocks Roland's first-week use), revisit the harness-side default-model option
  - **Scope:** if warranty/OTA descope blocks Phase 2 commercial story at manday 15 ROI conversation (scenario 08), reassess vision-tier capability promotion
  - **Persona:** if a second ÖBB operator outside Roland's team uses the harness during pilot and reveals genuinely different work patterns (not just different programme hats), reopen persona split

- **Scope of rewrite triggered:**
  - `A-Product-Brief/project-brief.md` — REWRITTEN 2026-05-25 (this entry)
  - `prd.md` — pending rewrite (architecture statement + FRs + MVP scope)
  - `B-Trigger-Map/trigger-map.md` — pending recompute (drivers around harness model)
  - `B-Trigger-Map/feature-impact-analysis.md` — pending recompute (Lukas-2 multiplier dissolves)
  - `B-Trigger-Map/personas/01-disponent-stefan.md` — pending revision
  - `B-Trigger-Map/personas/04-telematik-lukas.md` — pending archive
  - `C-UX-Scenarios/00-ux-scenarios.md` — pending revision (remove dual-hat pseudonymous framing; new scenarios list)
  - `C-UX-Scenarios/06-lukas-vendor-warranty-dispute/` — pending retirement (work descoped)
  - `C-UX-Scenarios/07-lukas-ota-canary-drift/` — pending retirement (work descoped)
  - New scenarios to author: Daily Brief / Kanban Triage / Chat+Canvas / Drift Visibility / BYOK Setup / Live Board
  - `D-Design-System/00-design-system.md` — pending update (Kondukt attribution + confidence on harness artifacts)

- **Cited sources for the reframe:**
  - Roland Ruisz email signature → real-world title (Flottenmanagement / Team Telematik / Teil-Projektleitung Telematik for DOSTO Neu + Enzo)
  - `dosto-troubleshooting/train-ip-allocation-commission/4736-xxx/4736-101/4736-001_IP-Allocation.pdf` (Stadler/Nomad-issued, TLP Amber) → subsystem taxonomy + VLAN layout
  - Karpathy "Software 3.0 / LLMs as new OS" framing → positioning
  - Existing memory entries (advisory-only MVP, Phase 2 trust criteria, party-mode LOW-DISABLE lock) — all retained

---

## Open Backlog Items (added 2026-05-25)

- ~~**Hermes rename decision**~~ — RESOLVED 2026-05-25: renamed to Kondukt. See decisions log above.
- **Reference reading list for Phase 4 implementation:**
  - `outsourc-e/hermes-workspace` — `src/screens/chat/` (SSE + tool-call rendering), `src/screens/mcp/` (MCP catalog), `src/screens/vt-capital/` (domain-state-with-guardian pattern for Lukas's Zone 1/2/3)
  - `NousResearch/hermes-agent` — `mcp_serve.py` and MCP client code (Rail MCP Server integration reference), SECURITY.md (structural template for ours)
