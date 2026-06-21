---
name: rfc
description: Use when designing, drafting, or structuring an engineering design doc or RFC — the chosen technical approach, why it beats the alternatives, and what could go wrong. Triggers when the user asks for a design doc, RFC, or technical proposal, has a problem or ERD to turn into a concrete design, or faces a one-way-door or cross-team decision that needs review before building.
---

# RFC — Engineering Design Doc

## Overview

An RFC (engineering design doc) proposes **how** to build something and argues **why this
approach over the alternatives**. It is a thinking tool and a review artifact, not a status
report: a reviewer should finish it able to explain, in their own words, why you chose this
design and what would break it. It sits downstream of intent — a PRD frames the product bet and
an ERD pins the testable requirements (use those skills for the *what* and *why*); the RFC commits
to a design and owns the tradeoff. Optimize for decisions, not ceremony.

## When to use

Write an RFC when any of these hold:

- **One-way door** — expensive or painful to reverse (data model, public API, migration, vendor commitment).
- **Blast radius** — touches multiple teams or services, shared data, or a contract others depend on.
- **Cost of being wrong is high** — rework, a backfill, or risk to data/users.
- **There's disagreement** — two reasonable engineers would design it differently.
- **You can't hold it in your head** — explaining the plan verbally takes more than ~15 minutes.

**Skip it** when the change is small, reversible, single-owner, and a short PR description is the
cheapest honest spec. Don't write a doc to look diligent.

**Not** for: product/user framing (use `prd`), or testable requirements without a design (use `erd`).

## When to write it

At **~70% formed** — you have a real proposal and at least one named alternative, but haven't sunk
days into an implementation you'll feel defensive about. The doc should still be able to change
your mind. Before the build, not during it.

## Workflow

1. **Gather grounding.** Collect the PRD/ERD, tickets, constraints, and data. Ask the user for
   anything a section needs but you don't have — don't fill gaps with guesses.
2. **Copy `TEMPLATE.md`** as the starting structure. Keep section order.
3. **Fill it section by section**, applying the authoring rules below.
4. **Match `EXAMPLE.md`** for tone, grounding, and the weak→strong level of detail.
5. **Delete every `> Guidance:` line and `[bracket]`** before sharing, and drop optional sections
   that don't apply.
6. **Write the TL;DR last**, once the design and the ask are settled.

## Authoring rules

1. **Reason, don't assert.** A reviewer must follow *why*, not just see a conclusion. Go high-level
   first, then drill in, and flag the one or two load-bearing decisions explicitly.
2. **Alternatives are mandatory and real.** A doc with no genuine alternative is a decision already
   made. For each: what it optimizes for, why rejected (tied to a requirement), and the condition
   that would make you revisit. Include "do nothing" when it's at all plausible.
3. **Put a number on it.** "Fast" → "p95 < 30s." "Scales" → "2k events/sec, 3× current." Unknown
   number → `[NEEDS INPUT: …]`, never a guess.
4. **Design the unhappy path.** Failure modes, rollback, and observability are not afterthoughts.
   Incident-question test: when this pages on-call, what's the first thing they'll ask, and can
   they answer it from what you're adding?
5. **State the ask.** The TL;DR must name the specific decision or approval you want.
6. **Scale depth to risk.** Lean default; fill non-functional and cross-cutting rows only where they
   carry real risk. High-traffic or shared-infra work earns the full treatment.
7. **Don't pad.** Length follows scope — a small design is one page. Delete optional sections rather
   than filling them with air.
8. **Resolve to a decision, not consensus.** Record "disagree and commit" outcomes and who decided.

## Conventions (use literally)

- `[NEEDS INPUT: …]` — a required fact, number, or decision you weren't given.
- `[ASSUMPTION: …]` — a reasonable inference made to keep moving; visible so a human can correct it.
- Keep both rare and load-bearing. A doc full of brackets wasn't really written.

## Sections (in `TEMPLATE.md`)

Metadata → TL;DR → Context & problem → Goals & non-goals → Requirements (functional /
non-functional) → Proposed design (overview / data model / APIs / key flows) → Alternatives
considered → Cross-cutting concerns (security & privacy / observability / failure modes /
migration / rollout & rollback) → Testing strategy → Open questions & risks → Rollout plan &
success metrics → Appendix.

## How to run the review

Default to async: share the doc, name reviewers and a **decider** up front (so it can't stall in
limbo), and give 2–3 working days for inline comments. Hold a synchronous review only if comments
reveal real disagreement, cross-team coordination is needed, or the stakes are high — require a
pre-read, and use the meeting for *contested points*, not a live walkthrough. Mark each thread
resolved with the decision and who made it.

## Common mistakes

- Conclusion with no reasoning → show the path and flag the load-bearing decisions.
- No real alternatives (or "too complex" with no "for what") → tie each rejection to a requirement.
- Vague non-functional claims ("fast", "reliable") → numbers or `[NEEDS INPUT]`.
- Only the happy path → failure modes, rollback, and the incident question.
- Empty open questions on an unsettled design → surface the real unknowns, each with an owner.
- Writing it during or after the build → it can no longer change your mind. Write at ~70% formed.

Before requesting review: can a reviewer who isn't you read §1–§6 and explain, in their own words,
why you chose this design over the alternatives and what would break it? If not, that gap is the
thing to fix.
