---
name: prd
description: Use when defining, drafting, or structuring product requirements for a feature or product — the problem, the users, the solution, success metrics, and scope. Triggers when the user asks for a PRD, has a rough idea or notes to turn into one, or needs to frame a product bet before engineering work begins.
---

# PRD — Product Requirements Document

## Overview

A PRD turns a rough idea into a clear bet a busy human can read in one sitting and
act on. It is **problem-first**: it does not describe a feature until the problem is
undeniable. It defines the product intent — for whom, why now, what success looks
like — not the engineering requirements or design. (For testable engineering
requirements, use the `erd` skill; for architecture, a separate design doc.)

## When to use

- The user asks for a PRD, or has an idea, request, or notes to turn into one.
- A product bet needs framing — problem, users, scope, metrics — before building.

**Not** for: engineering requirements (use `erd`), or design/architecture (the *how*).

## Workflow

1. **Gather grounding.** Collect the request, notes, research, or tickets. Ask the user
   for anything a section needs but you don't have — don't fill gaps with guesses.
2. **Copy `TEMPLATE.md`** as the starting structure. Keep section order.
3. **Fill it section by section**, applying the authoring rules below.
4. **Match `EXAMPLE.md`** for tone, grounding, and level of detail.
5. **Strip all `> Guidance:` lines** and replace every bracket before sharing.
6. **Write the TL;DR last**, once the rest is settled.

## Authoring rules

1. **Problem before solution.** Don't describe a feature until the problem is undeniable.
   If you can't state the problem crisply, the PRD isn't ready — fix that, not the feature list.
2. **Be concrete.** Replace adjectives with facts. "Slow" → "takes ~40s." "Many users" →
   `[NEEDS INPUT: how many?]`. Never guess a metric or invent data.
3. **Make requirements testable.** One idea per requirement, phrased so QA could mark it pass/fail.
4. **Prioritize ruthlessly.** Must / Should / Could. If everything is a Must, nothing is.
5. **Justify the timing.** Say why now and the cost of inaction. A PRD that doesn't is a
   backlog item, not a plan.
6. **Don't pad.** No "in today's fast-paced world," no restating headings, no summarizing
   what you just said. Every sentence earns its place.
7. **Length follows scope.** A small feature is one page. Don't inflate to look thorough.
8. **Prefer tables and short lists** when they cut reading time; prose when nuance matters.

## Conventions (use literally)

- `[NEEDS INPUT: …]` — a required fact, number, or decision you weren't given.
- `[ASSUMPTION: …]` — a reasonable inference made to keep moving; visible so a human can correct it.
- Keep both rare and load-bearing. A doc full of brackets is a doc that wasn't really written.

## Sections (in `TEMPLATE.md`)

Metadata → TL;DR → Problem → Goals and non-goals → Success metrics → Users and key
use cases → Requirements (Must / Should / Could) → Experience and key flows → Open
questions, assumptions, risks and dependencies → Rollout and milestones.

## Common mistakes

- Leading with the solution → state the problem and its evidence first.
- Vague claims ("slow", "many users") → concrete figures or `[NEEDS INPUT]`.
- Everything a Must → re-prioritize into Should/Could.
- No "why now" or cost of inaction → the bet isn't justified.
- Inventing metrics or evidence to look complete → `[NEEDS INPUT]` is more honest.

Before finishing, re-read the Problem and ask: *would a skeptical engineer agree this
is worth building?* If not, fix the problem statement.
