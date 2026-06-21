---
name: emd
description: Use when planning, drafting, or structuring an engineering milestone document or roadmap for a multi-milestone initiative — the engineering end state and how the journey phases into independently shippable milestones. Triggers when the user asks for an EMD, milestone plan, or phased engineering roadmap, has a PRD or large initiative to break into milestones, or needs to sequence a multi-quarter build with decision checkpoints before any single RFC is written.
---

# EMD — Engineering Milestone Document

## Overview

An EMD is the engineering plan for a **multi-milestone initiative**. It is the peer of
the PRD: the PRD makes the product case, the EMD describes the **engineering journey** —
the end state and how it phases into milestones. It is **not** a milestone-level doc: it
does not enumerate requirements, designs, or acceptance criteria. Each milestone gets its
own RFC proposing *how* to build it (use the `rfc` skill); a milestone big enough to pin
down *what* before *how* may also get an ERD first (use the `erd` skill). The EMD owns one
job: justify the **shape of the plan** — why the milestone boundaries fall where they do.

| Doc | Altitude | Answers |
|---|---|---|
| PRD | Initiative (product) | Who needs what, and why |
| **EMD** | **Initiative (engineering)** | **What the end state is, and how we phase the journey into milestones** |
| RFC | One milestone | The proposed approach for a milestone, circulated for comment |
| ERD *(optional)* | One milestone | Problem-first requirements, when a milestone is big enough to pin down "what" before "how" |

## When to use

- The user asks for an EMD, milestone plan, or phased engineering roadmap.
- A multi-milestone initiative needs its end state and sequencing framed before any single
  RFC — structural work spanning teams, systems, or quarters that can't be one change.

**Skip it** when the work is a single shippable change — that's one RFC, not a journey.

**Not** for: one milestone's approach (use `rfc`), one milestone's requirements (use
`erd`), or product/user framing (use `prd`).

## Workflow

1. **Gather grounding.** Collect the PRD, the initiative's goals, known constraints, team
   and dependency facts. Ask the user for anything a section needs but you don't have —
   don't fill gaps with guesses.
2. **Copy `TEMPLATE.md`** as the starting structure. Keep section order.
3. **Fill it section by section**, applying the authoring rules below. Repeat the milestone
   block per milestone, in sequence.
4. **Match `EXAMPLE.md`** for tone, grounding, and how precision decays with milestone distance.
5. **Strip all `> Guidance:` lines** and replace every bracket before sharing.
6. **Write the Summary last**, once the end state and milestone cut are settled.

## Authoring rules

1. **Ground every statement in the provided input.** Never invent dates, sizing,
   dependencies, team names, or metrics. Unknown fact → `[TBD: …]`; load-bearing inference
   → `[ASSUMPTION: …]`. A plan full of honest markers beats one full of confident guesses.
2. **Stay at milestone altitude.** Describe each milestone's *objective* and *exit
   criteria*, not its requirements or design. Catch yourself writing "P0: the system
   must…", acceptance criteria, or a schema → you've dropped an altitude; link to the RFC.
3. **Every milestone must deliver standalone value.** Independently shippable and worth
   shipping on its own. If a milestone only makes sense once the next lands, the boundary
   is wrong — merge or re-cut. This is the central discipline of the doc.
4. **Every milestone needs exit criteria and a decision checkpoint.** Exit criteria = the
   observable conditions that mean it's done. The checkpoint = what you re-evaluate before
   committing to the next. Sequencing is a series of revisable bets, not a fixed contract.
5. **Engage with approach, not implementation.** Name the key technical bets and justify
   *why the boundaries fall where they do*. Don't specify data models, libraries, or code
   structure — that's the RFC's job. You're justifying the plan's shape, not designing.
6. **Make sequencing and dependencies explicit.** State the critical path, what runs in
   parallel, and what each milestone assumes is already true. Hidden ordering is the main
   reason phased plans slip.
7. **Let precision decay honestly with distance.** Near milestones are concrete; far ones
   are coarse. Don't manufacture false precision for milestone 4 — mark it rough and flag
   what would sharpen it.
8. **Surface plan-invalidating risks, with pivot/kill criteria.** Name the bets that, if
   wrong, force a re-cut. Risks and open questions are mandatory — not empty unless settled.
9. **The Summary must stand alone.** Assume a reader who reads only the Summary: convey the
   end state, the shape of the journey, and the stakes.
10. **Be concise.** Length scales with scope and milestone proximity. Prefer concrete lists
    over prose. Aim for a doc a busy lead reads in fifteen minutes.

## Conventions (use literally)

- `[TBD: what's missing — and who should answer it]` — a required fact, date, or sizing you weren't given.
- `[ASSUMPTION: the inference — and what would confirm it]` — a load-bearing inference, visible so a human can correct it.
- Keep both load-bearing. A doc full of brackets wasn't really planned — but honest markers on a far milestone are a feature, not a gap.

## Sections (in `TEMPLATE.md`)

Metadata → Summary → Context and motivation → Goals and non-goals → Success criteria →
Current state → target state → Guiding principles and constraints → Milestones (objective
/ exit criteria / scope / key technical bets / dependencies / sizing / risks / RFC /
decision checkpoint, per milestone) → Sequencing and dependencies → Cross-cutting concerns
→ Risks and open questions → Out of scope and future → Appendix.

## Common mistakes

- A milestone that only pays off once the next one lands → re-cut the boundary so each ships standalone value.
- Dropping into requirements, acceptance criteria, or schemas → that's the milestone's RFC/ERD; link out.
- Manufacturing precise sizing for a distant milestone → mark it `[TBD]` and say what would sharpen it.
- Hidden ordering between milestones → state the critical path and what each assumes is already true.
- No decision checkpoint → every milestone re-evaluates the next bet before committing.
- Empty risks/open questions on an unsettled plan → name the bets that would force a re-cut, with pivot/kill criteria.

Before finishing, re-read the milestone cut and ask: *could we ship after any single
milestone and still be better off?* If a milestone fails that test, the boundary is wrong.
