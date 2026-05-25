# 09: Stefan's Shift Handover

**Project:** ÖBB Landside Platform
**Created:** 2026-05-25
**Method:** Whiteport Design Studio (WDS)
**Sprint:** 5
**Persona:** Stefan the Disponent (Primary)

---

## Transaction (Q1)

**What this scenario covers:**
End of Stefan's 12-hour shift. He generates a shift handover summary, walks his incoming-shift peer (Petra) through open RAOs and in-flight decisions, and the platform records the handover as a discrete event. Petra inherits Stefan's open work without losing context.

---

## Business Goal (Q2)

**Goal:** Shift continuity — supporting RAO acceptance threshold by avoiding context-loss-driven re-decisions
**Objective:** Trigger Map Objective 3 (supporting)

---

## User & Situation (Q3)

**Persona:** Stefan the Disponent (Primary)
**Situation:** Stefan's 12-hour shift ends at 18:00. Petra, the incoming Disponent, arrives 10 minutes before for handover. Wien operations centre. Stefan has 4 open RAOs, 2 with outcomes pending, 1 modification under review.

---

## Driving Forces (Q4)

**Hope:** Hand over open work cleanly so Petra picks up at full speed, and the platform records the handover so any escalation traces back correctly.

**Worry:** Petra acts on an RAO that Stefan was holding for a reason — and the platform's record can't distinguish "Petra accepted" from "Stefan handed over with a held-pending note Petra didn't see."

---

## Device & Starting Point (Q5 + Q6)

**Device:** Desktop (Stefan's workstation; Petra is over his shoulder)
**Entry:** Stefan clicks "Generate handover" from the Fleet Manager AI Interface chrome.

---

## Best Outcome (Q7)

**User Success:**
Stefan generates a handover document in under 30 seconds. He walks Petra through: 4 open RAOs (3 with status "pending downstream confirmation," 1 with "held pending — Stefan note attached"), 2 long-running incidents (one mid-investigation, one cleared this shift), the modification under review (Stefan modified an RAO, awaiting downstream confirmation). Petra inherits cleanly.

**Business Success:**
Handover is recorded as a discrete event. Subsequent platform actions by Petra are bound to her IdP. Stefan's "held pending" note on the RAO survives the handover and is visible to Petra at the moment she encounters that RAO.

---

## Shortest Path (Q8)

1. **Generate handover — entry** — Stefan clicks "Generate handover" in the Fleet Manager AI Interface chrome.
2. **Handover summary view** — Document-style page generates and renders inline. Sections: "Open RAOs (4)", "Held items (1)", "Long-running incidents (2)", "This shift's resolutions (this is the trend view 30/70 inverted — a list of what was closed)", "Outstanding outcome verifications (2 pending downstream confirmation)".
3. **Open RAO row — held item with note** — Stefan walks Petra through the 4 open RAOs. The held item is visually distinct (badge: HELD-PENDING). Stefan's attached note: "Holding pending Roland's verbal on whether maintenance window can shift to Saturday early — see scenario 02 modification." Petra reads.
4. **In-flight modification surface** — Stefan walks Petra through the modification under review. Both original RAO and Stefan's modification visible. Reason for modification quoted ("Friday rotation has higher passenger load — Saturday early less disruptive").
5. **Long-running incidents** — Two cards. Stefan summarises in spoken language (the platform does not autodescribe; it surfaces the facts and Stefan provides voice). Each card has a "Notes for handover" field where Stefan can add free-text context.
6. **Mark handover complete** — Stefan clicks "Mark handover complete". A prompt: "Confirming that Petra Müller is taking over shift coverage from now? This locks the handover summary as authoritative." Stefan confirms. The platform records: Stefan's IdP (outgoing), Petra's IdP (incoming — pulled from her active session via SSO), handover_at timestamp. Subsequent actions on any of the 4 open RAOs will be bound to Petra's IdP.
7. **Triage view — Petra's session** — Stefan walks away. Petra is now at the workstation. Her session shows the triage view inherited from Stefan with "Shift inherited from Stefan Weber at 18:02" badge in the chrome. The 4 RAOs are visible, including the HELD-PENDING one with Stefan's note. ✓

---

## Trigger Map Connections

**Persona:** Stefan the Disponent (Primary)

**Driving Forces Addressed:**
- ✅ **Want W4:** Be the operator whose decisions are captured — handover-complete records Stefan's "I'm leaving with X in this state" assertion
- ❌ **Fear F2:** Reconstructability gap — the held-pending note survives the handover; Petra inherits the *full* state, not a summary
- ❌ **Stefan's persona detail open question** — "shift handover summary" is addressed in this scenario; "handover-complete" definition lives here

**Business Goal:** Trigger Map Objective 3 (RAO acceptance — supports continuity across shifts so RAOs don't get re-litigated and modification rate stays clean)

---

## Scenario Steps

| Step | Folder | Purpose | Exit Action |
|------|--------|---------|-------------|
| 09.1 | `09.1-generate-handover-entry/` | Single-click handover generation | Document renders |
| 09.2 | `09.2-handover-summary-view/` | Document-style layout, sections per category | Stefan walks Petra through |
| 09.3 | `09.3-held-item-with-note/` | Visually distinct HELD-PENDING badge + Stefan's note attached | Stefan opens modification surface |
| 09.4 | `09.4-modification-under-review/` | Original + Stefan's modification + reason | Stefan covers long-running |
| 09.5 | `09.5-long-running-cards-with-notes/` | Free-text notes field per card | Stefan marks complete |
| 09.6 | `09.6-mark-handover-complete/` | IdP-bound discrete event; locks summary | Petra's session inherits |
| 09.7 | `09.7-petra-inherited-triage/` | Triage chrome shows inherited-from badge; held-pending visible | ✓ |

---

## Open Questions (for Phase 4)

- **Petra's IdP — pulled from active session or selected at handover?** — Active session is fastest; selected adds an explicit "I, Stefan, am handing to Petra" gesture. Tradeoff between speed and explicitness. Phase 4 spec.
- **Held-pending note visibility for Petra** — does the note appear inline on the RAO card when Petra encounters it in triage, or only in the handover summary? Both? Phase 4 spec.
- **Cross-session handover (Stefan handing off mid-shift to himself the next shift)** — same gesture? Or specifically two-person? Phase 4 spec.
- **Long-running incident voice narrative** — does the platform have a place to record Stefan's spoken summary as text (he types it after Petra leaves)? Or is the in-platform note enough? Phase 4 spec.
