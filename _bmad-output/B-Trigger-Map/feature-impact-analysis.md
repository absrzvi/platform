# Feature Impact Analysis — ÖBB Landside Platform

> Maps harness capabilities to persona drivers, ranks by impact, surfaces dependencies and risks.

**Created:** 2026-05-25
**Revised:** 2026-05-25 (harness-model reframe — Lukas-2 multiplier dissolves into Roland-3; persona-coverage recomputed; capability list re-aligned to three views + Constitution-as-Contract + Provenance Replay)
**Source:** `trigger-map.md` (rewritten 2026-05-25) + persona files + `prd.md` (rewritten 2026-05-25)

---

## Scoring Model

For each capability, score against the drivers it addresses. Score = (sum of driver weights addressed) × (priority multiplier of personas served).

- **Driver weight:** WANT = 1, FEAR addressed = 1.5 (avoiding harm scores higher than enabling good)
- **Persona multiplier:** Roland (Fleet Manager) = 3, Klaus (ECM 1) = 3, Renate (ECM Lead) = 3, Elena (ECM 4) = 1
- **Confidence:** H/M/L — how sure we are the capability addresses the driver as scoped today

This is a heuristic ranking for design prioritization, not a roadmap forcing function.

**What changed in this revision:** Lukas-2 multiplier dissolves into Roland-3 — capabilities previously serving "Roland (3) + Lukas (2)" now serve "Roland (3)" only. Several capabilities (cross-subsystem visibility, Kondukt skill registry, multi-fleet-stage view) move from Tier 2 to Tier 1 because Roland-3 absorbs the harness USP responsibility.

---

## Tier 1 — Must Ship in MVP (Phase 2 trust criteria depend on these)

| Capability | Personas served | Drivers addressed | Score | Conf | Notes |
|---|---|---|---|---|---|
| **Constitution-as-contract enforcement** — per-skill schemas + provenance validators + advisory-language linter; failed validations suppress (no MVP regeneration) | Roland (3) + Renate (3) | Roland F1 (LLM-hallucination protection), F2 (reconstructability), F5 (LOW-confidence guardrail); Renate W2 (engineering rigour), F1 (compliance theatre), F5 (constitution-as-theatre fear) | **24** | H | The architectural spine. Without this, the harness is "wrapper around an LLM you don't control" — pilot cannot start. Locked from Winston party-mode 2026-05-25 + operator confirmation. |
| **Recommendation Provenance Replay archive** — triplet (Envelope + Provenance Bundle + Validation Receipt) on every emission + auditor-runnable CLI | Roland (3) + Renate (3) | Roland F2 (reconstructability); Renate W4 (Replay CLI demo on her laptop), F4 (Replay non-deterministic six months later) | **18** | H | The headline pilot test artifact. Renate runs the CLI live at manday ~15; outside auditor runs it at manday 20–25. Reconciles 1:1 with shadow audit trail. |
| **ECM shadow audit trail** — append-only, IdP-bound, HMAC-signed, hash-chained | Klaus (3) + Renate (3) | Klaus W2, W4, F1, F3; Renate W1, W2, F1, F2 | **24** | H | The Phase 2 trust criterion that gates commercial ascension. The export *is* Renate's review surface. |
| **Live Board — fleet-wide moving map + per-subsystem health, glanceable** | Roland (3) | Roland W3 (multi-subsystem visibility), W6 (cross-subsystem correlation surface), F3 (harness is THE tab, not another one) | **10.5** | H | The multi-subsystem visibility no competitor produces. Operator-friendly names; no VLAN IDs on Live Board. Click → redirects to Kanban detail. |
| **Kanban — neglect-risk lane-sort + train detail card with progressive disclosure** | Roland (3) | Roland W2 (decay-without-attention focus), W4 (depth on demand), F2 (reconstructability via detail card history) | **10.5** | H | The continuous all-day surface AND the depth surface. Train detail card carries VLAN/IP/MAC/firmware/switch port as primary; MTTR/recurring-failure/24h trace/vendor portal deep-link as secondary disclosure. |
| **Kondukt — chat + canvas + skill registry + free-form chat** | Roland (3) | Roland W1 (Daily Brief skill = first-kill in 4 min), W5 (vendor meeting prep + ad-hoc research), W7 (xlsx/docx artifact production), F3 (harness is THE tab) | **12** | H | The LLM-native deep-research + artifact-authoring surface. Skill registry MVP: Daily Brief / Recurring-failure pattern / Vendor meeting prep / Fleet health summary. Free-form chat as catch-all. |
| **Pre-arrival depot brief, role-filtered** — generated 14 min before HAFAS scheduled arrival | Klaus (3) + Elena (1) | Klaus W1, W3, F4; Elena W1, W4 | **13** | H | >80% pre-arrival review rate target. Role filter governed by ECM-Manager-signed-off `role_relevance[]` taxonomy. |
| **ECM shadow sign-off via simple click** (MVP) | Klaus (3) | Klaus W2, W3 | **7.5** | H | Deliberate-confirmation gestures (hold-to-record, Mark-executed) descoped from MVP 2026-05-25. Hold-to-record preserved in design-system reference as a Phase 2 candidate when sign-off becomes authoritative. |
| **Recommendation→action→outcome chain capture** — generated_at, displayed, user_action, action_logged, downstream-polling, outcome_observed | Roland (3) + Renate (3) | Roland F1, F2; Renate W1, W4 | **15** | H | The dataset Phase 2 trust evaluation runs on. Reconciles 1:1 with Provenance Replay archive. |
| **`supersedes_id` correction model** — append-only correction, no mutation | Klaus (3) + Renate (3) | Klaus F3; Renate W3, F1 | **12** | H | Renate's rebuttal-ready answer to "what if someone signs off wrong?" |
| **Reconciliation queue + 24h divergence flagging** | Klaus (3) + Renate (3) | Klaus W4, F3; Renate W4, F3 | **18** | H | The advisory-only-era artefact that didn't exist when paper was alone. |
| **IdP-bound identity per audit row + per recommendation interaction** (Keycloak in MVP, ÖBB SSO post-MVP) | Klaus (3) + Renate (3) + Elena (1) + Roland (3) | Klaus F1; Renate W2, F2; Elena F2; Roland F2 | **22.5** | H | Pre-pilot blocker. Federation by manday 1 (Keycloak; ÖBB IdP integration is post-MVP change request). |
| **Confidence visual grammar (three-tier)** — HIGH / MEDIUM / LOW + LOW-confidence DISABLE rule + modification_distance capture | Roland (3) + Renate (3) | Roland F4 (LOW-confidence rubber-stamping protection), F5 (rate must stay <15% sustained); Renate W2 (engineering rigour), F5 (constitution-as-theatre fear) | **15** | H | Party mode 2026-05-25 lock. Modify-on-LOW visibly differentiated from Modify-on-HIGH; cursor pre-placed in justification field on LOW. |
| **BYOK key vault** — encrypted at rest, per-user, never logged; token usage transparent to operator | Roland (3) + Renate (3) | Roland F4 (BYOK token exhaustion mid-shift detection); Renate W2 (engineering rigour), F2 (Nomad-controlled identity vs ÖBB-controlled keys) | **9** | H | One-way door on key handling. NIS2-defensible. |
| **Train-to-landside-DB mapping integrity verifier** — weekly reconciliation, zero misroutes | Roland (3) + Renate (3) | Roland F6 (mapping divergence destroys trust premise); Renate F4 (Replay archive only as trustworthy as the data layer beneath it) | **9** | H | Pilot-kill trigger if breached. One-way door at the data layer. |
| **Allowlisted egress proxy** — curated approved-domains list; every fetch logged | Roland (3) + Renate (3) | Roland W5 (vendor portal research within MVP scope); Renate W2 (NIS2 defensibility) | **9** | H | Spec'd in PRD §5; signed off by ÖBB IT Security pre-pilot. |

---

## Tier 2 — High-value MVP (per-view adoption + commercial wedge depend on these)

| Capability | Personas served | Drivers addressed | Score | Conf | Notes |
|---|---|---|---|---|---|
| **Cross-subsystem correlation annotations from Kondukt** — recurring-failure pattern detection across the fleet | Roland (3) | Roland W6 (intercom + CCTV + Wi-Fi drops on same train = network-path issue, not three failures); the harness USP claim 3 | **6** | H | Without this Live Board is a dashboard, not a co-pilot. |
| **MTTR computed by harness from landside-observable timestamps** (`t_open` = Stadler alarm-raised IntentPacket arrival, `t_close` = Stadler alarm-cleared IntentPacket arrival; `MTTR = t_close − t_open`); surfaced per component on Kanban detail | Roland (3) | Roland W4 (MTTR per component is the operational truth), W6 | **6** | H | The harness owns this metric. Stadler's internal MTTR field is NOT read. Reasons: (a) the landside-observed window is the operator's experienced MTTR, (b) brain→landside forwarding latency is included on both ends (honest number), (c) auditable by harness without trusting Stadler's internal accounting. |
| **One-tap ACK + DEFER (with reason) for ECM 4** | Elena (1) + Klaus (3) | Elena W2, W3, F3; Klaus W1 | **8.5** | H | Without Elena's log loop, Klaus's brief is empty. |
| **Role selector — shared device vs personal phone** | Elena (1) | Elena F1, F2 | **3** | H | Adoption-or-die for ECM 4. Shared device = role every open + IdP per ACK. Personal phone = stored role. |
| **Live Board → Kanban click-through redirect** (depth on demand) | Roland (3) | Roland W4 (depth surface in Kanban not Live Board), F3 (harness stays glanceable, not two-depth) | **4.5** | H | The architecture of Live Board's glanceability. |
| **Kondukt skill registry inspectability** — operator can view constitution + skill schemas + validator rules in-product | Roland (3) + Renate (3) | Roland F1 (hidden prompting check); Renate W2 (engineering rigour), F5 (constitution-theatre check) | **9** | H | No hidden prompting. FR22. |
| **Suppression log inspection surface** — Renate can see what the validator rejected and why | Renate (3) | Renate W2, F5 | **4.5** | H | "Proof that the harness rejected recommendations that didn't meet the constitution, not just emitted recommendations." |
| **Phase 2 Readiness dashboard** — seven-criteria composite + threshold lines | Roland (3) + Renate (3) | Roland W4 (visibility into Phase 2 trajectory); Renate W1, W4 | **9** | H | The Roland + Martin + Renate sign-off artefact (PRD Journey 5). Updated from prior 5-criteria to 7-criteria composite. |
| **Shift handover summary** — open Kanban cards by lane + in-flight Kondukt skill invocations + suppressed-emission events + unverified-outcome states | Roland (3) | Roland F2 (reconstructability across shift boundary) | **4.5** | M | Handover-complete definition is a Phase 3 scenario open question. |

---

## Tier 3 — MVP but optional during pilot (capability ships, evidence may not)

| Capability | Personas served | Drivers addressed | Score | Conf | Notes |
|---|---|---|---|---|---|
| **Multi-tag `role_relevance` for cross-domain faults** | Elena (1) | Elena F4 | **1.5** | M | Pre-pilot blocker — sort order per role + cross-domain tagging both need ECM Manager sign-off. |
| **Vendor portal deep-link from Kanban detail card** (secondary disclosure) | Roland (3) | Roland W5 (continuity with raw vendor portals) | **3** | L | Vendor-portal-by-vendor coverage. Best-effort, opportunistic. |
| **Modification UX for Kondukt recommendations** (capture original + modification distance) | Roland (3) | Roland W2 modification path | **3** | H | Modification rate is a trust signal in its own right; modification_distance captured per design-system lock. |
| **Per-hat engagement breakdown view** (Live Board / Kanban / Kondukt skills / Kondukt free-form chats) | Roland (3) | Roland W4 (his own pilot performance visibility) | **3** | M | Roland-facing, not Roland-facing; complements the Phase 2 Readiness dashboard. |
| **Cellular dead-zone overlay on Live Board** — disambiguates stale-data-due-to-geographic-blackout vs stale-data-due-to-hardware-fault | Roland (3) | Roland F6 (mapping integrity ≠ telemetry integrity; need to distinguish geographic from hardware causes) | **4.5** | M | FR7. Reduces false alarms on Live Board during cellular gaps. |

---

## Tier 4 — Vision / Phase 2+ (NOT in MVP scope; must not be pre-empted)

| Capability | Reason it's not MVP |
|---|---|
| **Platform-mediated dispatch** | Gated on seven composite trust criteria at pilot end. Designing the surface today must not preclude or pre-empt. |
| **Authoritative ECM audit trail** | Same — MVP shadow becomes Phase 2 authoritative iff trust criteria met. Gesture and schema are identical to support transition. |
| **Regeneration loop on validator failure** | Phase 1.5. MVP failure mode is suppression — "silence is safe." |
| **Vendor warranty recovery workflow** | Phase 1.5 if pilot evidence supports it; harness *detection* capability lives in Live Board + Kondukt's Vendor meeting prep skill in MVP. |
| **OTA rollout management** | Vendor tooling remains canonical; out of MVP. |
| **Drift Visibility as a daily-harness view** | Moved to ECM module (quarterly cadence — not daily harness material). |
| **Probebetrieb commissioning workflow** as a dedicated surface | Roland's commissioning hat is *one of* his contexts; harness supports the work via the three views, not via a separate surface. |
| **Multi-operator deployment** | Per-operator architectural assumptions need spike before fleet rollout. |
| **SAP PM automatic G-Check work-order creation from harness recommendations** | Phase 2 integration; MVP is structured export for manual entry. |
| **Dismissal signal arbitration (conductor dismissals from brain → harness threshold tuning)** | Post-MVP feedback loop. |
| **Lastenheft procurement export from cumulative evidence** | Vision-tier. Format, consumer, failure threshold to be defined post-pilot. |
| **Kondukt proactive fleet-pattern push (no query)** | Vision-tier; MVP is operator-invoked skills + free-form chat. |
| **ECM audit integration with ÖBB internal compliance system** | Phase 2 integration target. |
| **ÖBB SSO IdP integration** | Post-MVP change request — MVP uses Keycloak with manually-created users. |

---

## Risk-Weighted Watchlist

These are capabilities where impact is high but confidence or scope is fragile.

| Risk | Capability affected | Mitigation |
|---|---|---|
| **Constitution-as-contract validator over-strict (suppression rate too high)** | All Kondukt emission surfaces | Validator spike on first skill (Daily Brief) before pilot. Suppression rate >5% sustained = calibration review. Tabletop exercise (manday 20–25) tests the validator behaviour. |
| **Constitution-as-contract validator-vs-runtime drift** (validator passes at emission time but a six-months-later replay diverges) | Recommendation Provenance Replay archive integrity | Deterministic validator (no LLM in the validation loop); content-addressed blob store for Provenance Bundles; CLI tested against the archive periodically through pilot. Existential bug if breached. |
| **BYOK provider-agnostic guarantee not holding** (Anthropic vs OpenAI produce materially different validator pass rates) | Customer's LLM choice | Sprint 2 spike: same constitution + skills + validators tested against Anthropic + OpenAI before pilot. If divergent, lock to single provider for MVP and document. |
| **BYOK token exhaustion mid-shift** | All Kondukt skill invocations + free-form chat | Degraded-mode banner (FR20); Live Board + Kanban remain functional (telemetry-driven). Token usage transparency throughout the day so operator can see depletion coming. |
| **Train-to-landside-DB mapping divergence** | Provenance Replay archive premise; all telemetry-driven surfaces | Weekly automated reconciliation (FR41); pilot-kill trigger if breached. Halt harness data writes for the affected train pending reconciliation. |
| **Phantom recommendation in early pilot mandays** | All Kondukt recommendation surfaces | Constitution-as-contract enforcement layer is the architectural answer (FR23–FR27). Tabletop exercise (manday 20–25) explicitly tests phantom scenarios; Replay CLI confirms what the validator concluded at emission time. |
| **`role_relevance` taxonomy not signed off by ECM Manager pre-pilot** | Pre-arrival depot brief, ACK/DEFER loop | Pre-pilot blocker — must close before MVP UX scenarios lock. |
| **Shadow audit ↔ paper divergence rate >2/week** | ECM trust composite | Reconciliation queue + 24h flagging. Persistent divergence downgrades Phase 2 readiness assessment. |
| **Renate's audit committee rejects export format** | Phase 2 readiness composite | Structured JSON export + Replay archive export PoC-acceptable; named target schema (BOOM or successor) in Phase 2 commercial proposal. |
| **LOW-confidence rate exceeds 15% sustained over 72h** | Confidence visual grammar premise; per-view acceptance signal validity | Trip-wire monitored on harness ops dashboard; if breached, calibration review (memory: 2026-05-25 design-system lock). LOW-rate above 30% sustained = forced-Modify friction becomes theatre. |

---

## Capability → Persona Coverage Matrix

| Capability | Roland | Klaus | Elena | Renate |
|---|---|---|---|---|
| **Constitution-as-contract enforcement** | ●●● | — | — | ●●● |
| **Recommendation Provenance Replay archive** | ●● (read in audit) | ● (read in audit) | — | ●●● (CLI demo, archive integrity sign-off) |
| **Replay CLI** | — | — | — | ●●● |
| **Live Board** (moving map + per-subsystem health) | ●●● | — | — | — |
| **Kanban** (neglect-risk + detail card with progressive disclosure) | ●●● | — | — | — |
| **Kondukt** (skill registry + free-form chat + xlsx/docx export) | ●●● | — | — | — |
| **Cross-subsystem correlation annotations** | ●●● | — | — | — |
| **Daily Brief skill** | ●●● | — | — | — |
| **Vendor meeting prep skill** | ●●● | — | — | — |
| **Recurring-failure pattern skill** | ●●● | — | — | — |
| **MTTR from Stadler per component** | ●●● | ● (depot view) | — | — |
| **Shadow audit trail** | ● (write via recommendation actions) | ●●● | ● (write via ACK) | ●●● (read) |
| **Pre-arrival depot brief** | — | ●●● | ●●● | — |
| **ECM shadow sign-off (simple click, MVP)** | — | ●●● | — | ●● (verifies in audit) |
| **Reconciliation queue** | — | ●●● | — | ●●● |
| **`supersedes_id` correction** | — | ●●● | — | ●●● |
| **IdP-bound identity** | ●● | ●● | ●●● | ●●● |
| **Confidence visual grammar + LOW-DISABLE** | ●●● | — | — | ●● (audit signal) |
| **BYOK key vault** | ●●● | — | — | ●● (Nomad-vs-ÖBB sovereignty) |
| **Allowlisted egress proxy** | ●● (Kondukt research) | — | — | ●● (NIS2 defensibility) |
| **Train-mapping integrity verifier** | ●● (mapping divergence trigger) | — | — | ●● (Replay archive premise) |
| **Phase 2 Readiness dashboard** | ●● | — | — | ●●● |
| **Live Board → Kanban click-through** | ●●● | — | — | — |
| **Suppression log inspection** | — | — | — | ●●● |

`●●●` = primary surface | `●●` = significant interaction | `●` = secondary touchpoint | `—` = not in this persona's path

---

## Design Sequencing Recommendation

Based on persona-coverage × score × dependency, aligned with PRD §15 Sprint Scope:

1. **Sprint 1 (~7 eng-days)** — Constitution-as-contract validator (Daily Brief skill) + Recommendation Provenance Replay archive + Replay CLI prototype + Confidence visual grammar with LOW-DISABLE + Shadow audit trail with hash-chain + Brain schema-bump trip-wire + Train-mapping verifier + Recommendation outbox + BYOK key vault + Stale-data banner state. PWA wake-from-sleep gate.
2. **Sprint 2 (~5 eng-days + 1 policy-day)** — Allowlisted egress proxy + Structured IncidentReport endpoint (NIS2 + CRA) + BYOK provider-agnostic spike (Anthropic + OpenAI against same constitution) + SBOM + SECURITY.md + Phase 2 Readiness dashboard + Vulnerability handling process.
3. **Third sprint cluster** — Live Board (fleet-wide moving map + per-subsystem health + click-through redirect) + Kanban (neglect-risk lane-sort + detail card with progressive disclosure) — Roland's two glanceable views.
4. **Fourth sprint cluster** — Kondukt skill registry (Daily Brief + Recurring-failure + Vendor meeting prep + Fleet health) + free-form chat + xlsx/docx artifact builders. The harness USP surfaces.
5. **Fifth sprint cluster** — Pre-arrival depot brief + role-filtered card schema + ACK/DEFER loop (Klaus + Elena online).
6. **Sixth sprint cluster** — ECM audit-export surface for Renate + Replay CLI polish for manday-15 demo + reconciliation queue UI + Phase 2 Readiness dashboard polish.
7. **Continuous** — Cross-subsystem correlation refinement, MTTR-from-Stadler surface, modification UX, per-hat engagement breakdown — opportunistic delivery once Tier 1 stable.

**The five spike gates before storyworking begins** (per PRD §8):
- pgvector P99 <500ms / <250ms internal target (before Kondukt semantic-search stories)
- BYOK key handling + provider-agnostic constitution (before harness LLM-emission stories)
- Constitution-as-contract validator end-to-end on Daily Brief skill + Replay CLI prototype (before any harness view stories)
- Train-to-landside-DB mapping integrity (before harness goes live in pilot)
- Recommendation outcome reconciliation semantics (before recommendation→action chain stories)
- Allowlisted egress proxy + per-fetch logging (before Kondukt internet-research stories)
- CRA-readiness tabletop exercise (manday 20–25)

---

_Generated with Whiteport Design Studio framework — harness-model revision 2026-05-25_
