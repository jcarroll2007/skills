---
name: erd
description: Use when defining, drafting, or structuring engineering requirements for a feature or system — what a solution must do and why, not how to build it. Triggers when the user asks for an ERD, has a PRD or rough idea to turn into engineering requirements, or needs to specify functional/non-functional requirements, constraints, acceptance criteria, and success metrics before design begins.
---

# ERD — Engineering Requirements Document

## Overview

An ERD defines **what** we are building and **why** — the requirements a solution
must satisfy. It does **not** define **how**: architecture, schemas, libraries, and
implementation steps belong in a separate design doc. An ERD is the bridge from a PRD
or idea to design — it turns intent into testable engineering requirements. (For
product/user framing, use the `prd` skill instead.)

## When to use

- The user asks for an ERD, or has a PRD/idea to turn into engineering requirements.
- Requirements, constraints, or acceptance criteria need to be pinned down before design.

**Not** for: design/architecture docs (the *how*), or product requirements (use `prd`).

## Workflow

1. **Gather grounding.** Collect the PRD, ticket, or notes. Ask the user for anything a
   section needs but you don't have — don't fill gaps with guesses.
2. **Copy `TEMPLATE.md`** as the starting structure. Keep section order.
3. **Fill it section by section**, applying the authoring rules below to each.
4. **Match `EXAMPLE.md`** for tone, grounding, and level of detail.
5. **Strip all `> Guidance:` lines** and replace every bracket before sharing.
6. **Leave honest `[TBD: what's missing — who answers it]` markers** wherever a fact is
   unknown. Open Questions must not be empty unless the problem is genuinely settled.

## Authoring rules

1. **Ground every statement in the provided input.** Never invent stakeholders, metrics,
   deadlines, budgets, or constraints. Unknown fact → `[TBD: …]`, not a guess.
2. **Separate problem from solution.** Describe the problem and the requirements, not the
   implementation. Catch yourself writing "we will use…" → that's a design decision; cut it.
3. **Make every requirement testable.** Rewrite vague terms ("fast", "scalable", "secure")
   into numbers or explicit acceptance criteria. Can't measure it → `[TBD: define criteria]`.
4. **Prioritize ruthlessly.** Every functional requirement is P0 (project fails without it),
   P1 (important, can follow up), or P2 (nice to have). If everything is P0, it's wrong.
5. **Be concise.** Every sentence earns its place. Prefer concrete lists over prose. Aim for
   a doc a busy engineer reads in under ten minutes.
6. **Surface disagreement and uncertainty** — don't paper over it. Name conflicting
   requirements instead of silently resolving them.
7. **The Summary must stand alone** — problem, goal, and stakes, readable on its own.
8. **State things plainly.** Mark uncertainty where it exists; don't hedge every sentence.

## Sections (in `TEMPLATE.md`)

Metadata → Summary → Problem → Goals and non-goals → Requirements (functional /
non-functional / constraints) → Success metrics → Assumptions and dependencies →
Risks and open questions → Appendix.

## Common mistakes

- Slipping the *how* in (architectures, tech choices) → belongs in a design doc.
- Unverifiable requirements ("user-friendly") → give a measurable target or `[TBD]`.
- Everything marked P0 → re-prioritize.
- Empty Open Questions on an unsettled problem → surface the real unknowns.
- Inventing metrics or stakeholders to look complete → `[TBD]` is more useful than a guess.
