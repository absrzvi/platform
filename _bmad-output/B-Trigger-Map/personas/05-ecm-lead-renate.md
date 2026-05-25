# Persona Detail: Dr. Renate Fischer (ECM Lead, Head of Vehicle Maintenance, ÖBB Technische Services)

**Priority:** Commercial Gatekeeper — one-time reviewer at pilot manday 15; her endorsement is one of five Phase 2 trust criteria
**Real-world equivalent:** Dr. Renate Fischer (or her institutional equivalent at ÖBB TS)
**Surface:** ECM shadow audit-trail export (read), reconciliation queue (read), structured JSON export

---

## Profile

Renate is not a daily user. She is the ECM-accountable executive — Head of Vehicle Maintenance at ÖBB Technische Services. She was not in the pilot planning meeting; Martin Lerch brings her in at approximately pilot manday 15.

Her question is **not** "does it work?" — it is "is this audit-trail capability sound enough that I would be willing, after pilot end, to make it the authoritative ECM record under EU Directive 2019/779?"

She has the institutional authority to say yes or no to Phase 2 commercial ascension. She also has an audit committee she has to present to. Her sign-off is one of five composite trust criteria.

## Behavioural anchors

- **Audit-committee-facing** — every artefact she reviews has to survive her own internal review process
- **Reputation-stake** — her name on a Phase 2 endorsement is her credibility
- **One-time engagement** — she sees the platform at pilot manday 15 and at pilot end; she is not a UAT participant
- **Specifically interrogates corner cases** — supersedes, divergence, identity sovereignty

---

## Positive Drivers (Wants)

### W1. Field-for-field paper-to-shadow reconciliation she can verify with her own eyes
Renate opens the shadow audit-trail export for Unit 4722 alongside the paper sign-off scan. She compares: IntentPacket generated at 05:50, shadow opened by Klaus at 05:53, fault resolution chain matches paper, sign-off at 06:08 (Klaus's IdP identity), state snapshot, directive reference, HMAC, hash chain. **Every field that's on the paper is on the shadow row, identical.** If yes, she has something to show her audit committee.

### W2. IdP-bound identity, HMAC + hash chain, append-only enforcement — engineering rigour she can stake her name on
Renate's audit committee will ask: "How do you know the record wasn't tampered with after the fact?" She needs the answer to be technical and verifiable, not procedural: append-only enforced at database level (trigger on UPDATE/DELETE), HMAC per row, hash chain `sha256(prev_row_hash || row_hmac)` linking each row to the previous.

### W3. A clear correction model (`supersedes_id`) that doesn't mutate the past
The compliance-correct way to fix an error is a new row pointing to the corrected one, never mutating the original. Renate needs to see this and walk through it on a concrete example. The platform's correction story is the rebuttal to "what if someone signs off wrong?"

### W4. A divergence-handling story she can present to her audit committee
Advisory-only invites a new question: "What happens if the paper says one thing and your shadow says another?" Renate needs to see the reconciliation queue, the 24h divergence flagging, the persistent-divergence Phase 2 downgrade rule. Divergence is not silenced — it is *surfaced and managed*. That's the story.

---

## Negative Drivers (Fears)

### F1. A platform that looks compliant but ducks the supersedes / correction question
If Renate asks "what happens when someone signs off incorrectly" and the answer is hand-wave-y, she stops the conversation. She needs to see the actual mechanism on screen, not be told about it.

### F2. Audit trail tied to a Nomad-controlled identity rather than ÖBB's IdP (sovereignty concern)
If the audit row's identity field is a Nomad-issued credential rather than ÖBB's IdP, the audit trail's authority is mediated by Nomad's continued cooperation. That's a non-starter for ÖBB's regulatory position. IdP-bound = ÖBB-bound = Renate can defend it.

### F3. Any pre-emption of Phase 2 in MVP scope — anything that smells like "we're already the record of authority, you just haven't noticed"
Renate's institutional caution: an advisory MVP that gradually starts acting authoritatively without explicit re-authorisation is the worst-case for her. The advisory-only contract has to be visible in the platform's voice, the export's labelling, and the reconciliation surface.

### F4. Export format that her documentation system can't ingest at fleet rollout
At PoC scale: structured JSON copy-paste-ready is acceptable (BOOM integration deferred — noted, not a blocker). At fleet rollout: BOOM (or whatever has replaced it) must be a target, with a defined schema. Renate's reference-customer endorsement assumes a workable export path at scale.

---

## Must-address design implications

| Driver | Design implication |
|---|---|
| W1 | ECM shadow audit-trail export view: paper-style layout alongside structured JSON, every field labelled, source-traced. Renate can read it as a document, her audit-committee analyst can read it as data. |
| W2 | Audit row schema includes: HMAC, prev_row_hash, IdP issuer + subject, directive reference. Export includes the *chain proof* — a verifier can confirm the row is consistent with the row before. |
| W3 | `supersedes_id` correction surface: Renate is shown a specific example (original row + correcting row + the relationship), not just told the feature exists. |
| W4 | Reconciliation queue view: shows divergence count per pilot week, reason codes, resolution status, persistent-divergence flag against Phase 2 readiness threshold. Renate sees the *operational reality*, not a sanitised summary. |
| F1 | Correction example is in the demo script for pilot manday 15. Pre-scripted, not improvised. |
| F2 | IdP integration is a pre-pilot blocker — ÖBB SSO IdP federated by pilot manday 1. Audit rows reject any non-IdP-bound identity. |
| F3 | Every export and reconciliation surface labels rows: *shadow record (companion to paper)* in MVP; pre-emptive language (*authoritative*, *of record*) is forbidden until Phase 2 commercial release. |
| F4 | Export schema documented. Post-pilot integration target (BOOM or successor) named in Phase 2 commercial proposal. Schema designed to be ingest-able by a documented standard format. |

---

## Open questions

- **Outside-auditor tabletop exercise** — Renate's endorsement is one of five trust criteria; outside-auditor tabletop verdict (mandays 20–25) is another. Both feed her audit-committee presentation. Coordinate the tabletop output format with her review needs.
- **Audit committee artefact** — does she want a written report, a live demo, a deck? The pilot manday 15 review format should be confirmed with Martin Lerch before manday 1.
- **Cross-operator IdP model** — at fleet rollout to non-ÖBB operators, the IdP is a different one each time. The audit-row identity field must support multi-IdP federation without weakening the chain proof. Architecture spike before fleet rollout.
- **Renate's continued engagement post-pilot** — is she a one-time endorser, or does she become an ongoing reviewer in Phase 2? Affects the design of the recurring audit-committee artefact.
