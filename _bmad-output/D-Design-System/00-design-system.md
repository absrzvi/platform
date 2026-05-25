# Design System — ÖBB Landside Platform (Minimal Seed)

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Status:** Phase 4 seed — minimal token vocabulary so page specs can reference real scales. Grows organically as actual usage emerges (Phase 7 formalizes; per Saga's `design_system_mode: none` for MVP, this is a working seed not a frozen library)
**Stack:** Tailwind CSS 4 — tokens map to Tailwind utilities by default

> This is a seed. Extract patterns as they appear across multiple page specs. Do not pre-design here — pre-design what's needed only when page specs need to reference it.

---

## Design principles (locked from Trigger Map)

1. **Advisory-only is visible in the grammar.** Every action verb identifies the operator as actor. "Mark executed" not "Execute." "Record sign-off" not "Sign off." "Approve" not "Send."
2. **Glanceable under pressure.** Stefan is on a 12-hour shift. Information density is high but readable in one scan. Cost-impact one-liner is *card-level*, not drill-down.
3. **Three-tier confidence visual grammar.** Every Kondukt-generated artefact carries a confidence indicator. Non-optional.
4. **Decision-traceable.** Every audit surface answers who-decided-when-what-outcome, not just "an event happened."
5. **Plain language for sign-off, technical depth for hands-on.** Same data, two presentations. Governed by role on a single card schema.
6. **One information model, role-filtered.** Depot PWA uses progressive disclosure by role, not tabs.

---

## Spacing Scale

Aligned to Tailwind 4. Token names are platform-internal; Tailwind class is the implementation.

| Token | Tailwind | Px | Use |
|---|---|---|---|
| `space-zero` | `gap-0` / `p-0` | 0 | Flush / overlap — explicit |
| `space-2xs` | `gap-1` / `p-1` | 4 | Within-component tight (icon-to-label) |
| `space-xs` | `gap-2` / `p-2` | 8 | Within-component default |
| `space-sm` | `gap-3` / `p-3` | 12 | Between related objects within a group |
| `space-md` | `gap-4` / `p-4` | 16 | Default gap between elements within sections |
| `space-lg` | `gap-6` / `p-6` | 24 | Between sections within a page |
| `space-xl` | `gap-8` / `p-8` | 32 | Major section boundaries |
| `space-2xl` | `gap-12` / `p-12` | 48 | Top-level page padding (desktop) |
| `space-3xl` | `gap-16` / `p-16` | 64 | Display-tier separation (rare; e.g. landing) |
| `space-flex` | `flex-1` | — | Pushes remainder to fill viewport |

**Defaults by viewport:**

| Property | Mobile (PWA) | Desktop (Fleet/Telematics) |
|---|---|---|
| Page padding (horizontal) | `space-md` (16px) | `space-2xl` (48px) |
| Section gap | `space-lg` (24px) | `space-xl` (32px) |
| Element gap within section | `space-md` (16px) | `space-md` (16px) |
| Component gap within group | `space-sm` (12px) | `space-sm` (12px) |

---

## Type Scale

Aligned to Tailwind 4. Typeface choices are deliberately conservative.

| Token | Tailwind | Px / Line | Use |
|---|---|---|---|
| `text-2xs` | `text-[10px]` | 10 / 14 | Metadata, timestamp footnotes |
| `text-xs` | `text-xs` | 12 / 16 | Helper text, captions, system labels |
| `text-sm` | `text-sm` | 14 / 20 | Dense data — fault codes, port numbers, technician-depth detail |
| `text-md` | `text-base` | 16 / 24 | Body text (default) |
| `text-lg` | `text-lg` | 18 / 28 | Card titles, prominent labels |
| `text-xl` | `text-xl` | 20 / 28 | Section headings (H2) |
| `text-2xl` | `text-2xl` | 24 / 32 | Page titles (H1) on mobile |
| `text-3xl` | `text-3xl` | 30 / 36 | Page titles (H1) on desktop / Stefan crisis chrome |
| `text-4xl` | `text-4xl` | 36 / 40 | Composite scores on pilot evidence dashboard |

**Weights:** `normal` (400), `medium` (500), `semibold` (600), `bold` (700). No light or extralight.

**Typeface:**

| Surface | Typeface | Notes |
|---|---|---|
| All UI | **Inter** | System fallback for now. Inter is the only typeface in v0; replace only if procurement requires |
| Code / structured data (tool result render, JSON view, payload fields) | **JetBrains Mono** | Monospace required for column alignment + parameter readability |

**Semantic vs visual:** H1/H2/H3 markup the semantic level (for accessibility + screen-reader navigation). Visual styling is separate (size + weight + typeface). An H2 may be `text-sm` if visual hierarchy calls for it.

---

## Color Tokens (seed — minimal)

The platform deliberately avoids high-decoration colour. Disponent persona is on a 12-hour shift; severity colour grammar must be unambiguous and not mistaken for branding.

### Neutral

| Token | Tailwind | Hex | Use |
|---|---|---|---|
| `bg-canvas` | `bg-stone-50` | `#FAFAF9` | Page background |
| `bg-surface` | `bg-white` | `#FFFFFF` | Card surface |
| `bg-elevated` | `bg-stone-100` | `#F5F5F4` | Elevated surfaces (modals, drawers) |
| `border-default` | `border-stone-200` | `#E7E5E4` | Card borders, separators |
| `border-strong` | `border-stone-300` | `#D6D3D1` | Active borders, focused inputs |
| `text-primary` | `text-stone-900` | `#1C1917` | Default text |
| `text-secondary` | `text-stone-600` | `#57534E` | Helper text, metadata |
| `text-disabled` | `text-stone-400` | `#A8A29E` | Disabled state |

### Severity (card-level visual grammar)

Coloured **bands** on cards (left edge), not full-card backgrounds. Reserve fills for explicit interactive state (selected, hovered).

| Token | Tailwind | Hex | Use |
|---|---|---|---|
| `severity-critical` | `bg-red-500 border-red-600` | `#EF4444` | CRITICAL faults — door motor, brake systems |
| `severity-warning` | `bg-amber-500 border-amber-600` | `#F59E0B` | WARNING — non-blocking, deferrable |
| `severity-info` | `bg-sky-500 border-sky-600` | `#0EA5E9` | INFO — observation, not action-needed |
| `severity-resolved` | `bg-emerald-500 border-emerald-600` | `#10B981` | RESOLVED — for closed/cleared cards |

### Confidence (Kondukt three-tier visual grammar — non-optional)

Confidence appears on every Kondukt-generated artefact (RAO card, Proactive Recommendation Card, warranty packet PDF, etc.). Visual is a small inline pill on the card; *never* hidden behind a tooltip.

| Token | Tailwind | Visual | Meaning |
|---|---|---|---|
| `confidence-high` | `bg-emerald-100 text-emerald-900 border-emerald-300` | Filled pill | High confidence — supported by multiple data sources, low uncertainty |
| `confidence-medium` | `bg-amber-100 text-amber-900 border-amber-300` | Half-filled pill | Medium confidence — partial data or model uncertainty |
| `confidence-low` | `bg-stone-100 text-stone-700 border-stone-400` | Outline pill | Low confidence — divergent signals or sparse data. RAOs with low confidence are visibly distinct AND vendor-delivery RAOs require explicit override |

### Identity / IdP

| Token | Tailwind | Use |
|---|---|---|
| `idp-bound` | `text-blue-700` | IdP-bound identity surfaces (sign-off rows, ACK rows, audit export) |
| `idp-pending` | `text-stone-500` | Identity awaiting confirmation (pre-IdP-bind state, edge case) |

### Degraded state

| Token | Tailwind | Use |
|---|---|---|
| `degraded-banner` | `bg-stone-200 text-stone-900 border-stone-400` | Persistent banner for Kondukt unavailable / pgvector unavailable |
| `degraded-text` | `text-stone-600` | Inline degraded-state labels ("Keyword search active", "Data source unavailable") |

---

## Card Schema (the platform's defining component)

The card is the foundational object. Variants exist (triage card, RAO card, fault card, drift card, etc.) but they share a schema.

### Universal fields

```
severity_band:    severity-{critical|warning|info|resolved}  — left edge, full card height
header:           { system: str, badge: confidence?, timestamp: utc }
summary:          plain-English one-line — text-md, text-primary
cost_impact:      optional one-liner — text-sm, text-secondary (Stefan-facing surfaces only)
technical_detail: collapsed by default — text-sm in JetBrains Mono when relevant
actions:          operator-verb buttons — never platform-action verbs
metadata_footer:  { logged_by_idp: str, logged_at: utc } when applicable
```

### Operator-verb button rules (LOCKED — derived from Trigger Map cross-group pattern)

| Allowed | Forbidden |
|---|---|
| "Mark executed" | "Execute" |
| "Mark executed (with modification)" | "Modify and execute" |
| "Record sign-off" | "Sign off" |
| "ACK — resolution logged" | "Resolve" |
| "DEFER" | "Defer (sent to ECM)" |
| "Approve evidence packet" | "Send to Stadler" |
| "Download PDF" | "Submit warranty claim" |
| "Generate recommendation" | "Get recommendation from Kondukt" (verb makes Kondukt the actor) |
| "View" / "Open" | "Launch" / "Activate" |
| "Mark manual decision" | "Submit manual override" |

Rule: the operator is *always* the grammatical subject of the verb. Kondukt proposes; the operator decides.

**Roland-test alternative (2026-05-25, party mode — Caravaggio):** "Log What You Did" — plain-language, operator-subject, conversational, scans in <1s. Stronger 3-second-rule performance than "Mark executed" in user testing. Worth a side-by-side Roland test before locking permanently across all RAO surfaces.

---

### LOW-confidence RAO behaviour (LOCKED 2026-05-25)

When `confidence === "low"` on any RAO, the following grammar applies:

| Element | Treatment |
|---|---|
| Confidence pill | Outline-only, dashed 1.5px border, "LOW" small-caps with tracking-wide |
| Card left-rail | Desaturates from severity colour to neutral-400 (chroma drains out) |
| Justification block | `data-vouched="false"` — desaturated, prefixed "Unverified context:", no inline confidence chip |
| "Mark executed" button | **DISABLED** (greyed, non-interactive, `aria-disabled="true"`). Tooltip: "Platform declines to vouch for low-confidence recommendations — use Modify to record your action." |
| "Modify" button | The only interactive execution path. Pre-filled with the AI's draft per Victor's chess move (friction = "tap, adjust, commit," ~2s). **Cursor pre-placed in the justification field** (not at end of draft) — forces a moment of attention before commit. |
| Modify-on-LOW visual differentiation | Modify on LOW must visibly differ from Modify on HIGH (the same affordance + different intent collapses under time pressure into rubber-stamping). Treatment TBD — likely outline weight or accent on the justification field — but the differentiation itself is LOCKED. |
| Audit row | `decision_basis = "operator_modified_low_confidence"` **plus `modification_distance`** (lexical or semantic diff between AI draft and committed text) — without this field, Victor's Phase 2 trust-signal argument collapses into noise |
| Visual restraint (Caravaggio) | Hold-to-record progress ring stays full-saturation on the Modify button — restraint must look intentional, not broken |

**Rationale (party mode 2026-05-25):** three independent paths converged on DISABLE — Saga (Renate's audit committee reads two row types more cleanly than three), John (Modify *is* the job on LOW), Victor (DISABLE produces the Phase 2 evidence dataset). Audit truth is preserved because Modify still records the action; the modification rate becomes the Phase 2 trust signal.

**Pre-mortem guardrails (added 2026-05-25, advanced elicitation pass):**

- **`modification_distance` is a required audit-schema field.** Recording *that* the operator modified is not enough; we need to record *how much*. Without it, a 1.8s tap-scan-commit rubber-stamp and a substantive rewrite both register as `operator_modified_low_confidence` and the Phase 2 trust dataset is junk. Add before Sprint 4 build — this is a one-way door we almost missed.
- **LOW-rate is a first-class operational metric.** The DISABLE behaviour is calibrated to a low-frequency edge case (~8% of RAOs at design time). If LOW-rate exceeds **15% sustained over any 72-hour window**, the calibration assumption is broken — page the design team. At 30%+ LOW, the forced-Modify friction stops being signal and starts being theatre, and operators will route around it.
- **Modify-on-LOW differentiation is non-negotiable.** Same gesture as Modify-on-HIGH = collapse under time pressure into the rubber-stamp pattern Saga's audit-truth argument was supposed to prevent.
- **Cursor pre-placement in the justification field is a forcing function.** Small ergonomic detail, large signal-quality consequence. Spec-locked.

---

## Gesture: Hold-to-Record (Klaus shadow sign-off; Stefan "Mark executed" analog)

The defining trust-building gesture. Identical across MVP shadow and Phase 2 authoritative — only the underlying write target changes.

### Spec

| Phase | Duration | Visual feedback | Cancellation |
|---|---|---|---|
| Idle | — | Button labelled (e.g.) "Record sign-off" | n/a |
| Hold (initiate) | 3 seconds | Button border-strong → severity-resolved fill progresses left-to-right | Release → revert to idle |
| Countdown | 5 seconds | Ring around button, countdown number visible (5→4→3→2→1) | Release OR cancel button → revert to idle, no write |
| Atomic write | At expiry | Button momentarily flashes; transitions to success state | None — write fires exactly once at expiry |
| Success | — | Card transitions to permanent state ("Cleared (shadow matched)" / "Marked executed") | None — record is immutable |

**Idempotency:** key generated at countdown initiation; safe retry semantics on 409.
**Audit row:** captures the *gesture timing* (hold_started_at, countdown_completed_at, atomic_write_at) alongside the action.

---

## ECM Audit Row Schema (LOCKED 2026-05-25, party-mode pass)

The canonical audit row written on every sign-off event. Hash-chained, HMAC-bound, IdP-identity-stamped, immutable. Corrections only via `supersedes_id`.

### `record_type` enum (the Phase 2 transition contract)

| Value | Renders as (EN) | When used |
|---|---|---|
| `"advisory"` | "Audit record — advisory mode (paper sign-off is authoritative)" | MVP — paper is legally authoritative; platform shadows + reconciles |
| `"authoritative"` | "Audit record — authoritative mode" | Phase 2 — platform is legally authoritative, gated by five trust criteria sign-off |

**LOCKED contract:**
- The enum is in HMAC-canonicalized content. The string is **not** — strings live in a renderer lookup (`labelFor(record_type, locale)`).
- Per-row enum is immutable. Phase 2 "flip" is a write-side default change for *new* rows; historical rows render per their own enum forever.
- Noun is invariant across modes — "Audit record" survives the promotion across downstream surfaces (contractor QA manuals, Schienen-Control field reports, ÖBB-Einkauf procurement language). Only the `— mode` sub-line changes.

### Why this matters (party-mode pass, 2026-05-25)

The prior label was "Shadow record (companion to paper)" → "Authoritative record." Four agents converged on the diagnosis that:
- Saga: "shadow" leaks into committee minutes, procurement language, contractor habit during MVP — relabelling becomes a contract-amendment cycle, not a flag flip
- Winston: the "single config flag" framing was an architecturally false slogan — historical rows must stay on their original label forever (hash chain), so "promotion" is really "from date X forward"
- Mary: ECM-4 maintenance contractor chain (8–15 subcontractor QA manuals) was the missed stakeholder — full relabel = 2–4 months of Renate's attention
- Mimir: schema must be (c) enum-in-canonical + string-in-renderer; (b) string-in-canonical is operationally impossible without breaking the chain

Rejected before any external leakage of the prior phrase.

### Phase 2 cutover work (NOT a flag flip — tracked separately)

Even with the invariant-noun label, Phase 2 promotion is comms work:
- Formal letter to Schienen-Control explaining the mode change
- Audit-committee briefing memo (Renate co-signs)
- Update to the ECM-4 contractor handover packet template (the `— mode` line shifts; QA manuals citing "Audit record" do not need reissue)
- A doctrine paragraph: "what 'authoritative from date X' means — historical advisory rows remain advisory by chain integrity"

A Phase 2 cutover runbook stub belongs in the planning artifacts, not here. Flag = code; runbook = comms.

---

## Component References (placeholder — extract on second use per WDS principle)

When the same pattern appears in a second page spec, extract it here.

- [Card — Triage] — TBD (first appears in `01.1-triage-view`, define on second use in scenario 02 or 09)
- [Card — RAO] — TBD (first appears in `01.5-rao-card`, define on second use in scenario 02 or 06)
- [Hold-to-record gesture] — defined above; first usage `03.6-hold-to-record`
- [Confidence pill] — defined above; first usage `01.5-rao-card`
- [Degraded banner] — defined above; first usage `10.1-triage-degraded-banner`

---

## Translation Strategy

`product_languages: [en]` per WDS config. **MVP is English-only.** German is post-MVP for Disponent + ECM Manager comfort — but pilot Disponent + ECM personas speak English in the operations centre during the pilot (confirmed in early scoping).

Translation keys (`page.section.object`) are pre-populated as if multi-language was on, so a German pass is a content-only add later.

---

## Accessibility Floor

| Requirement | Floor |
|---|---|
| Contrast | WCAG 2.2 AA — 4.5:1 for normal text, 3:1 for large text and UI components |
| Keyboard | Every operator-action button reachable + activatable via keyboard. The hold-to-record gesture must have a keyboard-accessible equivalent (e.g. `Hold Enter for 3s + 5s countdown`) |
| Screen reader | Severity bands announce their meaning ("Critical fault, door motor, Car 4"). Confidence pills announce their tier ("High confidence"). Audit export tables use proper `<th scope>` markup |
| Focus | Visible focus state on every interactive object — `outline-2 outline-offset-2 outline-blue-600` |
| Motion | Respect `prefers-reduced-motion` — countdown gesture works without the radial animation |

---

_Generated for Phase 4 spec authoring — grows organically as page specs accrete. Do not freeze; revise as patterns emerge._
