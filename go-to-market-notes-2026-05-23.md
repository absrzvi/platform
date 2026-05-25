# Go-to-Market & Company Building Notes
**Date:** 2026-05-23
**Status:** Internal — strategic notes, not for distribution
**Owner:** Abbas Rizvi

---

## Context

One-person company. Platform is separate from Nomad Digital. Long-term intent: build and run the business, not exit.

---

## Regulatory Reality Check

### What triggers ISO 27001 requirement
Not specific product features — supplier status. The moment Nomad has any footprint on ÖBB's operational network, ÖBB's supplier security assessment applies regardless of product design. Triggered by:
- Nomad employees accessing ÖBB systems for deployment/support
- OTA updates shipped to onboard hardware on active trains
- Credentials held for ÖBB SSO IdP integration
- Platform processing data classified as operational railway data under NIS2

You cannot design your way out of this requirement.

### What actually works without full ISO 27001 certification
For PoC: not required. Roland approves directly, below procurement committee threshold.

For fleet contract negotiation, this package often satisfies procurement qualification:
- Penetration test from reputable firm
- Data processing agreement (DPA) under GDPR
- Documented incident response procedure with named contacts
- Evidence of access control policies
- Credible ISO 27001 roadmap with committed certification date

### Key regulatory impacts on the platform

| Regulation | Impact | Status |
|---|---|---|
| EU AI Act | High-risk classification likely. Conformity assessment required pre-fleet. Advisory-only posture already compliant with human oversight requirement. | ✅ Product design sound. Nomad corporate action needed pre-fleet. |
| NIS2 | Rail operators critical infrastructure. Supplier chain requirements apply to Nomad. 24h incident reporting. ISO 27001 fastest path to ÖBB supplier qualification. | 🔴 Gap. Mitigated by BYOK architecture reducing Nomad attack surface. |
| EU 2019/779 (ECM) | Audit trail requirement. Already first-class PRD requirement. | ✅ Done. |
| EU Data Act | ÖBB has rights over data generated on their trains. BYOK model helps — data stays in ÖBB's API call chain. Fleet contract needs explicit data ownership clauses. | 🟡 Contract action needed. |
| GDPR | DPA required before any data touches Nomad systems. One-time legal cost ~£1-3K. | 🔴 Do before PoC signs. |

### EU AI Act — BYOK deployer/provider split
When ÖBB brings their own API key, they become the "deployer" under the Act. Nomad remains the "provider." Both have obligations. Must be explicit in fleet contract.

---

## Company Building Path (Long-Term Intent)

### What the PoC buys
Six months of live validation on a national operator's trains. With Roland + Martin sign-off at week 6:
- Reference customer (PoC scale)
- Validated ROI numbers (Roland's cost-per-delay-minute × prevented delay-minutes)
- Named pipeline (~€470K ARR fleet potential)
- Working product on production hardware

That is a **fundable event**.

### What to build in parallel during the PoC (not ISO 27001)

1. **Legal entity** — UK Ltd or Austrian GmbH. PoC contract must be signed by a company, not an individual. Do this before PoC signs.

2. **Data processing agreement template** — lawyer drafts once, ~£1-3K. Required before any ÖBB data touches systems.

3. **Co-founder or first hire** — owns commercial, compliance, customer success. You own product and engineering. Most important decision in next 12 months.

4. **Document everything during PoC** — every decision, integration, incident. Becomes ISO 27001 evidence base later. Also makes the company acquirable if plans change.

### Funding conversation (post-PoC proof point)
Live PoC on ÖBB + validated ROI + fleet pipeline = fundable story for:
- **UK Innovate UK** — rail and AI both priority sectors (if UK-based)
- **EU Horizon Europe** — Shift2Rail / Europe's Rail JU rail calls
- **Pre-seed rail-focused investors** — Speedinvest (Vienna-based, rail-adjacent), Extantia, specialist deep tech funds

€15,000 PoC fee is the proof point, not the business model.

### Channel partner route (if long-term solo becomes untenable)
If company building stalls, the fallback is a channel partner who carries procurement qualification:
- Wabtec / Speedcast / Icomera — onboard connectivity vendors, already ÖBB-qualified
- Thales / Alstom digital services — larger, slower, actively acquiring rail software IP
- Nomad Digital — most natural fit despite being separate; knows R5001C hardware

Not the preferred path given long-term intent, but worth keeping in mind.

---

## Key Actions Before PoC Signs

| Action | Cost | Urgency |
|---|---|---|
| Incorporate legal entity | ~£500-1K | 🔴 Before PoC contract |
| GDPR data processing agreement | ~£1-3K (lawyer) | 🔴 Before PoC contract |
| Get ÖBB cost-per-delay-minute from Roland | £0 | 🔴 Pilot week 1 |
| ISO 27001 gap assessment | ~£5-15K | 🟡 During PoC, before fleet negotiation |
| Pen test (for fleet procurement package) | ~£5-20K | 🟡 Before fleet contract |
| Identify co-founder candidate | — | 🟡 During PoC |

---

## What Doesn't Change

The product design is sound. The market gap is real. The competitive whitespace (offline inference + ECM audit trail + BYOK + role-differentiated UX) is validated. The regulatory analysis strengthens the ECM and advisory-only design decisions already made.

The problem to solve is the go-to-market wrapper, not the product.
