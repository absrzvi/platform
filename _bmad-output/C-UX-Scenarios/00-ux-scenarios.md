# UX Scenarios Index — ÖBB Landside Platform

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS) — Phase 3
**Mode:** Suggest (synthesized from Trigger Map + PRD journeys + UX decisions)
**Author:** Abbas Rizvi (drafted by Freya / WDS Designer)

---

## Overview

Ten scenarios, sequenced by Saga's sprint plan from the [feature impact analysis](../B-Trigger-Map/feature-impact-analysis.md). Each scenario is a **linear sunshine path** — no branches, no `if` statements — exposing the pages a developer can build with confidence.

Each scenario lives in its own folder (`NN-[scenario-slug]/`) with the outline file plus, in Phase 4, step-specific subfolders (`NN.1-[page-slug]/`, `NN.2-[page-slug]/`, …) carrying interaction and content specs.

---

## Sprint-aligned sequencing

| # | Scenario | Sprint | Persona | Surface |
|---|---|---|---|---|
| 01 | [Stefan's crisis redeployment](01-stefan-crisis-redeployment/01-stefan-crisis-redeployment.md) | 1 | Stefan | Fleet Manager AI Interface |
| 02 | [Stefan's cost-aware non-crisis triage](02-stefan-cost-aware-triage/02-stefan-cost-aware-triage.md) | 1 | Stefan | Fleet Manager AI Interface |
| 03 | [Klaus's pre-arrival brief → shadow sign-off](03-klaus-prearrival-shadow-signoff/03-klaus-prearrival-shadow-signoff.md) | 2 + 3 | Klaus | Depot Briefing PWA |
| 04 | [Elena's ACK + DEFER loop on shared device](04-elena-ack-defer-loop/04-elena-ack-defer-loop.md) | 3 | Elena | Depot Briefing PWA |
| 05 | [Renate's audit-export review at manday 15](05-renate-audit-export-review/05-renate-audit-export-review.md) | 2 | Renate | ECM Audit Export + Reconciliation |
| 06 | [Lukas's vendor warranty dispute](06-lukas-vendor-warranty-dispute/06-lukas-vendor-warranty-dispute.md) | 4 | Lukas | Telematics Control Board |
| 07 | [Lukas's OTA canary + drift monitoring](07-lukas-ota-canary-drift/07-lukas-ota-canary-drift.md) | 4 | Lukas | Telematics Control Board |
| 08 | [Roland + Martin's manday-15 ROI conversation](08-roland-martin-roi-conversation/08-roland-martin-roi-conversation.md) | 5 | Stefan (evidence) + Renate | Pilot Evidence Dashboard |
| 09 | [Stefan's shift handover](09-stefan-shift-handover/09-stefan-shift-handover.md) | 5 | Stefan | Fleet Manager AI Interface |
| 10 | [Kondukt degraded — Stefan keeps working](10-stefan-kondukt-degraded/10-stefan-kondukt-degraded.md) | 5 | Stefan | Fleet Manager AI Interface (degraded) |

> **Note on agent name:** The LLM co-pilot throughout these scenarios is **Kondukt** (renamed from "Hermes" on 2026-05-25 to avoid collision with Nous Research's hermes-agent — see [design log](../_progress/00-design-log.md)).

---

## Page Coverage Matrix

Every page from the Phase 3 inventory is exposed in at least one scenario. Pages flagged "future state" appear in 0 scenarios by design (Phase 2+ scope).

| Page / View | Scenarios | Note |
|---|---|---|
| Triage view (70/30 default) | 01, 02, 09, 10 | Stefan's default state |
| Crisis full-width | 01 | Disruption entry |
| Trend view (30/70) | 09 | Handover context |
| RAO card | 01, 02, 06 | Defining capability — three sprints |
| RAO execution log | 01, 02 | Shadow audit-trail surface for Stefan |
| Shift handover summary | 09 | Sprint 5 |
| Kondukt chat panel | 01 (pre-seeded), 02 (free-form), 10 (degraded) | Three states |
| Search results (pgvector) | 02 | Implicit; semantic search invoked from triage |
| Role selector — depot device | 03, 04 | Klaus (personal phone — bypass) + Elena (shared device — 4-tap) |
| Pre-arrival brief | 03, 04 | Same brief, two role views |
| Card detail | 03, 04 | Same template; role-filtered depth |
| ACK confirmation | 04 | Elena's one-tap |
| DEFER with reason | 04 | Elena's deferral |
| Hold-to-record gesture | 03 | Klaus shadow sign-off — trust muscle memory |
| Sign-off success / cancellation | 03 | Atomic write outcome |
| Zone 1 — fleet compatibility matrix | 06, 07 | Lukas macro |
| Zone 2 — SVG topology + drawer | 06 | Lukas micro |
| Zone 3 — Kondukt RAO console | 06 | Lukas co-pilot |
| OTA rollout view | 07 | Canary + drift |
| Warranty evidence packet export | 06 | Vendor-defensible PDF |
| Audit-trail export | 05, 08 | Renate review surface |
| Reconciliation queue | 05 | Divergence list |
| `supersedes_id` correction | 05 | Compliance correction model |
| Pilot evidence dashboard | 08 | Composite scores + threshold lines |
| Login / IdP federation entry | 01, 03, 04, 05, 06, 07, 08, 09, 10 | Pre-condition assumed for all scenarios |
| Error/degraded states | 10 | "Kondukt unavailable", "Data source unavailable" |
| Relay card | — | Future state — Dispatcher persona deferred |
| Lastenheft export workflow | — | Vision-tier — sketch only post-pilot |

---

## Driver Coverage Matrix

Every Trigger Map Must-Address driver is addressed by at least one scenario.

| Driver (from Trigger Map) | Persona | Scenario(s) |
|---|---|---|
| Cost consequence visible *before* confirming | Stefan | 01, 02 |
| Recommendation framed as proposal not command | Stefan | 01, 02 |
| Pre-arrival brief opens 14 min before HAFAS arrival, role-filtered | Klaus, Elena | 03, 04 |
| 3-second-hold gesture, identical MVP↔Phase 2 | Klaus, (Stefan in 01/02) | 03 (primary); 01, 02 (Stefan analog: "Mark executed") |
| IdP-bound identity, HMAC, hash chain, `supersedes_id` in audit export | Renate | 05 |
| Divergence between paper and shadow surfaced within 24h | Klaus, Renate | 05 |
| Crisis entry state (full-width chat, pre-seeded) | Stefan | 01 |
| Three-zone Telematics interaction binding | Lukas | 06 |
| Role persistence (session-only shared vs stored personal) | Elena, Klaus | 03, 04 |
| Confidence visual grammar (three-tier) | Stefan, Lukas | 01, 02, 06 |
| Modification UX captures original + modification | Stefan | 02 |
| Shift handover preserves open RAOs / in-flight decisions | Stefan | 09 |
| Recommendation→action→outcome chain capture | Stefan, Renate | 01, 02, 05, 08 |
| Kondukt degraded UX — triage + crisis remain functional | Stefan | 10 |
| Pilot evidence dashboard — Phase 1 + Phase 2 trust composite | Roland, Martin, Renate | 08 |

---

## Methodology Notes

- **Sunshine path only.** Every scenario is the success case. Error/degraded variants are *separate* scenarios (e.g., scenario 10) rather than `if` branches inside another scenario.
- **One transaction per scenario.** If a scenario has two transactions, it gets split.
- **Phase 4 produces page specs.** Each `NN.M-[page-slug]/` folder in Phase 4 will carry: interaction spec, content spec, functionality spec, state spec, error spec, and (where relevant) accessibility spec.
- **Personas in scenarios are pseudonymous.** Stefan, Klaus, Elena, Lukas are pseudonyms. Roland Ruisz wears Stefan's hat (Disponent) and Lukas's hat (Telematik PM) — same person, distinct contexts. Renate Fischer is the only named real-world person referenced in scenarios because her role surface is uniquely her own.

---

## Next Steps

1. **Review each scenario individually** for accuracy against your operational reality.
2. **Validate with Roland Ruisz** — scenarios 01, 02, 06, 07, 09 (he is Stefan + Lukas).
3. **Validate with Renate Fischer** — scenario 05 + audit-export surface in 08.
4. **Validate with an ECM Manager peer** — scenario 03 (Klaus archetype).
5. **Validate with an ECM 4 Technician peer (Electrical)** — scenario 04 (Elena archetype).
6. **Phase 4 entry** — Once scenarios are signed off, page specs proceed in Sprint 1 → 5 order.

---

_Generated with Whiteport Design Studio framework — Freya / Suggest mode_
