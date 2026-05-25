# ADR-001: Conductor App Transport — PWA via Onboard Access Point

**Date:** 2026-05-23
**Status:** Decided
**Deciders:** Abbas Rizvi

---

## Context

The Conductor App is a greenfield interface for ÖBB train crew (conductors) running on Android tablets. It needs to:
- Display real-time alerts from the onboard event-store via WebSocket
- Surface LLM advisory responses from the Hailo-10H via OpenAI-compatible REST
- Work fully offline — no internet dependency for any feature
- Be distributed to conductors without MDM, app store, or device management complexity

Two options were evaluated: React Native (Expo) and PWA (React + Vite).

---

## Decision

**PWA served from the train's onboard WiFi access point.**

The conductor connects to the train's local WiFi AP. The browser is redirected to the PWA automatically on connection. The app runs in Chrome on the conductor's device (ÖBB-issued or personal Android).

---

## Rationale

### Why PWA wins in this deployment model

The key concern with PWA (WebSocket death under Android Doze mode) does not apply here. The conductor is actively using the app while connected to the AP — it is not a background process. The Doze/tab-suspension risk applies to backgrounded apps; an active browsing session is not subject to the same throttling.

The AP redirect model also eliminates every other argument for React Native:

| Concern | React Native answer | PWA + AP answer |
|---|---|---|
| Distribution | MDM or app store | None — browser redirect on AP join |
| Updates | EAS build + MDM push | Deploy to AP server, instant for all |
| WebSocket reliability | Foreground service | Active session, not backgrounded |
| Device compatibility | ÖBB-issued only | Any Android with Chrome |
| Background alerts | Native push | Not needed — app is foregrounded |

### Team and stack alignment

The existing codebase is React 18 + Vite (`control-centre/`). The Conductor App becomes the second Vite app in the monorepo:
- Shared tsconfig, eslint, Playwright, vitest
- Shared component library from `shared/`
- Same CI pipeline — `vite build` → `playwright test` → artifact upload
- Zero new toolchain, zero new runners

React Native would require: Expo EAS (external cloud CI dependency), Metro bundler, Detox, expo-sqlite — none of which the team has configured, and all of which contradict the offline-resilient, self-contained deployment model.

---

## Implementation Specifics

- **Framework:** React 18 + Vite — new app at `conductor-app/`
- **PWA plugin:** `vite-plugin-pwa` + Workbox (cache-first on static assets)
- **Offline store:** `idb` (IndexedDB wrapper) — typed, Promise-based
- **WebSocket:** native `WebSocket` API, exponential backoff reconnect (same pattern as `control-centre/`)
- **LLM calls:** `fetch()` to Hailo-10H OpenAI-compatible REST endpoint (local network)
- **Manifest:** `display: standalone`, locked portrait orientation, ÖBB theme
- **Hosting:** Nginx on onboard AP server serves PWA static assets; AP DHCP redirect points browser to PWA URL on connection
- **Testing:** Playwright (E2E) + vitest (unit) — identical to `control-centre/`
- **Escape hatch:** if a native feature is ever needed, wrap in Capacitor — one command, no rewrite

---

## UX Non-Negotiables (transport-agnostic, from Freya)

These apply regardless of transport choice and must be acceptance criteria in every Conductor App story:

1. **One-handed operation** — every primary action reachable by thumb; no two-hand gestures
2. **Minimum tap target: 56dp / 14mm** — motion adds error, size up from standard 48dp
3. **Severity 1 alerts = full-screen takeover + vibration** — not a badge, not a banner
4. **"Is it safe to continue?" response ≤ 3 seconds** — conductor is under departure pressure; latency is a UX requirement
5. **No modal flows during alert state** — alert layer is non-destructive; conductor returns to their context
6. **Offline state invisible unless actionable** — everything works offline by design; do not show "you're offline"
7. **Glanceable coach diagram** — readable in under 2 seconds without interaction; colour + count, no prose
8. **Departure clearance = deliberate hold or double-tap + haptic** — must feel distinct from every other tap

---

## Consequences

- `conductor-app/` added as third app in monorepo alongside `control-centre/` and future `fleet-manager/`
- Onboard AP server must serve static files (Nginx recommended; FastAPI static mount acceptable for PoC)
- AP captive portal or DHCP redirect must be configured to point to PWA URL — infrastructure story required
- No Play Store, no MDM, no Expo account needed

---

## Rejected Alternative

**React Native (Expo)** — rejected because:
- WebSocket reliability concern (Doze mode) does not apply in the AP redirect / active session model
- Introduces EAS cloud dependency contradicting offline-first positioning
- Requires new toolchain the team has no experience with
- Adds 4–8 weeks ramp-up with no feature benefit for this use case
