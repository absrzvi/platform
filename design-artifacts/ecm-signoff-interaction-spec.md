# ECM Sign-Off Interaction Spec

**Date:** 2026-05-25
**Author:** Freya (WDS Designer), ratified in party-mode session (Sally, Winston, Mary, Freya)
**Status:** Decided — pending one ECM 4 Technician timing walkthrough pre-go-live
**Surface:** Depot Briefing PWA — ECM Manager sign-off flow
**Regulatory context:** EU Directive 2019/779 — append-only, hash-chained, tamper-evident, IdP-bound

---

## MVP vs Phase 2 status

Per the advisory-only pivot (PRD §G21–G22, §M1, §I5), this gesture records the platform's **shadow sign-off** alongside the paper sign-off in MVP. Paper remains the authoritative ECM record throughout the 6-month pilot. In Phase 2 (gated on the five trust criteria in PRD §12), the same gesture is promoted to write the authoritative ECM record. The interaction itself is identical in both phases — only the legal status of the resulting record changes.

The "ceremony" framing below is the Phase 2 target. In MVP, the ceremony is rehearsal: operators build the gesture muscle memory under low-stakes conditions where the paper trail is the safety net.

---

## Design Principle

The sign-off card is not a form. It is a ceremony. Klaus is making a legally binding attestation in Phase 2 — a shadow record of that same attestation in MVP. Every design decision below flows from that framing.

---

## Interaction Model: Option C

**Hold-to-initiate (3s) → 5-second countdown → single atomic write at expiry**

Two deliberate phases:
- Hold says "I am ready to begin"
- Countdown gives "I am watching this complete"

Client-side timer only. No server-side pending state. Single atomic write at countdown expiry.

---

## Gesture Parameters

| Parameter | Value | Rationale |
|---|---|---|
| Hold threshold | 3 seconds | 2s is within accidental-trigger range for gloved hands outdoors |
| Countdown duration | 5 seconds (tunable to 3s if ECM 4 Technician timing walkthrough requires) | Safety net; must feel like deliberate time, not a flash |
| Haptic — hold phase | `navigator.vibrate(40)` per second | Perceptible through gloves; "still tracking" signal |
| Haptic — countdown phase | `navigator.vibrate(80)` per second | Longer pulse marks "time is passing" |
| Haptic — commit | `navigator.vibrate([100, 50, 100])` | Double pulse; unmistakably "done" |

---

## Screen States

### STATE 0 — Idle Card

**Trigger:** Klaus opens the sign-off card.

**Layout:**
```
┌─────────────────────────────────────────────┐
│  ÖBB 4024.123                               │
│  Wien Westbahnhof · Gleis 3                 │
│  ECM 4 Freigabe · 2026-05-25 · 14:32        │
│                                             │
│  Techniker:        Klaus Gruber             │
│  Fahrzeugzustand:  ✓ Geprüft               │
│  Offene Mängel:    0                        │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │       HALTEN ZUM FREIGEBEN         │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  Diese Freigabe ist rechtsverbindlich       │
│  gemäß EU 2019/779.                         │
└─────────────────────────────────────────────┘
```

**Specs:**
- Vehicle designation: `text-2xl font-bold` — must confirm before thumb moves
- Timestamp: system-generated, refreshed on card open, displayed to the second
- "Offene Mängel: 0" — green. If > 0: amber with count, hold button **disabled**, copy: "Freigabe gesperrt — offene Mängel"
- Hold button: `min-height: 80px`, neutral fill (not green — green belongs to success states only), `border-radius: 12px`, `text-base font-semibold uppercase tracking-wide`
- Legal notice: `text-xs text-muted` — present, not prominent

---

### STATE 1 — Hold Active

**Trigger:** `touchstart` / `pointerdown` on hold button.
**Duration:** 3 seconds.

**Layout:**
```
┌─────────────────────────────────────────────┐
│  ÖBB 4024.123 · Wien Westbahnhof · Gleis 3  │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │  ████████████░░░░░░░░░░░░░░░░░░░░  │   │
│  │         HALTEN … (2s)              │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  Loslassen = Abbruch                        │
└─────────────────────────────────────────────┘
```

**Specs:**
- Progress bar fills left-to-right inside the button, ÖBB red fill
- Driven by `requestAnimationFrame` against `performance.now()` delta — not CSS animation (cannot pause/resume cleanly)
- Label updates each second: "HALTEN … 3s" → "HALTEN … 2s" → "HALTEN … 1s"
- Haptic: `navigator.vibrate(40)` on each second tick
- "Loslassen = Abbruch": `text-sm text-muted` below button
- Early release at any point → instant return to STATE 0, no animation, no warning, no log entry
- Context header remains fully visible

---

### STATE 2 — Countdown

**Trigger:** Hold completes (3s elapsed without release).
**Duration:** 5 seconds.

**Idempotency key generated at this moment:** `crypto.randomUUID()` — stored in component state. Not on press. Not on write. At STATE 1→2 transition.

**Layout:**
```
┌─────────────────────────────────────────────┐
│  ÖBB 4024.123 · Wien Westbahnhof · Gleis 3  │
│                                             │
│  Freigabe wird übermittelt …                │
│                                             │
│         ●●●●●○○○○○                          │
│         5  4  3  2  1                       │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │       FREIGABE ABBRECHEN           │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

**Specs:**
- Hold button removed from DOM
- 5 dots (`24px` diameter, `16px` gap): ÖBB red → grey as each extinguishes, one per second
- Numeric countdown: `text-3xl font-bold text-center` — two visual channels for glove/stress legibility
- Haptic: `navigator.vibrate(80)` per second
- Cancel button: full-width, `min-height: 64px`, bordered neutral (white fill, dark text — NOT red; red = error not cancellation), label `FREIGABE ABBRECHEN font-semibold uppercase`
- **No server request initiated during this state**
- Focus-loss handling: `visibilitychange` + `pagehide`/`pageshow` listeners pause countdown, store elapsed time, resume on focus return. Klaus is not committed while on a radio call or handling an incoming notification
- On cancel: countdown stops, idempotency key discarded, return to STATE 0, toast "Freigabe abgebrochen" (3s auto-dismiss). No log entry

---

### STATE 3 — Writing

**Trigger:** Countdown reaches zero.

**Specs:**
- Single atomic POST to `ecm_signoff_events` endpoint
- Header: `X-Idempotency-Key: {uuid generated at STATE 1→2 transition}`
- Full card dims to `opacity: 0.6` except spinner and copy
- Spinner: `24px` circular indeterminate, ÖBB red, `role="status"` `aria-label="Freigabe wird übermittelt"`
- Cancel button is gone — no undo once write initiates
- Context header remains dimmed but legible
- Minimum display: 800ms (prevent flash). Timeout: 10s → STATE 5
- `409 Conflict` response → treat as success, route to STATE 4

---

### STATE 4 — Confirmed

**Trigger:** Write returns `200 OK` or `409 Conflict`.

**Layout:**
```
┌─────────────────────────────────────────────┐
│  ✓  FREIGABE ERTEILT                        │
│                                             │
│  Fahrzeug:     ÖBB 4024.123                 │
│  Gleis:        Wien Westbahnhof · 3         │
│  Zeitstempel:  2026-05-25 · 14:32:47        │
│  Techniker:    Klaus Gruber                 │
│  Protokoll-ID: ECM-2026-0525-007423         │
│                                             │
│  Befund:       Fahrtauglich ohne Mängel     │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │     DISPATCHER BESTÄTIGT           │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  [ Karte schließen ]                        │
└─────────────────────────────────────────────┘
```

**Specs:**
- Header: `text-3xl font-bold text-green-600` — only green appearance in this flow
- All summary fields: `text-lg` minimum. Labels `font-medium text-muted`, values `font-bold`. Two-column layout, consistent left-alignment
- **Protokoll-ID is mandatory** — server-generated record ID from the append-only log. Klaus reads this over the radio; absence requires a separate lookup flow which is a chain-of-custody gap
- "Befund" field: human-readable verdict from vehicle state snapshot captured at STATE 0
- "Dispatcher bestätigt": lightweight dispatcher-ack write (separate audit row with Protokoll-ID + Klaus session ID). NOT a second commit to `ecm_signoff_events`
- "Karte schließen": `font-medium underline` text link, not a primary button. Closing does not undo anything
- Haptic on entry: `navigator.vibrate([100, 50, 100])`
- If card closed in < 15 seconds: non-blocking toast "Zusammenfassung noch nicht bestätigt" (nudge, not a gate)

---

### STATE 5 — Error

**Trigger:** Non-409 error response, or 10s timeout.

**Layout:**
```
┌─────────────────────────────────────────────┐
│  ✗  ÜBERMITTLUNG FEHLGESCHLAGEN             │
│                                             │
│  Die Freigabe wurde NICHT gespeichert.      │
│                                             │
│  Fehlercode: NET-503                        │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │        ERNEUT VERSUCHEN            │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  [ Dispatcher manuell benachrichtigen ]     │
└─────────────────────────────────────────────┘
```

**Specs:**
- Header: `text-3xl font-bold text-red-600`
- "Die Freigabe wurde NICHT gespeichert." — NICHT capitalised. Must be unmissable in loud, bright, stressful environment
- "Erneut versuchen": retries with **same idempotency key** from STATE 1→2 transition. If first request landed but response was lost, retry returns `409 Conflict` → routed to STATE 4 as success. No double-commit
- "Dispatcher manuell benachrichtigen": text link, opens pre-composed radio script or direct depot message. Fallback for network-down scenario
- Fehlercode: `text-sm text-muted` — for support logs, not Klaus's decision

---

## Component Inventory

| Component | States | Notes |
|---|---|---|
| `SignOffCard` | idle · hold-active · countdown · writing · confirmed · error | Top-level state machine; owns idempotency key |
| `ContextHeader` | default · locked | Visible in all states; dimmed only during writing |
| `HoldButton` | idle · charging (0–100%) | Removed from DOM in countdown+ states |
| `CountdownDisplay` | counting · paused · cancelled | Pause on `visibilitychange` |
| `CancelButton` | active · hidden | Only visible in countdown state |
| `WritingSpinner` | active | `aria-live` region |
| `ConfirmationSummary` | visible · dismissed | Protokoll-ID from server response |
| `ErrorPanel` | visible | Retry uses stored idempotency key |

---

## Decision Log

| Decision | Rationale |
|---|---|
| Option C over A | Hold-only (Option A) is accidental-trigger-prone with gloves outdoors; two phases separate deliberation from commitment |
| Option C over B | Option B's naive implementations both have dangerous failure modes (silent miss under client crash; accidental commit under network jitter). Option C with client-side timer eliminates both |
| 3s hold threshold (not 2s) | 2s is within accidental range for gloved hands in cold/wind conditions |
| Client-side timer, single atomic write | No distributed pending state; no cancel race conditions; server remains a simple append-only log |
| Idempotency key at STATE 1→2 transition | Generated before any network activity; safe across retries and response-loss scenarios |
| Countdown pauses on focus loss | Klaus must not be committed while on a radio call or handling an incoming notification |
| Green only in STATE 4 | Green in the button or progress states would dilute the success signal |
| Protokoll-ID mandatory in STATE 4 | Chain of custody — Klaus reads it aloud; absence requires a separate lookup |

---

## Open Gate (Pre-Go-Live, Not Pre-Dev)

One walkthrough with an actual ECM 4 Technician timing the 8-second total gesture (3s hold + 5s countdown) against a real 14-minute depot window.

If 8s is unacceptable: countdown compresses to **3 seconds**. The two-phase structure (hold-to-initiate + countdown) is non-negotiable — that is not tunable. The countdown duration is.

This validation does not block development. It blocks go-live sign-off.
