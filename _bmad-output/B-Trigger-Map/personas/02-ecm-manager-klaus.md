# Persona Detail: Klaus the ECM 1 Manager

**Priority:** Primary (regulatory) — owns >80% pre-arrival review rate + 100% shadow-match rate
**Real-world equivalent:** Existing ÖBB ECM 1 Managers at Wien Westbahnhof and other depots
**Surface:** Depot Briefing PWA (primary), ECM shadow audit (write), reconciliation queue (read)

---

## Profile

Klaus is an ECM 1 Manager at a Wien depot. He is accountable under EU Directive 2019/779 for every vehicle that returns to service. He arrives cold — he wasn't on shift when the train came in last night. His current process: walk to the bay, ask the technician what happened, read a paper fault log, make a judgment call, sign on paper. If he misses something and the vehicle fails in service, the liability is his and the paper trail is thin.

**The MVP does not change this process.** It adds: a structured brief 14 minutes before HAFAS scheduled arrival, role-filtered to ECM Manager (executive summary depth), and a shadow audit record alongside his paper sign-off using the same hold-to-record gesture.

## Behavioural anchors

- **Liability-conscious** — every action is filtered through "what if this comes back at me"
- **Process-conservative** — paper is the trusted artefact; anything that competes with it for authority is suspect
- **Bay-walk loyal** — talking to the technician on site is part of how he gathers signal; the brief informs, doesn't replace
- **Cross-shift** — he arrives cold, often hours after the fault occurred and the technician work happened

---

## Positive Drivers (Wants)

### W1. Arrive at the bay already knowing what failed, who resolved it, what was deferred
The pre-arrival brief is the gift. Klaus walks to the bay with context — detection time, conductor ACK, technician assignment, part fitted, test result, role-filtered to the executive-summary depth he needs. He still confirms with the technician on site; the brief makes that conversation faster and sharper.

### W2. Have a digital shadow that proves what he actually authorised, if the paper is ever disputed
Paper can be lost, damaged, illegible. Klaus's signature on paper is the regulatory record under EU 2019/779, but the shadow trail — IdP-bound identity, UTC timestamp, full vehicle state snapshot at sign-off, directive reference, HMAC, hash chain — is the *evidence* that protects him if the paper trail is ever challenged.

### W3. Be the persona whose process didn't change but whose downside risk dropped
Klaus didn't ask for a new tool; he doesn't want to learn one. The MVP respects that. He gets a brief on his phone and a recording gesture — that's the whole UI commitment. The risk reduction is *because* nothing else changed.

### W4. Build the 100% shadow-match evidence that lets Renate sign off Phase 2
Klaus is part of a six-month dataset. Every paper sign-off he produces has a matching shadow row. That dataset is what Renate Fischer reviews. Klaus's clean record is the operational input to the commercial gate.

---

## Negative Drivers (Fears)

### F1. Paper-vs-shadow ambiguity at the moment of regulatory clearance
If the UI grammar lets a colleague (or an internal auditor reading the platform) think the shadow is the sign-off, Klaus's regulatory position destabilises. The platform must say *shadow*, *companion*, *record of* — never *sign-off* in a way that competes with the paper.

### F2. A platform that, by being present, gets him asked to sign off faster than he's comfortable
Implicit pressure from the brief being open and the gesture being available. Klaus must control the pace. The platform shows the brief and waits; nothing in the UI ticks down toward a decision.

### F3. A shadow record that diverges from the paper, and the divergence comes back at him later
Divergences are surfaced within 24h, reconciled within the shift, and persistent divergence downgrades the Phase 2 readiness composite. Klaus needs to see his divergence count, not just have it stored somewhere.

### F4. Any pressure (real or perceived) to skip walking to the bay and confirming with the technician on site
If the brief is too good, Klaus risks being asked "why did you still walk to the bay if the brief covered everything?" The platform's voice must make clear: *the brief informs, the bay walk and the technician confirmation remain the authoritative inputs to the sign-off.*

---

## Must-address design implications

| Driver | Design implication |
|---|---|
| W1 | Brief opens at HAFAS scheduled arrival − 14 minutes, push-notified to Klaus's PWA. Role-filtered: ECM Manager view = executive summary (what failed, impact, recommended action), not technical detail. |
| W2 | Shadow audit record captures: IdP identity, UTC timestamp, vehicle state snapshot, directive reference EU 2019/779, HMAC, hash chain link. Export is structured JSON, copy-paste ready. |
| W3 | Depot Briefing PWA is mobile-first, role-selector is one tap, no confirmation, clears after 4 hours on shared device. No onboarding flow, no settings page, no preferences beyond role. |
| W4 | Reconciliation queue is visible to Klaus on his shift — "your sign-offs this shift, shadow status: matched / pending / divergent." |
| F1 | UI verbs: "Record sign-off" never "Sign off." Visual grammar: shadow row is visually subordinate to paper representation. Renate's audit export labels the shadow row as the *companion record* to the paper. |
| F2 | No countdowns or timers on the brief view. The 5-second countdown applies only after Klaus initiates the gesture — not before. |
| F3 | Divergence count is in Klaus's view, with reason codes and resolution status. He sees his own data, not just an aggregate. |
| F4 | Brief includes a "confirmation step" prompt — "Confirm with technician on site before recording sign-off" — not as a forced gate, but as a reinforcement. |

---

## Open questions

- **Shared-device identity** — Klaus is on a personal phone; Elena (and other ECM 4 techs) is sometimes on a shared depot device. Klaus's identity model is simpler (always personal). But the depot device's role-selector behaviour affects whether Klaus's brief ever appears on it (it shouldn't — he's never the role on the shared device). UX scenario phase needs to lock the cross-device boundary.
- **Paper trail digitisation** — the paper is the sign-off. Does Klaus scan and attach to the platform? Or is the paper a parallel artefact stored physically? MVP scope is the latter; Phase 2 may absorb the paper into the platform's record entirely.
- **Cross-depot view** — Klaus is depot-local but his peers at other depots run the same process. Does Klaus see aggregate divergence stats across depots, or only his own? Privacy + competitive dynamics between depots matters here.
