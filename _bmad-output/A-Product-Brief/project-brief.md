# Project Brief: ÖBB Landside Platform

> Simplified Brief — Essential context for design work

**Created:** 2026-05-25
**Revised:** 2026-05-25 (harness-model reframe)
**Revised:** 2026-05-25 (post-party-mode stress-test — contract enforcement, Provenance Replay as headline test, adoption-dominated scorecard)
**Revised:** 2026-05-25 (operator correction — restored Live Board + Chat+Canvas as MVP USP, reverted scorecard to adoption-dominated, removed capability admission)
**Party mode:** Saga · John · Winston · Mary · Victor — 3 rounds; see [_progress/00-design-log.md](../_progress/00-design-log.md). Outcome retained: contract-as-enforcement + Recommendation Provenance Replay. Outcome overridden by operator: 5→2 view collapse (restored to 4 views).
**Author:** Abbas Rizvi
**Brief Type:** Simplified
**Mode:** Freya / WDS Designer (revision pass)

---

## Project Scope

**A landside fleet management harness for an operator-provided LLM, exposing live multi-subsystem train telemetry through a constitution + skills + MCP scaffolding, producing daily briefs, kanban triage, chat-driven research, and recommendation artifacts — read-only in MVP.**

The product is **the harness**, not the LLM. ÖBB Flottenmanagement plugs in their LLM API key (Anthropic, OpenAI, or other — BYOK) and the harness gives that LLM:

1. A **constitution** — the platform's voice, principles, and operator-facing identity (Kondukt — the persona any customer-provided LLM adopts inside the harness)
2. A library of **skills** — domain-specific procedures the LLM can invoke (read fleet telemetry, summarise a fault history, build a daily brief, draft a report)
3. **MCP server access** — structured tools over the landside fleet data layer (Rail MCP Server: train telemetry, subsystem health, depot state, drift detection, MTTR from Stadler). The MCP server reads from a **PostgreSQL + pgvector store** that holds every IntentPacket and every subsystem-telemetry stream from every train, with embeddings layered over fault summaries, vendor documentation, and prior recommendations for retrieval-augmented context. One engine, two paradigms: relational/timeseries tables for structured queries (MTTR, depot dwell, fleet counts) + pgvector for semantic search ("show me trains with intermittent CCTV faults matching the pattern on 4736-101").
4. **Tools** for artifact production — spreadsheet (`.xlsx`) and document (`.docx`) generation with operator-triggered download
5. **Allowlisted internet access** — curated approved-domains list (Stadler / Siemens / Knorr-Bremse vendor docs, CENELEC, ERA, EU regulatory texts) for deep-research troubleshooting; no open web egress

The harness produces **four views in MVP** (revised 2026-05-25 after party-mode + operator correction — party mode over-cut; Live Board and Chat+Canvas are the USP, not optional):

| View | What it is | First-kill decision | LLM role |
|---|---|---|---|
| **Daily Brief** | Morning situation report — fleet overnight state, changes since last shift, recommended focus | The 06:30 disposition triage: which of last night's anomalies actually changes today's plan | LLM-generated on session open; emitted only if the Recommendation Provenance Replay validator passes |
| **Kanban — Neglect-Risk Triage** | Open faults lane-sorted by neglect-risk ("which incident am I ignoring that will bite me by Friday"), NOT by queue order | The continuous all-day question: where is decay accumulating without attention | LLM proposes lane placement + neglect-risk score; operator moves cards; every placement carries provenance |
| **Live Board** | Fleet-wide moving map + per-subsystem health across CCTV (VLAN 5), PIS (VLAN 3), intercoms (VLAN 9), AFZ (VLAN 8), audio (VLAN 9), Wi-Fi (10/20/30/31/131/150), switches (trunk), TCMS (brain VLAN 7 SNMP), Stadler RDC (VLAN 200/202). Group-pivot both ways: by subsystem ("all CCTV faults across the fleet") OR by train ("everything wrong with 4736-101"). | The continuous situational-awareness read: where is the fleet, what's degrading, what's the cross-subsystem pattern | Telemetry-driven; LLM annotates with recurring-failure detection and cross-subsystem correlations (provenance-validated per the contract) |
| **Chat + Canvas** | Free-form conversation with Kondukt; side-canvas for in-progress reports/spreadsheets; allowlisted internet research; `.xlsx` and `.docx` export | The on-demand deep-research question: "why is fault X recurring on DOSTO Neu but not Enzo?" — and the on-demand artifact: a vendor-facing report or fleet summary | Operator-driven; LLM responds via constitution+skills+MCP+egress; every chat turn carries a Validation Receipt; downloaded artifacts are working artifacts (not audit records) |

**Why these four (operator-locked 2026-05-25 — overriding party-mode demotion):** Daily Brief + Kanban are the *structured* operational surfaces. **Live Board is the live multi-subsystem visibility no competitor produces** — Nomad Monitoring sees only IoB; the harness sees CCTV, PIS, intercoms, AFZ, audio, Wi-Fi, switches, TCMS, RDC, all correlated. **Chat+Canvas is the LLM-native deep-research + artifact-authoring surface** — this is what makes the product an *agentic OS for fleet ops*, not a dashboard. Live Board and Chat+Canvas together carry the USP. Demoting them collapses the harness reframe into a generic recommendation tool.

**Deferred to post-MVP:**

| View | Status | Reason |
|---|---|---|
| Drift Visibility | Move to ECM module | First-kill cadence is quarterly at best — belongs in the ECM audit trail surface, not the daily harness. |

**Posture (post-party-mode + operator-corrected):** The MVP serves the Fleet Manager across Roland's full work — Disponent (Daily Brief + Kanban), Telematik (Live Board), Teil-PL (Chat+Canvas for vendor-meeting prep and Lastenheft research). Saga's party-mode caution remains valid as a *measurement* concern (each view's effectiveness must be measured per-hat in the pilot scorecard, not aggregated), but the *scope* decision is locked: all four views are MVP because Live Board + Chat+Canvas are constitutive of the product, not optional surfaces.

**MVP execution posture: read-only, recommendation-only.** The LLM proposes; the operator acts via their existing systems (vendor portals, depot work orders, HAFAS direct, paper sign-off). The harness records what was recommended and what the operator did — the shadow audit trail. No autonomous actions in MVP.

**Cross-cutting capabilities:**

- **Recommendation Provenance Replay** — the load-bearing pilot test (party-mode lock, Winston, 2026-05-25). Every recommendation the harness emits is stored as a triplet at emission time: (a) the Recommendation Envelope (consist ID, recommended action, advisory class, confidence band, timestamp, harness version, model+version, skill invoked); (b) the Provenance Bundle (IntentPackets consumed, MCP queries issued, raw LLM output before parsing); (c) the Validation Receipt (schema check, citation-trace check, advisory-class check, disposition: emitted / regenerated-then-emitted / suppressed). The headline test: any recommendation from the pilot corpus can be **replay-verified** by an auditor-runnable CLI six months later, with no access to the original LLM session. Aggregate pass criterion: ≥99% replay-verify cleanly; zero recommendations exist without a triplet; advisory-only language linter has zero breaches. This is what gets shown to Renate Fischer at the manday-20–25 tabletop and what differentiates the harness from a chat-log wrapper.
- **ECM audit trail** — append-only, IdP-bound, HMAC-signed, hash-chained record under EU 2019/779. MVP shadows paper sign-off; Phase 2 promotes to authoritative. **Scope:** recommendation + operator-action events only — daily briefs and chat transcripts are working artifacts, not audit records. The Recommendation Provenance Replay reconciles 1:1 with this trail — every recommendation triplet maps to a shadow-audit entry; every shadow-audit entry touching a harness-influenced workflow has a triplet.
- **Rail MCP Server** — the harness's connection to the landside data layer. One toolset, used by Kondukt in every view.
- **Constitution-as-Contract (not prompt).** The constitution is not just a system prompt — it is **partially enforced by the harness** via per-skill output schemas and provenance validators (Winston, party mode 2026-05-25). The LLM's structured outputs are validated synchronously before emission; if validation fails, the recommendation is suppressed (logged with reason) rather than emitted. **Regeneration loop is deferred from MVP** — suppression is the MVP failure mode. *"Silence is safe. The Disponent does not get a recommendation; they fall back to current practice. You lose availability, not safety."* This is the safety posture that makes a one-person Einzelunternehmen tolerable as a NIS2-scope supplier on a safety-adjacent system.

**Scope boundary:** The onboard brain (Hailo-10H inference, Conductor App PWA, IntentPacket schema, edge resilience) is a separate product in `oebb-brain/`. Landside consumes IntentPackets from the brain; the brain does not consume landside output. **TCMS data path:** brain polls TCMS on onboard VLAN 7 via SNMP and forwards landside — landside sees TCMS through the brain, not directly. SAP PM, ServiceNow, BOOM are post-MVP.

---

## Challenge / Opportunity

**The problem.** ÖBB's Fleet Manager makes high-stakes decisions across fragmented technical systems: vendor diagnostic portals (Stadler, Siemens, Knorr-Bremse), the Nomad Monitoring System (current live view — Internet-on-Board only, no other subsystems), Power BI dashboards, ServiceNow tickets, MS Project schedules, Excel reconciliation files. The current fleet visibility tool (Nomad) monitors only the IoB layer; CCTV, PIS, intercoms, AFZ, audio, TCMS, and the broader subsystem fleet have no unified landside view. Every switch costs cognitive load; every gap is a place a fault goes unattributed. Recurring failures stay invisible across vendor silos. MTTR is not surfaced. Daily situational awareness is reassembled from scratch each shift.

**Why now.** Three forces converge:

- **Software 3.0 / agentic-OS moment.** LLMs have crossed the threshold where a domain-specific harness over real operational data is a defensible product, not a demo. The operator owns the LLM key — no vendor lock-in to a specific model, no inference-resale economics, no model-churn risk.
- **EU AI Act + NIS2 + ECM Directive 2019/779.** Advisory-only LLM posture with allowlisted egress and an append-only audit trail is the compliance-native architecture for a regulated-rail-operations agent. Autonomous-action AI in this domain is regulatorily prohibitive; advisory + recommendation + audit is feasible today.
- **ÖBB internal driver.** Roland Ruisz — DI(FH), Flottenmanagement / System- und Betriebstechnik / Team Telematik, Teil-Projektleitung Telematik for Cityjet DOSTO Neu and Cityjet Enzo — has named the multi-subsystem visibility gap and the lack of an LLM-native fleet-management tool as live operational pain. The pilot window is end-Aug / Sept 2026 (manday 1), with Hailo procurement and Einzelunternehmen registration as pre-pilot blockers.

**The opportunity.** Build the harness that turns a fleet manager's LLM into a constitution-bound fleet co-pilot across every IP-addressed subsystem and every train. Close the *information* gap in MVP; earn trust via five measurable criteria; close the *execution* gap in Phase 2 — becoming the regulated-rail-operations LLM-native system of record for the operator.

**Differentiators competitor dashboards cannot replicate:**

- **The constitution-as-contract enforcement layer + provenance archive.** This is the non-acquirable asset (Victor + Winston, party mode 2026-05-25). Constitution + skills + MCP + pgvector + egress, as configuration, is commodity — assemblable by any competent platform team in a quarter. **Constitution + skills + MCP enforced by a harness with per-skill output schemas, provenance validators, and an auditor-runnable Replay CLI** is the actual moat — because the moat isn't the components, it's the proof that the LLM cannot escape them. The asset compounds with usage: each pilot week, the schemas + validators encode more of Roland's regulated-rail decision context, and that encoding is what "two engineers + an Anthropic contract" cannot replicate without six months of access to Roland.
- **Multi-subsystem visibility.** CCTV (VLAN 5), PIS (VLAN 3), intercoms (VLAN 9), AFZ (VLAN 8), audio (VLAN 9), Wi-Fi (VLAN 10/20/30/31/131/150), switches (trunk), TCMS (brain VLAN 7 SNMP), Stadler RDC (VLAN 200/202) — all in one harness with one LLM context. The Nomad Monitoring System sees only IoB; competitors see only their own vendor's slice.
- **ECM audit trail engineered as the regulatory record under EU 2019/779**, not as a log. Phase 2-ready from day one; advisory-mode in MVP. Reconciles 1:1 with the Recommendation Provenance Replay archive.
- **Recommendation-only posture as a feature**, not a limitation. Preserves operator authority; defensible under EU AI Act; matches Roland's expressed preference for advisory over autopilot.

**Commercial framing:** The harness is the *chassis*; the *fit* is the asset. The chassis alone is not the moat. The fit — regulated-rail recommendation logic + ECM 2019/779 reconciliation + Roland's operational decisions encoded into the constitution over the pilot — emerges from usage and is what compounds. **The pilot is the discovery process for the fit**, not the proof of a pre-existing wedge. We commit to adoption-dominated pilot metrics now; euro-denominated metrics are post-pilot research outputs, not pre-pilot pass criteria.

---

## Design Goals

**Functional**

- Fleet Manager opens the harness in the morning and the **daily brief** is already there — fleet overnight state, changes since last shift, recommended focus areas, top incidents needing triage.
- **Kanban board** surfaces every train with an open critical fault, lane-placed by the LLM, movable by the operator.
- **Live Board** shows every train (in service + in depot) on a moving map with per-subsystem health legible without drill-down; time-since-fault visible per train. MTTR computed by the harness (`t_close − t_open` of Stadler alarm IntentPackets — landside-observed timestamps; harness does not read Stadler's internal MTTR field) surfaces on the Kanban detail card (secondary disclosure).
- **Drift visibility** shows version mismatches across the fleet at a glance — no rollout management, just visibility.
- **Chat + Canvas** allows the operator to ask the harness LLM anything (with allowlisted internet research), produce reports and spreadsheets, download as `.xlsx` / `.docx`.
- Recommendation acceptance rate ≥ 70% within 30 pilot mandays (working assumption; confirm with Roland).

**Experience**

- **Glanceable under pressure.** Fleet Manager is on a 12-hour shift. Information density is high but readable in one scan.
- **Group-pivot both ways.** Alerts groupable by subsystem ("all CCTV faults across the fleet") AND by train ("everything wrong with 4736-101"). Same data, two pivots.
- **LLM attribution + confidence on every artifact.** Every Kondukt-produced view (brief, kanban placement, chat reply, downloadable report) carries a confidence indicator. LOW-confidence DISABLE rule (locked 2026-05-25) applies to harness recommendations.
- **Decision-traceable.** Every shadow audit entry answers: who decided, when, what action was taken, what was the outcome. Daily briefs and chat conversations are *working artifacts*, not audit records.
- **Recommendation, not command.** Kondukt proposes; operator decides; operator acts via existing systems; harness records what they did. MVP must not pre-empt the Phase 2 contract.
- **Visible to colleagues.** Roland works alone, but his screens are read by Team Telematik peers — every view must read cold by a peer, not just by Roland.

**Business** *(operator-corrected 2026-05-25 — euro-denomination was premature; the pilot exists to discover the euros, not prove them)*

- **Pilot scorecard is adoption-dominated, not euro-denominated.** Euro figures (delay-minutes attributable, ECM audit hours saved, warranty-recoverable detections, dispatch-decision concurrence) are post-pilot research outputs, not pre-pilot pass criteria. We don't have baseline numbers yet — committing to euro thresholds in the brief would be making numbers up. The pilot is the instrument that produces them.
- **Primary pilot metric: Recommendation acceptance rate ≥70%** within 30 pilot mandays (working assumption, confirm with Roland at first alignment session). Per-hat measurement, not aggregated — acceptance on Disponent-hat decisions (Daily Brief + Kanban), Telematik-hat reads (Live Board annotations), Teil-PL-hat artifacts (Chat+Canvas research and reports) tracked separately to avoid survivorship bias (Saga's party-mode caution).
- **Phase 2 readiness verdict at pilot week 24** with the five composite trust criteria visibly tracked weekly throughout the pilot. The five criteria are the *texture* — the qualitative + quantitative signals we accumulate. Sign-offs are the *gate*.
- **Renewal-conversation script — sign-offs as gate, trust criteria as texture:**
  - **Gate (3/3 required to extend):** Did Roland Ruisz sign off? Did Renate Fischer sign off? Does the Recommendation Provenance Replay archive pass the outside-auditor tabletop verdict at manday 20–25?
  - **Texture (what we describe in the renewal conversation):** the five composite trust criteria as the operational story, plus the adoption-rate breakdown per-hat, plus the qualitative observations Roland and Renate name in their sign-off rationale.
  - 3/3 sign-offs + clean Replay archive → extend. Anything less → don't.
- **Reference customer + qualitative ROI narrative + named pipeline (~€470K ARR fleet potential)** sufficient to support a fundable event with Speedinvest / i5invest. The "validated ROI numbers" framing is post-pilot — pre-pilot, we have a credible adoption hypothesis and a defensible safety architecture, not validated euro figures.

---

## Constraints

**Regulatory (one-way doors)**

- **EU ECM Directive 2019/779** — audit trail is a first-class requirement; MVP shadows paper, Phase 2 must be authoritative-ready.
- **EU AI Act** — high-risk classification likely; advisory-only posture is compliance-native and must not regress in MVP.
- **NIS2** — supplier-chain requirements apply to the harness vendor with footprint on ÖBB's operational network. **Allowlisted LLM egress** is the NIS2-defensible posture for internet research.
- **GDPR** — DPA required before any ÖBB operational data touches the harness. BYOK key handling: encrypted at rest, scoped per user, never logged, never transmitted to harness vendor's logs.

**Architectural**

- **Landside data store: PostgreSQL + pgvector, single engine.** All IntentPackets + all subsystem telemetry land here. Structured tables for relational/timeseries queries; pgvector extension for embedding-based semantic retrieval. Single sovereign instance per ÖBB deployment; NIS2-defensible. Selected over multi-engine architectures for surface-minimisation per project principle (smaller surface + earned trust).
- **IntentPacket schema v1.0.0** is owned by `oebb-brain/`; landside is a downstream subscriber.
- **TCMS data path** — onboard brain polls TCMS on VLAN 7 via SNMP and forwards landside. TCMS is brain-mediated, not Stadler-RDC-mediated.
- **MTTR source** — Stadler diagnostic system (RDC). Component alert created → cleared = MTTR. Read by harness; not recomputed.
- **BYOK** — operator brings their LLM API key. Harness never stores plaintext keys. Per-user scope. Token usage transparent to the operator (BYOK = their money). Provider-agnostic.
- **Constitution + skills versioned in-product** — operator can inspect what Kondukt is told to be. No hidden prompting.
- **Allowlisted egress** — curated approved-domains list for LLM internet access. Vendor portals (Stadler, Siemens, Knorr-Bremse), standards bodies (CENELEC, ERA), EU regulatory texts. No open web.
- **Single ÖBB instance** for PoC; per-operator deployment at fleet rollout. Multi-tenancy is post-MVP.
- **Hailo hardware** — one Hailo-8 + Hailo-10H M.2 pair per consist; platform fee unit is 1 train consist.

**Process / commercial**

- Pilot: **3 consists, 6 months, €15,000 total** (€7,500 on signature, €7,500 week 4) — below ÖBB procurement-committee threshold so Roland can approve directly.
- Manday 1: end-Aug / Sept 2026; Hailo procurement and Einzelunternehmen registration are pre-pilot blockers.
- Pilot success gate: composite health score + Roland Ruisz sign-off + Renate Fischer sign-off + outside-auditor tabletop verdict + zero unresolved pilot-kill triggers in final 30 days.

**Persona scope**

- **One primary persona — Fleet Manager (Roland).** Real-world: Roland Ruisz, DI(FH) — Flottenmanagement / System- und Betriebstechnik / Team Telematik, with Teil-Projektleitung Telematik for Cityjet DOSTO Neu + Cityjet Enzo. **One role, two programme hats** — not two roles. (Previously pseudonymous as "Stefan" — renamed 2026-05-25 to use the real first name directly.)
- **Secondary personas** (recommendation-consumer side): ECM Manager (pre-arrival brief), ECM 4 Technician (ACK/DEFER on resolutions). These remain valid; their surfaces are downstream of harness recommendations.
- **Retired:** *Teil-Projektleitung Telematik as a separate persona* (was: Lukas). The role is real — it is one of Roland's hats — but it is not a distinct persona requiring a distinct UI surface. Merged into the Fleet Manager persona on 2026-05-25 (see design log).
- Dispatcher (Fahrdienstleiter) remains deferred from MVP.

**Design / brand**

- One-person company (Einzelunternehmen, Vienna) — no design system, no existing brand assets to honour. Greenfield surface.
- Bias toward smaller surface + earned trust over over-designed systems (operator preference: Abbas).

---

## What's Cut from the Previous Brief

The harness reframe (2026-05-25) descopes several prior MVP capabilities. Logged for traceability:

| Cut from MVP | Reason | Vision-tier status |
|---|---|---|
| **Vendor warranty dispute workflow** (was scenario 06) | Not the wedge; harness positioning carries the commercial argument | Post-MVP capability; harness can support it via Chat+Canvas + skills once pilot evidence supports it |
| **OTA rollout management** (was scenario 07) | Out of scope; vendor OTA tooling remains canonical | Vision tier |
| **Drift Visibility as a daily view** | Quarterly cadence; belongs in the ECM audit module, not the daily harness | Lives in ECM module |
| **HAFAS validation against onboard GPS** | Out of scope for MVP — separate validation surface not needed | Possible post-MVP capability via new skill |
| **"Three surfaces" framing** (Fleet Manager AI Interface + Depot PWA + Telematics Control Board as separate products) | Collapses into one harness with four MVP views (Daily Brief + Neglect-Risk Kanban + Live Board + Chat+Canvas) | N/A — Depot PWA remains as a downstream consumer of recommendations; Telematics-as-separate-product retired |
| **Kondukt as autonomous RAO-producing agent** | Reframed: Kondukt is the LLM-persona identity any customer-provided LLM adopts inside the harness, with constitution **enforced as contract** by per-skill schemas + provenance validators | Kondukt brand survives; autonomous-action posture deferred to Phase 2 |
| **Probebetrieb commissioning workflow** as a primary scope item | Roland's commissioning hat is one of his contexts, not a separate persona; harness supports the work through the four MVP views, not a dedicated surface | Vision tier |

---

## Next Steps

This brief sets the scope for the harness-model reframe. The following work proceeds:

- [ ] **PRD rewrite** — architecture statement, FRs, MVP scope re-aligned to harness model
- [ ] **Trigger Map recompute** — retire Lukas drivers; add harness/BYOK/daily-brief/kanban/chat-canvas/drift drivers under Roland
- [ ] **Feature Impact Analysis recompute** — Lukas-2 multiplier dissolves into Roland-3; feature priorities resort; sprint plan implications surfaced
- [ ] **Persona revision** — rewrite `01-disponent-stefan.md` around the harness model; archive `04-telematik-lukas.md`
- [ ] **New MVP scenarios (4 views, plus BYOK as config flow):**
  1. Daily Brief (Disponent hat first-kill = 06:30 disposition triage)
  2. Neglect-Risk Kanban (continuous all-day; lane-sort by decay-without-attention, not queue order)
  3. Live Board (multi-subsystem moving map; group-pivot by subsystem or by train; cross-subsystem correlation annotations from Kondukt)
  4. Chat + Canvas (free-form LLM session with allowlisted internet research; xlsx/docx artifact production; the USP that distinguishes this from a dashboard)
  Plus BYOK key setup + LLM config as a first-run flow (not a top-line scenario).
  Drift Visibility moved to ECM module (quarterly cadence; not in daily harness).
- [ ] **Existing scenarios reassessed** — 01/02 (Roland triage) reframe into the neglect-risk kanban + daily brief; 03/04 (Klaus/Elena) remain as recommendation-consumer surfaces; 05 (Renate audit) largely unchanged but reconciles with the Recommendation Provenance Replay; 09 (handover) gains brief+kanban structure; 10 (Kondukt degraded → harness degraded) rewritten with suppression-not-regeneration as the MVP failure mode
- [ ] **Recommendation Provenance Replay spec** — Winston's pilot-test architecture as a standalone document; auditor-runnable CLI; reconciliation with shadow audit trail. Drives PRD architecture section.
- [ ] **Pilot scorecard instrumentation** — operationalize the 4 euro-denominated numbers (delay-minutes attributable, warranty-recoverable detections, ECM audit hours saved, dispatch-decision concurrence rate)
- [ ] **Design System updates** — Kondukt attribution + confidence grammar on all LLM artifacts; LOW-confidence DISABLE rule extends to harness outputs

---

_Revised by Whiteport Design Studio — Freya / WDS Designer (harness-reframe pass)_
