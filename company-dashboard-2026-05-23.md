# Company Dashboard
**Date:** 2026-05-23
**Owner:** Abbas Rizvi
**Status:** Pre-incorporation. PoC not yet signed.

> This document is the single source of truth for company status across all non-technical functions.
> Update it when anything changes. Read it when you've been heads-down on product for too long.

---

## The One-Page Picture

```
WHERE WE ARE:         Pre-incorporation. Product built (Phase 1). PoC opportunity live.
WHERE WE'RE GOING:    Sign PoC → Run pilot → Raise pre-seed → Fleet contract → Scale.
THE CRITICAL PATH:    Incorporate → Sign PoC → Get Roland's KPI number → Week 6 sign-off
                      → Fundraise → Fleet contract → Hire.
TIME HORIZON:         6 months to fundable event. 12-18 months to fleet contract.
```

---

## 1. Legal & Corporate

**Status: 🔴 Not started — blocker for PoC contract**

| Item | Status | Action | Urgency |
|---|---|---|---|
| Legal entity incorporated | ❌ | UK Ltd or Austrian GmbH — decide and file | 🔴 Before PoC signs |
| Registered office / address | ❌ | Required for incorporation | 🔴 Before PoC signs |
| Bank account (business) | ❌ | Open after incorporation | 🔴 Before PoC signs |
| Shareholder structure | ❌ | Sole founder for now — document clearly | 🔴 Before PoC signs |
| IP ownership documented | ❌ | All IP must be owned by the company, not you personally | 🔴 Before PoC signs |
| PoC contract (with ÖBB) | ❌ | Must be signed by company entity, not individual | 🔴 Before PoC signs |

**Structure:** Sole proprietorship (Einzelunternehmen), Vienna, Austria.

---

## 2. Commercial & Sales

**Status: 🟡 PoC opportunity live, not yet signed**

| Item | Status | Notes |
|---|---|---|
| PoC opportunity | ✅ Live | Roland Ruiz (ÖBB) — 3 consists, 6 months |
| PoC pricing agreed | ✅ | €15,000 total (€5K/consist). Below procurement threshold. |
| PoC contract drafted | ❌ | Needs legal entity first |
| Fleet GTM pricing model | ✅ | €12,000/consist/year, BYOK, 3yr min, volume tiers |
| Commercial model document | ✅ | platform/commercial-model-2026-05-23.md |
| Roland's cost-per-delay-minute | ❌ | **Get in pilot week 1 — anchors entire ROI case** |
| Fleet contract pipeline | ⏳ | ~€470K ARR potential. 12-18 months away. |
| Second operator pipeline | ❌ | Target after ÖBB reference established |

**Key contacts:**
- Roland Ruiz — ÖBB Fleet Manager, PoC champion, ROI sign-off
- Martin Lerch — ÖBB operations, union conversation, consist count
- Dr. Fischer — ÖBB Head of Vehicle Maintenance, ECM compliance sign-off

**Pilot success gates (week 6):** Roland + Martin sign-off on composite pilot health score.

---

## 3. Finance

**Status: 🔴 No business finances exist yet**

| Item | Status | Notes |
|---|---|---|
| Business bank account | ❌ | After incorporation |
| PoC revenue (expected) | ⏳ | €15,000 — 50% on signature, 50% week 4 |
| Runway | ❌ | Unknown — no visibility yet |
| LLM cost absorption (PoC) | ⚠️ | Nomad absorbs LLM costs for PoC duration. Cap this. |
| Hardware cost (PoC) | ⚠️ | Hailo-8 + Hailo-10H M.2 cards supplied for PoC. Transfers to ÖBB on fleet contract. Budget this. |
| Pre-seed fundraising target | ❌ | TBD — set after PoC week 6 sign-off |
| Investor pipeline | ❌ | Start building during PoC |

**Funding options post-PoC:**
- UK Innovate UK (if UK-based) — rail + AI priority sectors
- EU Horizon Europe / Shift2Rail / Europe's Rail JU
- Pre-seed: Speedinvest (Vienna), Extantia, specialist deep tech funds

**The financial model to build (during PoC):**
- Monthly burn rate
- PoC cash flow (€7,500 on sign, €7,500 week 4)
- Fleet contract revenue model (ARR, payment terms)
- LLM cost pass-through estimates (€150–400/consist/month at PoC rates)

---

## 4. Compliance & Regulatory

**Status: 🟡 Product design compliant. Corporate compliance gaps.**

| Item | Status | Urgency |
|---|---|---|
| GDPR — Data Processing Agreement (DPA) | ❌ | ~£1-3K lawyer. Required before PoC signs. | 🔴 |
| EU 2019/779 ECM audit trail | ✅ | Built into PRD. First-class requirement. | — |
| NIS2 supply chain compliance | ⚠️ | Applies once on ÖBB network. BYOK reduces attack surface. | 🟡 |
| ISO 27001 certification | ❌ | Required for fleet procurement qualification. Not for PoC. | 🟡 During PoC |
| ISO 27001 gap assessment | ❌ | ~£5-15K consultant. Start during PoC. | 🟡 |
| Penetration test | ❌ | ~£5-20K. Required for fleet procurement package. | 🟡 Before fleet |
| EU AI Act — conformity assessment | ❌ | High-risk classification likely. Nomad must commission pre-fleet. | 🟡 Before fleet |
| EU AI Act — BYOK deployer/provider split | ⚠️ | Must be explicit in fleet contract. | 🟡 Fleet contract |
| EU Data Act — data ownership clauses | ⚠️ | Fleet contract must include data portability terms. | 🟡 Fleet contract |
| Incident response procedure | ❌ | 24h reporting under NIS2. Document and test. | 🟡 During PoC |

**Advisory-only product posture = already EU AI Act compliant on human oversight requirement. This was the right call.**

---

## 5. People & Organisation

**Status: 🔴 Solo founder. Single biggest risk.**

| Item | Status | Notes |
|---|---|---|
| Founder | ✅ | Abbas Rizvi — product + engineering |
| Co-founder | ❌ | **Most important hire/decision in next 12 months** |
| First hire profile needed | — | Commercial + compliance + customer success |
| Advisor(s) | ❌ | Rail industry advisor would help with operator relationships |
| Employment contracts template | ❌ | Needed before first hire |

**Co-founder criteria:**
- Owns commercial, compliance, customer success
- Complements technical founder (not another engineer)
- Ideally: rail industry background OR enterprise B2B SaaS sales
- Equity conversation: significant stake required to attract right person

**Where to find:**
- Rail industry networks (ERA, UIC events, InnoTrans)
- Enterprise B2B SaaS founders/operators looking for next thing
- ÖBB or operator alumni who understand the buyer

---

## 6. Product & Technology

**Status: ✅ Ahead of everything else — don't let this mask company gaps**

| Item | Status | Notes |
|---|---|---|
| Phase 1 (Hailo-8 vision pipeline) | ✅ Built | oebb-agent repo |
| PRD (unified platform) | ✅ Complete | platform/prd.md, 933 lines, 43 FRs |
| Commercial model | ✅ Complete | platform/commercial-model-2026-05-23.md |
| Market research | ✅ Complete | Competitive whitespace validated |
| Architecture doc | ❌ | Next step — bmad-create-architecture |
| Epics & stories | ❌ | After architecture |
| Phase 2 build | ❌ | After epics |
| Pre-story spikes | ❌ | Hailo-10H load test, pgvector P99, Kondukt ×4, dismissal arbitration |

---

## 7. Fundraising (Optional Path)

**Status: 🔵 Not started — target post-PoC week 6 sign-off**

> This is an option, not a requirement. The PoC is self-funded at €15K. Fundraising only makes sense if the fleet contract requires company infrastructure (ISO 27001, team, support SLA) that can't be bootstrapped from PoC revenue alone.

| Item | Status | Notes |
|---|---|---|
| Warm references at Speedinvest | ✅ | Use to build relationship now, don't ask for meeting yet |
| Warm references at i5invest | ✅ | Same — warm the relationship during PoC |
| PoC proof point ready | ❌ | Wait until week 4-5 of live pilot |
| Roland's validated ROI number | ❌ | Required before any raise conversation |
| Named customer reference (Roland) | ❌ | Strongest possible asset for investor conversation |
| Second operator in pipeline | ❌ | Not required but significantly strengthens case |
| Co-founder in place or credible plan | ❌ | Investors will push hard on solo founder risk |
| Financial model (ARR, burn, runway) | ❌ | Build during PoC |

**Target raise:** €300K–€600K pre-seed
**Target pre-money valuation:** €2–4M (defensible with live PoC + named pipeline)
**Expected dilution:** 15–25% depending on valuation negotiation
**Typical cheque size:** Speedinvest / i5invest write €200K–€1M at pre-seed

**What tips the valuation up:**
- Roland's cost-per-delay-minute number locked (converts ROI from range to precise figure)
- Week 6 sign-off in hand before the investor meeting
- Second operator conversation started
- Co-founder identified or term sheet to join post-close

**Approach sequence:**
1. **Now** — use references to get warm intros, build relationship, no ask yet
2. **PoC week 4–5** — request first meeting with proof point in hand
3. **Post week 6 sign-off** — run the raise process with full validation package
4. **Target close** — 2–3 months after week 6 (months 8–9 overall)

**What investors will challenge:**
- Solo founder risk — have a co-founder answer ready
- Rail procurement cycles are long — show the named pipeline and timeline
- Hardware dependency (Hailo) — explain why this is a moat not a risk
- EU AI Act exposure — show advisory-only posture as the answer

---

## 9. Marketing & Brand

**Status: 🔴 Nothing exists yet — not the priority right now**

| Item | Status | Notes |
|---|---|---|
| Company name | ❌ | Decide before incorporation |
| Domain | ❌ | After name decision |
| LinkedIn company page | ❌ | After incorporation |
| Website | ❌ | Minimum viable: one-pager. Not needed for PoC. |
| Thought leadership (LinkedIn) | ❌ | Start during PoC — rail AI + ÖBB pilot story |
| Conference presence | ❌ | InnoTrans (Berlin, biennial), RailTech Europe, Rail Live |

**Priority:** Marketing is not blocking anything right now. Focus on legal, PoC, and people first. Start LinkedIn presence mid-PoC when you have a live story to tell.

---

## 10. Partnerships & Business Development

**Status: 🔴 No formal partnerships**

| Item | Status | Notes |
|---|---|---|
| ÖBB relationship | 🟡 | PoC level. Roland is champion. Not yet a reference customer. |
| Nomad Digital relationship | ⚠️ | Separate company. Natural channel partner candidate for fleet. Relationship needs defining. |
| Systems integrator pipeline | ❌ | Wabtec, Speedcast, Icomera — approach post-PoC with live proof point |
| Second operator target | ❌ | DB, SBB, or SNCF — identify during PoC |
| Hardware partner (Hailo) | ⚠️ | Volume pricing TBD. Relationship exists via R5001C SYS2. |

---

## Critical Path Summary

```
NOW              Incorporate legal entity
                 Draft GDPR DPA (lawyer)
                 Decide company name
                       ↓
MONTH 1          Sign PoC contract with ÖBB
                 Open business bank account
                 Get Roland's cost-per-delay-minute (week 1)
                       ↓
MONTHS 1-6       Run PoC (3 consists)
                 Start ISO 27001 gap assessment
                 Build financial model
                 Start co-founder search
                 Document everything (ISO 27001 evidence base)
                       ↓
MONTH 6          Week 6: Roland + Martin sign-off
                 Validated ROI numbers locked
                 Fundable event
                       ↓
MONTHS 6-9       Raise pre-seed
                 Bring in co-founder
                 Start fleet contract negotiation
                       ↓
MONTHS 9-18      Fleet contract signed
                 ISO 27001 certification
                 First hire(s)
                 Second operator pipeline
```

---

## Weekly Review Prompt

Ask yourself these questions once a week:

1. **Legal:** Is the company incorporated? Is the DPA signed?
2. **Commercial:** Have I spoken to Roland this week? Is the PoC on track?
3. **Finance:** Do I know my burn rate? Is PoC cash in the bank?
4. **Compliance:** Am I documenting everything for ISO 27001?
5. **People:** Have I had one conversation toward finding a co-founder?
6. **Product:** Am I building what the PRD says, or have I gone off-piste?

If you can't answer these, the dashboard needs updating.
