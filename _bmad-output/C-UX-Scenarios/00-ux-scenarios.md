# UX Scenarios Index — ÖBB Landside Platform

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Revised:** 2026-05-25 (harness-model reframe + scenario cleanup: 01/02/06/07 archived, 03/04/05/08/09/10 revised in place, 11+ are new harness-MVP scenarios)
**Method:** Whiteport Design Studio (WDS) — Phase 3 / 4
**Author:** Abbas Rizvi (drafted by Freya / WDS Designer)

---

## Overview

The harness model (locked 2026-05-25) produces three Fleet Manager views (Live Board / Kanban / Kondukt) + Depot Briefing PWA (ECM Manager + Technician recommendation-consumer surfaces) + ECM Audit Export (Renate review surface). Scenarios describe **linear sunshine paths** — no branches, no `if` statements — exposing the pages a developer can build with confidence.

Each scenario lives in its own folder (`NN-[scenario-slug]/`) with the outline file plus, where authored, step-specific subfolders (`NN.1-[page-slug]/`, `NN.2-[page-slug]/`, …) carrying interaction + content specs.

**Number scheme:**
- **01, 02, 06, 07** — archived under `_archive-pre-harness-2026-05-25/`. Pre-harness-reframe scenarios; product they describe is no longer the product being built. See archive README for retirement rationale.
- **03, 04, 05, 08, 09, 10** — survive the harness reframe. Revised in place (Stefan→Roland rename in content + harness-model alignment + new sub-pages where the harness adds surface — e.g. Renate's Replay CLI demo in 05).
- **11+** — new harness-MVP scenarios (Live Board, Kanban, Kondukt skills, BYOK setup, …) authored fresh.

---

## Active scenarios (post-harness reframe)

| # | Scenario | Persona | Surface | Status |
|---|---|---|---|---|
| **11** | [Roland's Live Board — multi-subsystem diagnostics on the moving map](11-roland-live-board/11-roland-live-board.md) | Roland | Harness — Live Board | 🔴 NEW — pending |
| **12** | [Roland's Kanban triage — neglect-risk lane discipline](12-roland-kanban-triage/12-roland-kanban-triage.md) | Roland | Harness — Kanban | 🔴 NEW — pending |
| **13** | [Kondukt: Daily Brief skill on session open](13-kondukt-daily-brief-skill/13-kondukt-daily-brief-skill.md) | Roland | Harness — Kondukt | 🔴 NEW — pending |
| **14** | [Kondukt: Vendor meeting prep skill](14-kondukt-vendor-meeting-prep/14-kondukt-vendor-meeting-prep.md) | Roland | Harness — Kondukt | 🔴 NEW — pending |
| **15** | [BYOK key setup + LLM provider configuration](15-byok-setup/15-byok-setup.md) | Roland | Harness — config flow | 🔴 NEW — pending |
| 03 | [Klaus's pre-arrival brief → shadow sign-off](03-klaus-prearrival-shadow-signoff/03-klaus-prearrival-shadow-signoff.md) | Klaus | Depot Briefing PWA | 🟡 REVISE — hold-to-record sub-page (03.6) retires; sign-off success (03.7) simplifies to single-click; VLAN/IP/MAC/firmware surfaced in card detail per progressive-disclosure rule; Stefan→Roland rename N/A (Klaus persona unchanged) |
| 04 | [Elena's ACK + DEFER loop on shared device](04-elena-ack-defer-loop/04-elena-ack-defer-loop.md) | Elena | Depot Briefing PWA | 🟡 REVISE — survives mostly intact; references to RAO confidence grammar update; Stefan→Roland rename N/A (Elena persona unchanged) |
| 05 | [Renate's audit-export review at manday 15](05-renate-audit-export-review/05-renate-audit-export-review.md) | Renate | ECM Audit Export + Reconciliation | 🟡 REVISE — **add new sub-page for Recommendation Provenance Replay CLI live demo** (load-bearing for the seven-criteria Phase 2 composite); Suppression log inspection sub-page added; existing 6-page structure retained |
| 08 | [Roland + Martin's manday-15 ROI conversation](08-roland-martin-roi-conversation/08-roland-martin-roi-conversation.md) | Roland + Martin | Pilot Evidence Dashboard | 🟡 REVISE — adoption-dominated scorecard replaces euro-counterfactual; seven-criteria composite display; Replay archive integrity row; per-view acceptance breakdown |
| 09 | [Roland's shift handover](09-stefan-shift-handover/09-stefan-shift-handover.md) | Roland | Harness (across all three views) | 🟡 REVISE — handover summary inherits Kanban-lane-state + Kondukt-skill-invocation-history + suppressed-emission-log + unverified-outcome states; Stefan→Roland rename in content |
| 10 | [Harness degraded — Roland keeps working](10-stefan-kondukt-degraded/10-stefan-kondukt-degraded.md) | Roland | Harness (degraded) | 🔵 REWRITE — three-surface degradation (stale telemetry / MCP tool timeout / BYOK LLM unreachable) per PRD Journey 8; suppression-not-regeneration MVP failure mode |

**Status legend:** 🔴 NEW (not yet authored) · 🟡 REVISE (existing content needs harness-model + Roland-rename update) · 🔵 REWRITE (existing content materially reframes)

**Folder/filename note:** scenarios 09 + 10 keep their folder names with `stefan-` slug despite Stefan→Roland rename — deferred sweep (see [_progress/00-design-log.md](../_progress/00-design-log.md) "Persona pseudonym retired: Stefan → Roland" entry). Filename rename is a deliberate later pass; scenario *content* gets the rename now.

---

## Archived scenarios (pre-harness reframe)

Located at `_archive-pre-harness-2026-05-25/`. See archive `README.md` for retirement rationale per scenario.

| # | Scenario | Retirement reason |
|---|---|---|
| ~~01~~ | ~~Stefan's crisis redeployment~~ | RAO card grammar retired; cost-impact-one-liner reframes; crisis-redeployment work covered by Kanban + Kondukt |
| ~~02~~ | ~~Stefan's cost-aware non-crisis triage~~ | RAO card + modification panel retired; triage covered by Kanban + Kondukt |
| ~~06~~ | ~~Lukas's vendor warranty dispute~~ | Lukas persona retired (merged into Roland); warranty recovery workflow descoped from MVP; detection lives via Kondukt skill |
| ~~07~~ | ~~Lukas's OTA canary + drift monitoring~~ | Lukas persona retired; OTA management descoped from MVP; drift moved to ECM module |

---

## Kondukt skill registry (FR13)

Kondukt skills are not standalone scenarios — they're *invocations* inside the Kondukt view. Each scenario that demonstrates a skill anchors on a specific invocation context. The MVP skill registry:

| Skill | Demonstrated by | Notes |
|---|---|---|
| **Daily Brief** | Scenario 13 | First-kill artifact at session open; validated emission; Provenance Replay triplet |
| **Vendor meeting prep doc** | Scenario 14 | Multi-IntentPacket cross-fleet analysis + allowlisted vendor portal fetch + docx export |
| **Recurring-failure pattern report** | Live Board annotation context (scenario 11); deeper skill invocation deferred to future scenario | Live Board surfaces the annotation; the deep report is a Kondukt skill |
| **Fleet health summary** | Scenario 08 (ROI conversation) — used by Roland to brief his stakeholders | Periodic / pre-meeting use; xlsx export |

New skills evolve through pilot via versioned ADR (FR13). The registry is versioned in-product; operator can inspect constitution + schemas + validator rules.

---

## Page Coverage Matrix (post-harness)

Every page from the harness MVP surface inventory is exposed in at least one scenario.

| Page / View | Scenarios | Note |
|---|---|---|
| Live Board — moving map, fleet-wide | 11 | Roland's glanceable surface; group-pivot by subsystem / by train |
| Live Board — group-pivot UI | 11 | Subsystem-pivot ↔ train-pivot toggle; state persisted per-user |
| Live Board → Kanban click-through redirect | 11 | Clicking a train on Live Board navigates to that train's Kanban detail card |
| Live Board annotations from Kondukt (recurring-failure, cross-subsystem correlation) | 11 | Provenance-attached inline annotations |
| Kanban — lane sort by neglect-risk (not queue order) | 12 | Roland's continuous all-day surface |
| Kanban — Kondukt-proposed lane moves with provenance | 12 | Validation Receipt visible on card |
| Kanban — train detail card (primary disclosure) | 12 | Subsystem name + VLAN + IP + MAC + switch port + firmware + time-since-fault |
| Kanban — train detail card (secondary disclosure on click) | 12 | MTTR (harness-computed) + recurring-failure history + 24h packet trace + vendor portal deep-link |
| Kanban — filters (train / subsystem / lane / neglect-risk band) | 12 | Filter state per-user |
| Kondukt — skill registry list + invocation | 13, 14 | Daily Brief (13), Vendor meeting prep (14); skill set evolves per ADR |
| Kondukt — canvas (in-progress artifact + xlsx/docx export) | 13, 14 | Working artifacts; not in audit trail |
| Kondukt — free-form chat | 14 (revision request inside Vendor meeting prep flow) | Catch-all for non-codified queries |
| Kondukt — allowlisted internet egress (per-fetch logged) | 14 | Stadler/Siemens/Knorr-Bremse vendor docs + CENELEC + ERA + EU regs |
| Kondukt — suppression banner (validation failure) | 13 (degraded variant); 10 (harness-degraded) | "Kondukt suppressed — reason: {reason}" |
| Kondukt — BYOK-degraded banner | 10 | Provider 429/5xx; skill invocations paused; telemetry views still functional |
| Pre-arrival brief (Depot Briefing PWA) | 03, 04 | Klaus + Elena role views |
| Role-filtered card detail (Electrical / Mechanical / IT) | 03, 04 | Elena's discipline filter |
| ACK confirmation (one-tap) | 04 | Elena's resolution log |
| DEFER with reason | 04 | Elena's deferral with structured reason |
| ECM shadow sign-off (simple click, MVP) | 03 | Klaus's MVP gesture — hold-to-record descoped to Phase 2 candidate |
| ECM audit-trail export (JSON) | 05 | Renate's review surface |
| Reconciliation queue + divergence flagging | 05 | Paper-vs-shadow divergence list |
| `supersedes_id` correction model | 05 | Append-only correction; no in-place mutation |
| **Recommendation Provenance Replay CLI live demo** | 05 | Renate (manday ~15) + outside auditor (manday 20–25) run the CLI on their laptop |
| **Validation Receipt suppression log inspection** | 05 | Renate sees what the validator rejected and why |
| Pilot Evidence dashboard — seven-criteria composite | 08 | Roland + Martin sign-off artefact |
| Per-view acceptance breakdown | 08 | Live Board / Kanban / Kondukt skills / Kondukt free-form |
| Provenance Replay archive integrity row on dashboard | 08 | Replay-verify-clean rate + suppression count |
| Harness shift handover summary | 09 | Open Kanban cards + in-flight Kondukt skill invocations + suppressed-emission log + unverified-outcome states |
| Harness degraded — stale telemetry / MCP tool timeout / BYOK LLM unreachable | 10 | Three independent degradation surfaces |
| BYOK key vault — first-time setup + token usage transparency | 15 | Encrypted at rest, per-user, never logged |
| Allowlist domain inspection | 15 (config side) | Operator can view active allowlist |
| Constitution + skills inspectability | 15 (config side) | Operator can view what Kondukt is told to be |
| Login / IdP federation entry | 03, 04, 05, 08, 09, 10, 11, 12, 13, 14, 15 | Pre-condition assumed for all scenarios |
| Relay card | — | Future state — Dispatcher persona deferred from MVP |
| Lastenheft export workflow | — | Vision-tier |
| Three-zone Telematics Control Panel | — | RETIRED — Unified Telematics Control Panel framing retired in harness reframe |
| RAO card with three-tier confidence visual grammar | — | RETIRED in MVP — Kondukt emissions carry confidence pill; LOW-confidence DISABLE applies to recommendation card Accept; no Mark-executed gesture in MVP |
| Hold-to-record gesture | — | DESCOPED FROM MVP — preserved in design-system reference as Phase 2 candidate |

---

## Driver Coverage Matrix (post-harness)

Every Trigger Map Must-Address driver is addressed by at least one scenario.

| Driver (from revised Trigger Map) | Persona | Scenario(s) |
|---|---|---|
| Three views, not four (Live Board / Kanban / Kondukt) | Roland | 11, 12, 13, 14 |
| Daily Brief as Kondukt skill, not separate view | Roland | 13 |
| No VLAN IDs on Live Board; VLANs in Kanban detail | Roland | 11 (no VLANs visible), 12 (VLANs visible on expansion) |
| Live Board click → Kanban detail card redirect | Roland | 11 → 12 transition |
| Kanban progressive disclosure (primary / secondary on click) | Roland | 12 |
| Kondukt skill registry (Daily Brief + Vendor meeting prep + Recurring-failure + Fleet health) | Roland | 13 (Daily Brief), 14 (Vendor meeting prep), 08 (Fleet health), 11 (Recurring-failure annotation) |
| Constitution-as-contract enforcement (per-skill schema + provenance validator + advisory-language linter) | Roland | 13, 14 (validated emissions); 10 (suppression on tool timeout) |
| Recommendation Provenance Replay triplet at emission | Roland | 13, 14, 11; verified in 05 by Renate |
| Replay CLI demo | Renate | 05 |
| Suppression-not-regeneration ("silence is safe") | Roland | 10 (degraded), 13 (validation failure variant) |
| LOW-confidence DISABLE rule + modification_distance | Roland | 12 (Kanban card Accept disabled on LOW); 13 (Daily Brief LOW emission) |
| BYOK key + token transparency + degraded mode | Roland | 15 (setup), 10 (degraded) |
| Allowlisted internet egress | Roland | 14 (vendor portal fetch) |
| Cross-subsystem correlation from Kondukt | Roland | 11 (Live Board annotation), 14 (vendor meeting prep input) |
| MTTR computed by harness from landside-observed timestamps | Roland | 12 (Kanban secondary disclosure) |
| Train-to-landside-DB mapping integrity | Roland (visible in ops dashboard) | (Implicit in all 11–14; explicit handling in 10 if mapping divergence is a degradation scenario) |
| Pre-arrival brief opens 14 min before HAFAS arrival, role-filtered | Klaus, Elena | 03, 04 |
| Simple-click sign-off (MVP) | Klaus | 03 |
| IdP-bound identity, HMAC, hash chain, `supersedes_id` in audit export | Renate | 05 |
| Divergence between paper and shadow surfaced within 24h | Klaus, Renate | 05 |
| Role persistence (session-only shared vs stored personal) | Elena, Klaus | 03, 04 |
| Shift handover preserves open Kanban cards + Kondukt skill invocations + suppressed emissions + unverified outcomes | Roland | 09 |
| Recommendation→action→outcome chain capture | Roland, Renate | 11, 12, 13, 14 (emission); 05, 08 (audit) |
| Harness degraded UX — Live Board + Kanban remain functional when BYOK LLM unreachable | Roland | 10 |
| Pilot Evidence dashboard — 7-criteria composite | Roland, Martin, Renate | 08 |

---

## Methodology Notes

- **Sunshine path only.** Every scenario is the success case. Error/degraded variants are *separate* scenarios (e.g., scenario 10) rather than `if` branches inside another scenario.
- **One transaction per scenario.** If a scenario has two transactions, it gets split.
- **Sub-page granularity per scenario.** Scenarios with substantial UX depth (03, 04, 05, 11, 12) carry sub-page specs in `NN.M-[page-slug]/` folders. Simpler scenarios (08, 09, 10, 13, 14, 15) may be single-document depending on what the surface earns.
- **Personas are real names where the real-world equivalent has signed off.** Roland Ruisz (formerly pseudonymous as Stefan — renamed 2026-05-25 to use real first name). Klaus / Elena / Renate retain their pseudonymous form pending real-world equivalent identification.
- **Constitution + skill registry are versioned in-product.** New skills don't require new scenarios — they require an ADR + an addition to the FR13 registry list. Scenarios anchor on the *invocation pattern* not on individual skills.

---

## Next Steps

1. **Author new harness-MVP scenarios:** 11 (Live Board) → 12 (Kanban) → 13 (Daily Brief skill) → 14 (Vendor meeting prep skill) → 15 (BYOK setup). Sequenced to surface the harness USP first (Live Board glanceability + Kanban depth via progressive disclosure + Kondukt skill invocation) before the supporting flows (BYOK config).
2. **Revise scenarios in place:** 03 (sub-page 03.6 retires, 03.7 simplifies), 04 (RAO refs update), 05 (Replay CLI demo sub-page added + Suppression log inspection sub-page), 08 (dashboard contents reflect 7-criteria composite + adoption-dominated scorecard), 09 (handover summary scope expanded).
3. **Rewrite scenario 10** for three-surface harness-degraded model.
4. **Validate with Roland Ruisz** — primary-persona driver list + new harness-view scenarios 11, 12, 13, 14, 15 confirmed by the real-world equivalent. (Previous Stefan / Lukas dual-hat validation framing retired — one role, one set of scenarios.)
5. **Validate with Renate Fischer** — scenario 05 + audit-export surface in 08 + Replay CLI demonstrated live at manday ~15.
6. **Validate with an ECM Manager peer** — scenario 03.
7. **Validate with an ECM 4 Technician peer (Electrical)** — scenario 04.

---

_Generated with Whiteport Design Studio framework — Freya / Suggest mode_
_Harness-reframe revision 2026-05-25_
