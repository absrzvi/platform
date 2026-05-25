# Roadmap to Fundraise
**Date:** 2026-05-23
**Owner:** Abbas Rizvi
**Goal:** Everything needed from now to first Speedinvest / i5invest meeting
**Update this file** when a task is done — change `[ ]` to `[x]`

---

## Phase 1 — Before PoC Signs
*Complete these before the ÖBB contract is signed. All are blockers.*

- [ ] **#1 Register Einzelunternehmen in Vienna**
  Register sole proprietorship via WKO (Wirtschaftskammer Österreich). Can be done online via USP.gv.at or in person at WKO Wien. Need: passport, Austrian address, business activity description (software/AI services for rail). Cost ~€100–200.

- [ ] **#3 Decide and register company name / trade name**
  Choose a trading name. Check availability at WKO and domain availability. Needs to work in English for international operator conversations. Register the domain once decided.

- [ ] **#2 Open a business bank account**
  After Einzelunternehmen registration. Options: Erste Bank, Raiffeisen, or online (Wise Business, Revolut Business for lower fees). Required to receive PoC payment (€7,500 on signature, €7,500 week 4).

- [ ] **#4 Get GDPR Data Processing Agreement drafted**
  Engage an Austrian lawyer to draft a GDPR-compliant DPA (Auftragsverarbeitungsvertrag) between the company and ÖBB. Required before any ÖBB operational data touches company systems. Cost ~€1–3K. Draft as a reusable template for future operators.

- [ ] **#5 Transfer IP ownership to the business**
  Document that all IP (codebase, PRD, schemas, Rail MCP Server spec, commercial model) is owned by the Einzelunternehmen, not Abbas personally. Important for investor due diligence. Note: if converting to GmbH for fleet contract, IP transfer will need to be formalised again.

- [ ] **#6 Draft and sign PoC contract with ÖBB**
  Contract must cover: €15,000 total (€7,500 on signature, €7,500 week 4), 3 consists, 6 months, hardware ownership transfer on fleet contract execution, LLM cost absorption capped, success gate (composite pilot health score, Roland + Martin sign-off week 6), pilot-kill triggers. Signed by the registered Einzelunternehmen. Lawyer review before signing.

---

## Phase 2 — Technical (Parallel to Legal)
*Can run in parallel with Phase 1. Must be complete before PoC goes live.*

- [ ] **#7 Complete platform architecture document**
  Run `bmad-create-architecture` → `platform/architecture.md`. Winston resolves 5 open decisions: onboard edge RTO, Kondukt intent disambiguation rules (FR31), MQTT broker identity + failover, schema evolution strategy post-v1.0.0, Conductor App native wrapper.

- [ ] **#8 Complete epics and stories for Phase 2**
  Run `bmad-create-epics-and-stories` → `platform/epics.md`. 4 epics: Conductor App, Depot Briefing Interface, Fleet Manager AI Interface, Rail MCP Server. Requires architecture complete first.

- [ ] **#9 Run pre-story technical spikes**
  Four spikes before build: (1) Hailo-10H load test — sub-5s P99 under concurrent load; (2) pgvector P99 — <500ms semantic search SLA; (3) Kondukt spikes ×4 — intent disambiguation, BYOK key rotation, cost simulator, fallback; (4) Dismissal arbitration spike — two-stage loop mechanics.

- [ ] **#10 Build and deploy Phase 2 platform for PoC**
  Implement 4 epics on 3 PoC consists. Must pass PRD success criteria: Conductor App <5s P95, ECM audit trail 100% write success, pgvector P99 <500ms, Fleet Manager query <3s P95.

---

## Phase 3 — During PoC (Months 1–6)
*Run these in parallel while the pilot is live.*

- [ ] **#11 Get Roland's cost-per-delay-minute figure (pilot week 1)**
  Ask Roland Ruiz: *"What is ÖBB's internal cost-per-delay-minute figure used for Disponent KPI reporting?"*
  This single number converts the ROI from estimated ranges to a precise ÖBB-validated figure. It makes Roland a co-author of the commercial case and is the most critical input for the investor narrative. Record in `platform/commercial-model-2026-05-23.md`.

- [ ] **#12 Document everything during PoC for ISO 27001 evidence base**
  Maintain a running log from day 1: architectural decisions, access control, incident responses, deployment procedures, OTA update processes, data flows. Retroactive documentation is weak evidence — start immediately.

- [ ] **#13 Commission ISO 27001 gap assessment**
  Engage external consultant. Cost ~€5–15K. Target: complete by PoC month 3. Output feeds fleet procurement qualification and investor due diligence.

- [ ] **#14 Build financial model**
  Cover: monthly burn rate, PoC cash flow (€7,500 × 2), fleet ARR projections (€470K at 47 consists × €10K), LLM cost pass-through (€150–400/consist/month), hardware costs at volume, runway scenarios. Use actual PoC usage data to validate LLM estimates.

- [ ] **#15 Start co-founder search**
  Profile: owns commercial, compliance, customer success — not another engineer. Ideal background: rail industry OR enterprise B2B SaaS sales. Where to look: ERA/UIC events, InnoTrans network, ÖBB/operator alumni, Vienna startup ecosystem. Must have a co-founder in place (or credible plan) before Speedinvest/i5invest meeting.

- [ ] **#16 Warm up Speedinvest and i5invest relationships**
  Use existing references to make introductions NOW — not to pitch, just to be on their radar. Share that a live PoC is running on ÖBB trains. The ask for a meeting comes at PoC week 4–5, not before.

---

## Phase 4 — Fundraising Trigger (Month 6)

- [ ] **#17 Achieve pilot week 6 sign-off (Roland + Martin)**
  Composite pilot health score sign-off by Roland Ruiz and Martin Lerch. Required inputs: validated ROI numbers, conductor adoption rate (>3 new task types/week), ECM audit trail 100% write success, Dr. Fischer sign-off. **This is the fundable event.**

- [ ] **#18 Prepare investor pitch package**
  Assemble for Speedinvest/i5invest: (1) Deck — problem, solution, market (€290M, 12–24% CAGR), traction, business model (€12K/consist/year BYOK), competitive whitespace, team, ask; (2) Validated ROI one-pager (Roland's number × prevented delay-minutes); (3) Financial model; (4) Named reference — Roland willing to speak to investors; (5) Co-founder plan.
  Target: €300–600K pre-seed at €2–4M pre-money valuation.

- [ ] **#19 Request investor meetings at PoC week 4–5**
  With live pilot running, Roland's number in hand, week 6 sign-off imminent — reach out to Speedinvest and i5invest for first meetings. Pitch package and co-founder answer must be ready. Goal: close pre-seed within 2–3 months of week 6 sign-off (months 8–9 overall).

---

## Critical Path

```
Register Einzelunternehmen
        ↓
Sign PoC contract with ÖBB
        ↓
Deploy platform on 3 consists
        ↓
Get Roland's cost-per-delay-minute (week 1)
        ↓
Warm investor relationships (ongoing)
        ↓
Week 6 sign-off — Roland + Martin ← FUNDABLE EVENT
        ↓
Investor meetings (week 4–5 of pilot)
        ↓
Pre-seed close (months 8–9)
```

---

## Watch-outs

- **Liability:** Einzelunternehmen = no liability separation. Get professional liability insurance before PoC signs. Consider converting to GmbH before fleet contract.
- **Solo founder risk:** Speedinvest/i5invest will push hard on this. Have the co-founder answer ready before walking in.
- **EU AI Act:** High-risk classification likely. Advisory-only product posture already compliant. Conformity assessment needed pre-fleet — flag this in the pitch as a known item with a plan.
- **LLM cost cap:** Nomad absorbs LLM costs for PoC duration — make sure this cap is explicit in the PoC contract, not open-ended.
