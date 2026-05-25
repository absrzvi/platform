# Persona Detail: Elena the ECM 4 Technician (Electrical)

**Priority:** Secondary (operational) — ACK loop closer; Klaus's pre-arrival brief depends on her log
**Real-world equivalent:** Existing ECM 4 technicians across Electrical / Mechanical / IT specialisms
**Surface:** Depot Briefing PWA (write — ACK + DEFER)

---

## Profile

Elena is on the depot floor — Electrical specialism. She fits replacement parts, runs tests, deals with vendor portals when SNMP traps fire. Her current "logging" is a clipboard sheet and a phone call to whoever's covering ECM 1. She's competent and her tools — multimeters, vendor portals, the depot's diagnostic kit — are not negotiable. She doesn't want a new screen unless it cuts steps.

Three ECM 4 specialisms share the platform with her: Mechanical, IT, and (in some depots) a generalist. They all see the same card schema; they differ in `role_relevance` filtering and sort order. Card depth is technical (fault code, port, last-known-good state, diagnostic step) — not the executive plain-English summary Klaus sees.

## Behavioural anchors

- **Tool-conservative** — adopts new tools only when they're additive, not displacing
- **Specialism-filtered** — IT alerts in her queue are noise; electrical alerts in IT's queue are noise too
- **Time-pressured** — depot turnaround windows are tight; logging overhead competes with actual repair
- **Mixed device context** — shared depot device some shifts, personal phone others

---

## Positive Drivers (Wants)

### W1. A role-filtered view that doesn't drown her in brake alerts when she's working a camera firmware ticket
Card visibility is governed by `role_relevance[]`. Elena (Electrical) sees electrical-system cards first, mechanical and IT filtered or hidden. Brake wear alerts aren't in her queue at all — not hidden, not present.

### W2. Log an ACK once and have it propagate to Klaus's brief automatically — no phone call
Today she calls the ECM 1 cover and reads the fault number and her action. Tomorrow she taps ACK on a card and Klaus's pre-arrival brief shows her resolution chain before the vehicle docks. One step replaces two.

### W3. DEFER something with a reason logged, instead of "yeah I told someone"
A non-blocking warning Elena defers to the next maintenance window — she logs the deferral with a reason ("approaching threshold, schedule with next G-Check"). The reason is captured, attached to her IdP identity, surfaced in Klaus's brief as `WARNING — deferred by Elena Müller at 03:22, reason: ...`. Accountability without overhead.

### W4. Card depth tuned to her, not Klaus
Klaus sees "Door motor fault, Car 4 Door 2L — resolved." Elena sees "DOOR_MOTOR_FAULT, Car 4 Door 2L, fault code EM-447, last known good 02:11, replacement part 78-2299-A fitted at 03:22, test result PASS." Same fault, two views.

---

## Negative Drivers (Fears)

### F1. A shared depot device where she has to re-select her role every shift (friction → adoption drops)
The PRD specifies: shared devices = role selector on every open (4 large touch targets, one tap, no confirmation, clears after 4 hours); personal phones = stored preference. Elena's friction tolerance is *one* role tap per shift, max. Anything more and she'll fall back to clipboard.

### F2. Being held accountable for an ACK she didn't make (identity confusion on shared devices)
If the shared depot device caches her role but not her identity, and the next tech taps ACK on her open session, the wrong IdP gets attached to the audit trail. This is a regulatory-grade defect — Klaus's brief and Renate's audit export both become unreliable.

### F3. A platform that asks her to log structured detail she'd otherwise scribble on a clipboard in 3 seconds
If logging a resolution takes 30 seconds in the PWA vs 3 seconds on paper, Elena will do the paper and the platform will be empty. The card schema's `technical_detail` field must be quick to fill — defaults, dropdowns, paste-from-vendor-portal patterns — not a free-text essay.

### F4. Card visibility rules that hide something she needed to see
Over-filtering creates blind spots. If a cross-domain fault (e.g., camera power fault spanning Electrical + IT) is tagged only Electrical, IT misses it. If tagged only IT, Elena misses it. `role_relevance[]` must support multi-tag, and the ECM Manager-signed-off taxonomy (open decision Q1 in `ux-decisions-2026-05-23.md`) is the binding contract.

---

## Must-address design implications

| Driver | Design implication |
|---|---|
| W1 | Card sort order for Electrical role: electrical systems first (power, cameras, doors). Card visibility: brake wear alert not present, not hidden. |
| W2 | ACK action is one tap + IdP-bound timestamp. Propagation to Klaus's brief is automatic — Klaus does not pull, Elena does not push twice. |
| W3 | DEFER action is one tap → reason selector (dropdown of common reasons + free-text option) → IdP-bound timestamp. Reason is required, not optional. |
| W4 | Card schema: `summary` (plain English for Klaus) and `technical_detail` (technical for Elena) are separate fields, both generated by LLM at card creation. Tone differentiation is a prompt requirement, not a frontend concern. |
| F1 | Shared device: role selector clears after 4 hours, not at session end (multi-shift coverage). Personal phone: role stored after first selection, no re-prompt. |
| F2 | Shared device requires IdP login per ACK — not per session. Role is shared, identity is per-action. Audit row binds to the IdP that made the action, never the device's last-active role. |
| F3 | `technical_detail` field on cards: pre-populated from IntentPacket where possible. Vendor-portal-aware fields (port number, MAC, firmware version) accept paste with auto-parsing. |
| F4 | `role_relevance[]` is multi-tag. Cross-domain faults appear in every relevant queue. ECM Manager taxonomy sign-off is a pre-pilot blocker (lift the open decision from `ux-decisions-2026-05-23.md`). |

---

## Open questions

- **Sort order per role** — open decision in `ux-decisions-2026-05-23.md` Q1. Needs ECM Manager sign-off before Phase 3 scenarios lock.
- **Card depth toggle** — should Elena be able to see the executive summary view (Klaus's depth) if she wants, or is her view technical-only? Cross-role transparency may help cross-domain debugging.
- **Vendor portal integration depth** — does the PWA deep-link into the vendor portal, or just accept paste? MVP is paste; deep-link is post-MVP but design pattern should support upgrade.
