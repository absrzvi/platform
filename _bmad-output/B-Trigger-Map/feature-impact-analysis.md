# Feature Impact Analysis — ÖBB Landside Platform

> Maps platform capabilities to persona drivers, ranks by impact, surfaces dependencies and risks.

**Created:** 2026-05-25
**Author:** Abbas Rizvi
**Source:** `trigger-map.md` + persona files + `prd.md`

---

## Scoring Model

For each capability, score against the drivers it addresses. Score = (sum of driver weights addressed) × (priority multiplier of personas served).

- **Driver weight:** WANT = 1, FEAR addressed = 1.5 (avoiding harm scores higher than enabling good)
- **Persona multiplier:** Stefan (Disponent) = 3, Klaus (ECM 1) = 3, Renate (ECM Lead) = 3, Lukas (Telematik) = 2, Elena (ECM 4) = 1
- **Confidence:** H/M/L — how sure we are the capability addresses the driver as scoped today

This is a heuristic ranking for design prioritization, not a roadmap forcing function.

---

## Tier 1 — Must Ship in MVP (Phase 2 trust criteria depend on these)

| Capability | Personas served | Drivers addressed | Score | Conf | Notes |
|---|---|---|---|---|---|
| **Kondukt Recommended Action Objects (RAOs)** — structured multi-system action proposals as readable cards | Stefan (3) + Lukas (2) | Stefan W2, W3, F1, F3; Lukas W1, F2 | **22.5** | H | The single capability that defines the platform vs competitor dashboards. Confidence visual grammar is non-optional. |
| **ECM shadow audit trail** — append-only, IdP-bound, HMAC-signed, hash-chained | Klaus (3) + Renate (3) | Klaus W2, W4, F1, F3; Renate W1, W2, F1, F2 | **24** | H | The Phase 2 trust criterion that gates commercial ascension. The export *is* Renate's review surface. |
| **Pre-arrival depot brief, role-filtered** — generated 14 min before HAFAS scheduled arrival | Klaus (3) + Elena (1) | Klaus W1, W3, F4; Elena W1, W4 | **13** | H | >80% pre-arrival review rate target. Role filter governed by ECM-Manager-signed-off `role_relevance[]` taxonomy. |
| **Cost-impact one-liner on every triage card** | Stefan (3) | Stefan W1, F3 | **7.5** | H | The cost-visible-before-confirming contract. Card-level, not drill-down. |
| **3-second hold → 5-second countdown → atomic write gesture** | Klaus (3) + Stefan (3) | Klaus W2, W3; Stefan W3, F1 | **15** | H | The trust-building muscle memory. Identical gesture across MVP shadow and Phase 2 authoritative. |
| **Recommendation→action→outcome chain capture** — generated_at, displayed, user_action, action_logged, outcome_observed | Stefan (3) + Renate (3) | Stefan F1, F2; Renate W1, W4 | **15** | H | The dataset Phase 2 trust evaluation runs on. |
| **`supersedes_id` correction model** — append-only correction, no mutation | Klaus (3) + Renate (3) | Klaus F3; Renate W3, F1 | **12** | H | Renate's rebuttal-ready answer to "what if someone signs off wrong?" |
| **Reconciliation queue + 24h divergence flagging** | Klaus (3) + Renate (3) | Klaus W4, F3; Renate W4, F3 | **18** | H | The advisory-only-era artefact that didn't exist when paper was alone. |
| **IdP-bound identity per audit row** (ÖBB SSO federation) | Klaus (3) + Renate (3) + Elena (1) | Klaus F1; Renate W2, F2; Elena F2 | **15** | H | Pre-pilot blocker. Federation by manday 1. |

---

## Tier 2 — High-value MVP (acceptance rate + persona adoption depend on these)

| Capability | Personas served | Drivers addressed | Score | Conf | Notes |
|---|---|---|---|---|---|
| **Kondukt confidence visual grammar (three-tier)** | Stefan (3) + Lukas (2) | Stefan F4; Lukas F2 | **7.5** | H | Phantom-recommendation mitigation. Low-confidence RAOs visibly distinct, vendor-delivery RAOs require explicit override. |
| **Crisis entry state (full-width chat, pre-seeded)** | Stefan (3) | Stefan W2, W4 | **6** | M | Transition gesture from 70/30 triage to full-width crisis is the Phase 3 scenario gap. |
| **One-tap ACK + DEFER (with reason) for ECM 4** | Elena (1) + Klaus (3) | Elena W2, W3, F3; Klaus W1 | **8.5** | H | Without Elena's log loop, Klaus's brief is empty. |
| **Role selector — shared device vs personal phone** | Elena (1) | Elena F1, F2 | **3** | H | Adoption-or-die for ECM 4. Shared device = role every open + IdP per ACK. Personal phone = stored role. |
| **Three-zone Telematics control panel (Zone 1/2/3 interaction binding)** | Lukas (2) | Lukas W1, W2 | **4** | H | Zone selection → topology → RAO. Locked in PRD FR36–FR50. |
| **Cross-domain topology drawer** (onboard memory + landside latency in one view) | Lukas (2) | Lukas W1, W2, F4 | **5** | H | The structural moat vs cloud-only competitors. |
| **Vendor warranty evidence packet PDF (Kondukt-generated)** | Lukas (2) | Lukas W1, F1, F2 | **5** | M | Probebetrieb-overlap-gated. Vendor-defensible: explicit timezones, causal chain, evidentiary trail. |
| **Pilot evidence dashboard** — composite Phase 1 score + Phase 2 trust composite + threshold lines | Stefan (3) + Renate (3) + Lukas (2) | Stefan W4; Renate W4; Lukas W2 | **8** | H | The Roland + Martin + Renate ROI artefact (PRD Journey 5). |
| **Shift handover summary** | Stefan (3) | Stefan F2 | **4.5** | M | Handover-complete definition is a Phase 3 scenario open question. |

---

## Tier 3 — MVP but optional during pilot (capability ships, evidence may not)

| Capability | Personas served | Drivers addressed | Score | Conf | Notes |
|---|---|---|---|---|---|
| **OTA rollout view — canary + drift detection** | Lukas (2) | Lukas W3 | **2** | M | Ships in MVP. Evidenced during pilot iff OTA cycle overlaps pilot window. |
| **Multi-tag `role_relevance` for cross-domain faults** | Elena (1) | Elena F4 | **1.5** | M | Open decision Q1 from `ux-decisions-2026-05-23.md` — sort order per role + cross-domain tagging both need ECM Manager sign-off pre-pilot. |
| **Vendor portal deep-link from topology drawer** | Lukas (2) | Lukas F4 | **3** | L | Vendor-portal-by-vendor coverage. Best-effort, opportunistic. |
| **Modification UX for RAOs** (capture original + modification) | Stefan (3) | Stefan W2 (modification component) | **3** | M | Modification rate is a trust signal in its own right; UX must preserve the proposal alongside the modification. |
| **Pilot evidence dashboard — personal-impact view ("decisions you made this shift")** | Stefan (3) | Stefan W4 | **3** | M | Identity, not aggregate. Stefan-facing, not Roland-facing. |

---

## Tier 4 — Vision / Phase 2+ (NOT in MVP scope; must not be pre-empted)

| Capability | Reason it's not MVP |
|---|---|
| **Platform-mediated dispatch** | Gated on five composite trust criteria at pilot end. Designing the surface today must not preclude or pre-empt. |
| **Authoritative ECM audit trail** | Same — MVP shadow becomes Phase 2 authoritative iff trust criteria met. Gesture and schema are identical to support transition. |
| **Lastenheft procurement export from cumulative evidence** | Vision-tier. Format, consumer, failure threshold to be defined post-pilot. |
| **Kondukt proactive fleet-pattern push (no query)** | Vision-tier. |
| **ECM audit integration with ÖBB internal compliance system** | Phase 2 integration target. |
| **Multi-operator deployment** | Per-operator architectural assumptions need spike before fleet rollout. |
| **SAP PM automatic G-Check work-order creation from RAOs** | Phase 2 integration; MVP is structured export for manual entry. |
| **Dismissal signal arbitration (conductor dismissals from brain → Fleet Manager threshold tuning)** | Post-MVP feedback loop. |

---

## Risk-Weighted Watchlist

These are capabilities where impact is high but confidence or scope is fragile.

| Risk | Capability affected | Mitigation |
|---|---|---|
| **Probebetrieb cycle does not overlap pilot window** | Vendor warranty evidence packet (Tier 2), OTA rollout view (Tier 3) | Pre-pilot gate with Lukas's programme team. If no overlap, capabilities ship but FR41 is unevidenced in pilot. Telematics surfaces still ship for trial-run validation + cross-fleet monitoring. |
| **Phantom recommendation in early pilot mandays** | All Tier 1 RAO capabilities | Kondukt spikes (4) pre-story. Confidence visual grammar enforced. Tabletop exercise (manday 20–25) explicitly tests phantom scenarios. |
| **`role_relevance` taxonomy not signed off by ECM Manager pre-pilot** | Pre-arrival depot brief, ACK/DEFER loop | Open decision Q1 in `ux-decisions-2026-05-23.md`. Pre-pilot blocker — must close before MVP UX scenarios lock. |
| **Kondukt BYOK API instability (429/5xx) during pilot** | All RAO surfaces | Spike 4. Triage and crisis views remain fully functional without Kondukt. UI labels degradation explicitly. |
| **Shadow audit ↔ paper divergence rate >2/week** | ECM trust composite | Reconciliation queue + 24h flagging. Persistent divergence downgrades Phase 2 readiness assessment. |
| **Renate's audit committee rejects export format** | Phase 2 readiness composite | Structured JSON export PoC-acceptable; named target schema (BOOM or successor) in Phase 2 commercial proposal. |
| **Modification UX captures modification but loses original** | Stefan acceptance rate signal | Schema-level decision pre-Fleet-Manager stories: every RAO row preserves the original alongside any user modification. |

---

## Capability → Persona Coverage Matrix

| Capability | Stefan | Klaus | Elena | Lukas | Renate |
|---|---|---|---|---|---|
| RAOs | ●●● | — | — | ●●● | ● (read in audit) |
| Shadow audit trail | ● (write via Mark executed) | ●●● | ● (write via ACK) | — | ●●● (read) |
| Pre-arrival brief | — | ●●● | ●●● | — | — |
| Cost-impact one-liner | ●●● | — | — | — | — |
| Hold-to-record gesture | ●● | ●●● | — | — | ●● (verifies in audit) |
| Reconciliation queue | — | ●●● | — | — | ●●● |
| `supersedes_id` correction | — | ●●● | — | — | ●●● |
| IdP-bound identity | ● | ●● | ●●● | — | ●●● |
| Confidence visual grammar | ●●● | — | — | ●●● | — |
| Crisis entry state | ●●● | — | — | — | — |
| Three-zone Telematics panel | — | — | — | ●●● | — |
| Vendor warranty PDF | — | — | — | ●●● | — |
| Pilot evidence dashboard | ●● | — | — | ● | ●●● |
| OTA rollout view | — | — | — | ●●● | — |

`●●●` = primary surface | `●●` = significant interaction | `●` = secondary touchpoint | `—` = not in this persona's path

---

## Design Sequencing Recommendation

Based on persona-coverage × score × dependency:

1. **First sprint** — RAO + cost-impact one-liner + Hold-to-record gesture + IdP federation (the Stefan-Klaus-Renate-defining capability surfaces; everything else depends on these)
2. **Second sprint** — Shadow audit trail write + reconciliation queue + `supersedes_id` correction (Renate's review surface complete)
3. **Third sprint** — Pre-arrival brief + role-filtered card schema + ACK/DEFER loop (Klaus and Elena online)
4. **Fourth sprint** — Three-zone Telematics panel + topology drawer (Lukas online)
5. **Fifth sprint** — Pilot evidence dashboard + confidence visual grammar polish + crisis entry state transition (manday 15 readiness)
6. **Continuous** — Vendor warranty PDF, OTA rollout view, modification UX — opportunistic delivery once Tier 1 stable

---

_Generated with Whiteport Design Studio framework — Saga / Suggest mode_
