# [Title — short, specific name of the initiative]

> Copy this file, replace every bracketed placeholder, and delete all `> Guidance:`
> lines before sharing. Keep the section order. Repeat the milestone block per milestone.
> Near milestones are concrete; far ones are coarse — use `[TBD]` over fake precision.

## Metadata

| | |
|---|---|
| **Author** | [name / LLM + reviewing human] |
| **Status** | `Draft` → `In review` → `Approved` / `Superseded` |
| **Last updated** | [YYYY-MM-DD] |
| **Reviewers** | [names, or `[TBD]`] |
| **Related docs** | [PRD link; child RFCs/ERDs as they're created; or `[TBD]`] |

## Summary

> Guidance: 3–5 sentences. The end state, the shape of the journey (how many milestones,
> roughly), and why it matters now. Must stand alone — assume a reader who reads only this.

[…]

## Context and motivation

> Guidance: The bigger-picture problem or opportunity this initiative addresses — a higher
> altitude than an RFC's problem statement. Why a multi-milestone investment is justified,
> and why now. Stay factual and grounded.

[…]

## Goals and non-goals

> Guidance: What the whole initiative achieves, and what it explicitly will not. These are
> initiative-level, not per-milestone.

**Goals**
- […]

**Non-goals**
- […]

## Success criteria

> Guidance: How we'll know the *initiative* succeeded — outcome-level, not per-feature
> acceptance criteria. Numbers where possible, `[TBD]` where not.

[…]

## Current state → target state

> Guidance: Where we are today and where we're going — the north star. Describe the
> high-level shape of the end state. Do not design it.

**Current:** [...]

**Target:** [...]

## Guiding principles and constraints

> Guidance: Technical tenets and hard constraints that span all milestones (e.g.
> zero-downtime migrations, backward compatible until milestone N, regulatory limits).
> These constrain every milestone below.

- […]

## Milestones

> Guidance: The heart of the doc. One block per milestone, in sequence. Near milestones are
> concrete; far ones are coarse and may carry `[TBD]`s. Repeat the block as needed.

### Milestone [N] — [name]

- **Objective:** [The capability or outcome this delivers, and the standalone value it has on its own.]
- **Exit criteria:** [Observable conditions that mean it's done.]
- **Scope:** In — […]. Out — […].
- **Key technical bets:** [The approach-level decisions, and why the boundary falls here.]
- **Dependencies:** [On prior milestones, teams, external factors. What it assumes is already true.]
- **Sizing:** [Rough order of magnitude. Coarser for distant milestones; use `[TBD]` over fake precision.]
- **Risks:** [Milestone-specific risks.]
- **RFC:** [Link to the milestone's RFC, or "None — small enough to proceed from this doc." Link an ERD here too if the milestone warranted separate requirements.]
- **Decision checkpoint:** [What we re-evaluate before committing to the next milestone.]

## Sequencing and dependencies

> Guidance: The overall order, the critical path, and what can run in parallel. A simple
> ordered list or dependency sketch is fine.

[…]

## Cross-cutting concerns

> Guidance: Things that span milestones and are easy to forget per-milestone: data
> migration, rollback / feature-flagging, observability, security and compliance, backward
> compatibility.

- […]

## Risks and open questions

> Guidance: Initiative-level risks — the big bets that could change the whole plan — and
> explicit pivot/kill criteria. Mandatory; do not leave empty unless genuinely settled.

- **Risks:** […]
- **Pivot/kill criteria:** […]
- **Open questions:** [question — owner — `[TBD]`]

## Out of scope and future

> Guidance: What's explicitly deferred beyond this initiative, so reviewers don't assume
> it's included.

- […]

## Appendix `[optional]`

> Guidance: Links, glossary, anything a reader might need but that would clutter the main flow.

- Glossary: […]
