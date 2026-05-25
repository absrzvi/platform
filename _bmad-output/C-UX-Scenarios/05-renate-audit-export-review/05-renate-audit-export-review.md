# 05: Renate's Audit-Export Review at Manday 15

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** 2
**Persona:** Dr. Renate Fischer — ECM Lead (Commercial Gatekeeper)

---

## Transaction (Q1)

**What this scenario covers:**
Pilot manday 15. Martin Lerch has brought Renate in for her review of the ECM shadow audit-trail capability. She reviews the export for Unit 4722 alongside the paper sign-off scan, walks through a `supersedes_id` correction example, and inspects the reconciliation queue to see how paper-vs-shadow divergences are surfaced and handled. Her question: is this audit-trail capability sound enough to be the authoritative ECM record in Phase 2?

---

## Business Goal (Q2)

**Goal:** Phase 2 readiness composite — Renate Fischer endorsement (one of five trust criteria)
**Objective:** Trigger Map Objective 5

---

## User & Situation (Q3)

**Persona:** Dr. Renate Fischer — Head of Vehicle Maintenance, ÖBB Technische Services (Commercial Gatekeeper)
**Situation:** Renate is in a conference room at ÖBB Technische Services, pilot manday 15 (mid-Sept / mid-Oct 2026). Martin Lerch presents the platform; Abbas (Nomad) is in the room. Renate has the platform export for Unit 4722 and the corresponding paper sign-off scan side by side on her laptop. She is reading both like a regulatory document, not a UI.

---

## Driving Forces (Q4)

**Hope:** Find that the shadow record is field-for-field reconcilable with paper, that corrections are explicit and append-only, and that divergences are surfaced and managed — so she can take the endorsement to her audit committee.

**Worry:** That the platform looks compliant but ducks one of the corner cases — supersedes, divergence, or non-IdP-bound identity — and her endorsement creates institutional risk for her.

---

## Device & Starting Point (Q5 + Q6)

**Device:** Desktop (Renate's ÖBB-issued laptop, browser; ÖBB SSO IdP authenticated)
**Entry:** Renate has the export URL for Unit 4722's shadow audit row. Martin pastes it in chat; she opens it.

---

## Best Outcome (Q7)

**User Success:**
Renate verifies (1) field-for-field reconciliation between shadow and paper, (2) IdP-bound identity, HMAC, hash chain, (3) the `supersedes_id` correction model on a worked example, and (4) the reconciliation queue + divergence-handling story. She is satisfied. She does not object. She conditions her Phase 2 endorsement on the 100% shadow-match rate holding across the full pilot.

**Business Success:**
Trust criterion #3 (Renate sign-off) — earned. Phase 2 readiness composite moves toward threshold. The reference-customer-for-audit-trail narrative becomes viable for fleet GTM.

---

## Shortest Path (Q8)

1. **Audit-trail export view — Unit 4722** — Renate opens the export URL. Document-style layout, paper-style header (Vehicle: 4722, Date: 2026-10-12, Directive: EU 2019/779), followed by a structured field grid: IntentPacket generated_at, brief opened_at, fault resolution chain, sign-off recorded_at (IdP: klaus.weber@oebb.at), vehicle state snapshot at sign-off, HMAC, prev_row_hash. Alongside (or below) the document, JSON view is one click away.
2. **Compare with paper** — Renate places the platform export next to the paper sign-off scan on her screen. She reads field-for-field. Time on platform: 05:53 (open), 06:08 (sign-off). Time on paper: 06:08 (signature). Fault chain: identical. IdP identity: matches the signature name. She nods.
3. **`supersedes_id` correction example** — Martin walks Renate to a prepared example. The export view shows a sign-off row, then below it a *correcting row* — same vehicle, later timestamp, `supersedes_id` field pointing to the original. Both rows visible. The original is *not* mutated; the correcting row is additive. The hash chain shows: original row's hash is preserved; correcting row's hash includes the original's hash in its chain.
4. **`supersedes_id` view detail** — Renate clicks the `supersedes_id` link on the correcting row. The view pivots to a two-pane comparison: original on the left, correcting row on the right, fields with differences highlighted. A "reason for correction" field is part of the correcting row.
5. **Reconciliation queue — divergence list** — Renate clicks through to the reconciliation queue view. Filtered to the pilot to date. Shows: total sign-offs (N), shadow-matched (N), pending reconciliation (0), divergences (1 over 14 days, resolved within 16h). Each divergence shows reason code + resolution status + IdP identity who resolved it.
6. **Divergence detail (the one that fired)** — Renate clicks the single divergence row. It surfaces: paper sign-off time 06:08, shadow record time 06:14, root cause "depot network delay, queued and synced on reconnect," resolution: "shadow row updated with supersedes_id pointing to a temporary placeholder, paper trail remains source of truth." She reads the reason. She nods.
7. **Renate's decision (external — not in platform)** — Renate closes the laptop. She tells Martin she has what she needs. Phase 2 endorsement conditional on 100% match rate holding across the pilot. Trust criterion captured. ✓

---

## Trigger Map Connections

**Persona:** Dr. Renate Fischer — ECM Lead (Commercial Gatekeeper)

**Driving Forces Addressed:**
- ✅ **Want W1:** Field-for-field paper-to-shadow reconciliation she can verify with her own eyes
- ✅ **Want W2:** IdP-bound identity, HMAC, hash chain — engineering rigour visible in the export
- ✅ **Want W3:** Clear `supersedes_id` correction model on a worked example
- ✅ **Want W4:** Divergence-handling story she can present to her audit committee
- ❌ **Fear F1:** Compliance theatre — the correction example is concrete, not hand-waved
- ❌ **Fear F2:** Nomad-controlled identity — IdP identity field visible and traceable to ÖBB SSO
- ❌ **Fear F3:** Phase 2 pre-emption — export labels rows "shadow record (companion to paper)" throughout

**Business Goal:** Trigger Map Objective 5 (Phase 2 readiness composite — Renate sign-off criterion)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 05.1 | `05.1-audit-export-view/` | Document-style header + structured field grid + JSON-on-demand | Renate compares with paper |
| 05.2 | `05.2-compare-with-paper/` | Side-by-side layout pattern; field-for-field alignment | Renate moves to correction example |
| 05.3 | `05.3-supersedes-example/` | Two-row view: original + correcting, hash chain links visible | Renate clicks supersedes_id link |
| 05.4 | `05.4-supersedes-detail/` | Two-pane comparison with diff highlighting + reason text | Renate moves to reconciliation queue |
| 05.5 | `05.5-reconciliation-queue/` | Pilot-to-date list with match rate, pending, divergences | Renate clicks divergence row |
| 05.6 | `05.6-divergence-detail/` | Single divergence — reason code, resolution status, IdP identity | ✓ |

---

## Open Questions (for Phase 4)

- **Export format export options** — PRD says PoC is structured JSON, copy-paste ready; BOOM integration deferred. Phase 4 needs to spec: is the document-style header a generated PDF, or HTML rendered in browser? Both? Renate's audit-committee artefact format matters.
- **Audit-committee-facing artefact** — Renate's presentation to her audit committee is a separate artefact downstream of this scenario. Is it: (a) the same export view; (b) a generated summary PDF; (c) Martin builds a deck from the export? Phase 4 spec.
- **Hash-chain verification UI** — should the export view show a "Verify integrity" button that recomputes the chain and confirms? Renate's audit committee will likely ask. Phase 4 spec.
- **`supersedes_id` reason taxonomy** — like Elena's DEFER reasons, the correction reason needs a controlled vocabulary that the audit committee finds palatable. Pre-Phase 2 blocker.
- **Pre-pilot blocker dependency** — ÖBB SSO IdP federation must be live by pilot manday 1 (Trigger Map). Renate's review at manday 15 depends on it. Confirm with Martin Lerch.
