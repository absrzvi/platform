# Commercial Model — ÖBB Unified Rail Operations Platform
**Date:** 2026-05-23
**Status:** Internal — not for distribution without review
**Owner:** Abbas Rizvi / Nomad Digital

---

## Pricing Unit

**1 train consist = 1 platform fee.**

One R5001C SYS2 per consist, one Hailo-8 + Hailo-10H M.2 pair per consist. Platform fee does not scale with wagon/car count. A locomotive + 8 coaches = 1 consist = 1 fee. Consist defined as the traction unit and all attached coaches operating as a single service.

---

## PoC Pricing (Option A — Flat per consist)

For the ÖBB 3-consist pilot.

```
Platform fee:    €5,000 per consist (6-month PoC, all-in)
                 × 3 consists
                 = €15,000 total fixed fee

Includes:        Conductor App, Depot Briefing Interface,
                 Fleet Manager AI Interface, Rail MCP Server,
                 ECM audit trail, onboard inference
                 (Hailo-8 vision + Hailo-10H LLM)

LLM costs:       Absorbed by Nomad Digital for PoC duration
                 (capped — actual usage data feeds fleet pricing)

Hardware:        Hailo-8 + Hailo-10H M.2 cards supplied for PoC
                 Ownership transfers to ÖBB on fleet contract execution

Term:            6 months
Payment:         50% on contract signature, 50% on pilot week 4
Success gate:    Composite pilot health score — Roland Ruiz +
                 Martin Lerch sign-off at week 6
```

**Rationale:** €15,000 total is below ÖBB's internal procurement committee threshold for direct sign-off. Roland can approve without a formal tender process. Low enough to be taken seriously; high enough to signal commercial intent.

---

## Fleet GTM Pricing (Option C — Platform fee + BYOK pass-through)

For rollout beyond the ÖBB PoC.

```
Platform fee:    €12,000 per consist per year
                 (subject to upward revision once ÖBB cost-per-
                  delay-minute figure confirmed with Roland Ruiz)

Includes:        All modules — Conductor App, Depot Briefing,
                 Fleet Manager AI Interface, ECM audit trail,
                 Rail MCP Server, onboard inference

Hardware:        Hailo-8 + Hailo-10H M.2 — operator choice:
                 (a) One-time per consist (cost TBD at volume)
                 (b) Amortised into platform fee over contract term

LLM costs:       Operator-direct via BYOK API key
                 Nomad provides cost simulator pre-contract
                 Estimated €150–400/consist/month at PoC usage rates
                 (to be validated against 6-month PoC data)

Contract term:   3 years minimum
                 Annual uplift: CPI + 2%
                 Extension options: 1+1 years at fixed uplift

Volume tiers:
  1–10 consists:    €12,000/consist/year
  11–30 consists:   €10,000/consist/year
  31+ consists:     €8,500/consist/year (negotiated)

Support SLA:     Included — depot-dwell OTA updates,
                 99.5% platform uptime commitment
                 (Hailo hardware SLA separate — Nomad hardware agreement)
```

**ÖBB full fleet scenario:** ~47 regional train consists × €10,000 (11–30 tier) = **€470,000 ARR**. At 31+ tier: **~€400,000 ARR**. Reference customer value: multiples of this when used to close second operator.

---

## ROI Framework

### The Argument

The platform is priced against **prevented delay-minutes** — the KPI the Disponent (Fleet Manager) is already measured on. This is not a feature sale; it is a counterfactual evidence sale.

### Calculation Inputs

| Input | Value | Source |
|---|---|---|
| EU passenger delay cost | €1–3 per passenger-minute | EU transport appraisal guidelines |
| ÖBB average consist load | 200–300 passengers (regional service) | Estimate — confirm with Roland |
| ÖBB internal cost-per-delay-minute | **TBD — get from Roland in pilot week 1** | ÖBB internal KPI |
| Pilot fault detections (6 months, 3 consists) | 7 pre-disruption detections | PRD Journey 9 baseline |
| Historical mean delay, same fault class | 23 minutes | PRD Journey 9 baseline |

### Prevented Cost Calculation (Pilot Period)

```
Prevented delay-minutes:    7 faults × 23 min = 161 minutes

Conservative estimate:
  161 min × 200 passengers × €1.00 = €32,200 prevented cost
  Over 3 consists, 6 months

Mid estimate:
  161 min × 250 passengers × €2.00 = €80,500 prevented cost
  Over 3 consists, 6 months

Per consist per year (extrapolated):
  Conservative: €21,000–€27,000/consist/year prevented
  Mid estimate: €53,000–€80,000/consist/year prevented
```

### Pricing vs. ROI

| Platform fee | % of prevented cost captured (conservative) | % of prevented cost captured (mid) |
|---|---|---|
| €8,000/consist/year | 30–38% | 10–15% |
| €12,000/consist/year | 44–57% | 15–23% |
| €15,000/consist/year | 56–71% | 19–28% |

**At €12,000/consist/year, the platform captures 15–57% of the value it creates.** The remaining 43–85% stays with ÖBB — a strong value-share argument for procurement.

### The Number to Get from Roland (Pilot Week 1)

> "What is ÖBB's internal cost-per-delay-minute figure used for Disponent KPI reporting?"

This single number:
1. Converts the ROI argument from estimated ranges to a precise ÖBB-validated figure
2. Makes Roland a co-author of the commercial case — he cannot object to his own number
3. Unlocks the counterfactual evidence panel in the Fleet Manager interface (Journey 9)
4. Is the input for the prevented delay-minutes methodology agreed with Roland before pilot week 3

---

## Commercial Pitch — Three-Layer Value Stack

*For the Roland + Martin + Dr. Fischer rollout decision room.*

**Layer 1 — Regulatory compliance (Dr. Fischer / Head of Vehicle Maintenance)**
- ECM audit trail under EU 2019/779 — required by law, not a feature
- Budget line: risk and compliance, not operations
- Pitch: "We give your ECM Manager a legally defensible sign-off record that survives regulatory inspection. Every return-to-service is documented, tamper-evident, and IdP-bound."
- Close: Dr. Fischer sign-off at pilot week 3 → reference customer for ECM compliance

**Layer 2 — Fleet Manager KPI alignment (Roland / Disponent)**
- Prevented delay-minutes maps to the Disponent's existing performance metric
- Pitch: "We give your fleet managers the counterfactual evidence they currently cannot produce. Here is the number: [Roland's cost-per-delay-minute × 161 minutes prevented]."
- Close: ROI positive within 2 months of full fleet deployment at mid estimate

**Layer 3 — Zero displacement risk (Martin / union conversation)**
- No existing workflow replaced for conductors — entirely additive
- Pitch: "Conductors have no existing digital tools for onboard decisions. This platform adds capability without replacing anything. Clean adoption story, clean union conversation."
- Close: Conductor adoption rate from pilot (target: >3 new conductor-initiated task types per week)

---

## Competitive Positioning on Price

| Competitor | Pricing model | Comparable unit price | Gap |
|---|---|---|---|
| Railnova | Per traction unit/year | ~€3,600 implied | Freight only, no inference, no LLM, no conductor UI |
| KONUX | Per switch asset/year | ~€4,500 implied | Infrastructure only, cloud, no rolling stock |
| Siemens Railigent X | Per-vehicle DaaS, bundled | Undisclosed | Cloud, no ECM, bundled with rolling stock purchase |
| Palantir AIP | Per operator/year | $1M–$10M+ | Cloud, no edge inference, no ECM module |
| **ÖBB Platform** | **Per consist/year** | **€12,000** | **Full stack — inference + ECM + LLM + role UX** |

**The €12,000/consist/year price point sits between Railnova's freight-focused SaaS (~€3,600) and Palantir's enterprise platform ($1M+/year). It is premium to Railnova because the product is materially more capable. It is accessible where Palantir is not.**

---

## Open Commercial Questions

| Question | Owner | When needed |
|---|---|---|
| ÖBB internal cost-per-delay-minute | Roland Ruiz | Pilot week 1 |
| ÖBB consist count (regional fleet) | Martin Lerch | Pre-fleet proposal |
| Hardware volume pricing (Hailo M.2 at fleet scale) | Nomad procurement | Pre-fleet proposal |
| BYOK cost simulator (actual PoC LLM usage data) | Platform team | Post-PoC, pre-fleet contract |
| ÖBB SSO IdP integration cost (per-operator tax) | Martin Lerch / Nomad IT | Fleet contract negotiation |
| Contract template — registry ownership, credential rotation | Nomad commercial | Pre-fleet gate |
| ISO 27001 certification timeline | Nomad compliance | Pre-fleet procurement qualification |
| EU AI Act conformity assessment timeline | Nomad legal | Pre-fleet rollout |
