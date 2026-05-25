# Agent Experience — Party Mode resolved RAO low-confidence grammar

**Date:** 2026-05-25
**Topic:** Sprint 1 Roland questions resolved without Roland
**Phase:** 4 — UX Design

---

## The pattern worth remembering

When a question is tagged "needs operator sign-off," check whether the question is *actually* operator-specific or whether it's a design question disguised as one.

Of 5 Sprint 1 questions tagged for Roland, only **1 fragment** genuinely needed him — item 6 of the modification-reason taxonomy (commercial priority hierarchy is ÖBB-specific). The other 4.something were design questions our agents could answer with their own expertise.

## What worked

1. **Different lenses on the same question expose different failure modes.**
   - Winston (architect) saw the *audit-record-truth* failure mode (DISABLE editorialises on reality).
   - Sally (UX) saw the *operator-rubber-stamp* failure mode (GATE produces autopilot signature).
   - Both are real. Neither is the whole picture.

2. **A fresh voice in round 2 broke the impasse.** John's Jobs-to-be-Done question ("at 70% acceptance rate, can we tell the difference between Stefan-agreed and Stefan-tired?") reframed the debate from UX vs architecture into *what signal does the audit row carry?* — which both Sally and Winston could then answer differently.

3. **Saga authored the persona file Renate is in. Her vote is decisive on Renate-facing decisions** because she has the deepest reading of the constraint. Use Trigger-Map-authors as the lens for downstream decisions about *their* personas.

4. **The "third path" (WITNESS) was elegant but lost on the audit-row-count argument.** Dr. Quinn's TRIZ-style contradiction-resolution proposed relabelling the button. Caravaggio loved the visual; Mimir revised to support it on build economics. But Saga showed it introduced a third row type that Renate's audit committee would question. Lesson: in a regulated-rail context, **ledger semantics beat UI elegance**.

## What I'd reuse next time

- **Round 1: 4 voices, each with a distinct lens** (architect, UX, analyst, builder). Get the initial split.
- **Round 2 if the split is genuine: bring fresh voices** (PM, problem-solver, innovation-strategist) PLUS one voice from round 1 (the dissenting one — give them a chance to defend or move).
- **Round 3 (final round) tests the alternative the room landed on against the *gatekeeper* persona lens** (in this case Saga reading Renate). The "would this artefact survive the audit committee?" question is the trump card in regulated-domain decisions.

## What I'd avoid

- **Don't bring in too many voices early.** Round 1 was 4; round 2 was 4 more; round 3 was 3. Each round had a clear job. Bringing 10 voices into round 1 = chaos.
- **Don't let the orchestrator (me) synthesize agent positions.** The party rules say each agent gets their full unabridged section. I tested this — when I summarized in the orchestrator note, the user got value from the *positions still being distinct*, not from me reconciling them.
- **Don't force a consensus.** The room genuinely split 3 ways (DISABLE / GATE / WITNESS). The lock came from one *strong* argument (Saga's audit-row-count), not from anyone changing their mind under pressure. That's how it should be.

## Translation principle

**Visual restraint must look intentional, or it just looks like a bug ticket.** (Caravaggio.) Applies far beyond LOW-confidence RAOs.

**The trust artefact is the audit row, not the screen.** (Victor + Saga.) When designing regulated-domain UI, optimise for what the ledger reads like in month 9, not what the screen looks like at month 0.

**Modify is the job on LOW, not Mark Executed.** (John.) When confidence is low, the AI is saying "I'm not sure" — the honest operator response is "let me tell you what I actually did," which IS modification. Don't make the operator choose between two paths that mean different things.

---

_This experience worth referencing the next time a regulated-domain question feels stuck or splits the room evenly._
