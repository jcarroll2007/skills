# [Title — short, specific name of the thing being built]

> Copy this file, replace every bracketed placeholder, and delete all `> Guidance:`
> lines before sharing. Keep the section order. Sections marked optional can be deleted
> if they don't apply; don't fill them with air.

## Metadata

| | |
|---|---|
| **Author(s)** | [name(s) / LLM + reviewing human] |
| **Reviewers** | [names — and mark who the *decider/approver* is] |
| **Status** | `Draft` → `In review` → `Approved` / `Superseded` |
| **Created / Last updated** | [YYYY-MM-DD] |
| **Tracking issue / Epic** | [link, or `[TBD]`] |
| **Target ship** | [rough date or milestone; optional] |

## 1. TL;DR / Summary

> Guidance: 3–5 sentences a busy reader can act on in 30 seconds. Problem → proposed
> approach → the single most important tradeoff or risk → the specific decision or
> approval you're asking for. Write this last. State the ask explicitly.

[…]

## 2. Context / background & problem

> Guidance: Enough grounding for a reader who isn't deep in this area. What does the
> world look like today, what's the actual pain (quantify it), why now, and what
> constraints are fixed (deadlines, existing systems, compliance). Link data/dashboards.

[…]

## 3. Goals & non-goals

> Guidance: Goals = outcomes that define success (outcome-shaped, not solution-shaped).
> Non-goals = what you're deliberately not doing — mark each "not now" (and where it
> lives instead) or "not ever". Non-goals are your strongest defense against scope creep.

**Goals**
- […]

**Non-goals**
- [explicit thing we are not doing — and where it lives instead, or "not ever"]

## 4. Requirements

> Guidance: The contract the design must satisfy. Scale the depth to the risk.

**Functional** — what the system must *do*, behaviorally. Phrase as MUST / MUST NOT / SHOULD,
and name the important edge cases.

- [MUST / MUST NOT / SHOULD …]

**Non-functional** — the qualities it must have. Fill only the rows that matter for this change;
put a number on every one you can.

| Dimension | Requirement |
|---|---|
| Performance / latency | [target p50/p95/p99, with numbers] |
| Scale | [current → expected peak, + headroom] |
| Reliability | [availability; delivery guarantee; RTO/RPO if relevant] |
| Security / privacy | [sensitive data classes; see §7a] |
| Cost | [rough $ delta; sign-off needed?] |

## 5. Proposed design

> Guidance: The core of the doc. Reviewers must follow your reasoning, not just see a
> conclusion. Go high-level first, then drill in.

### 5a. High-level overview

> Guidance: Major components and how they fit (one diagram beats three paragraphs — see §11).
> Trace the happy path in a few sentences. Call out the 1–2 genuinely non-obvious, load-bearing
> decisions a reviewer should focus on.

[Diagram + narrative of the happy path. Flag the load-bearing decisions.]

### 5b. Data model `[delete if no persisted data changes]`

> Guidance: Entities/tables that change, key fields and types, the index serving each access
> pattern, source of truth, retention, consistency requirements.

[…]

### 5c. APIs / contracts `[delete if no interface changes]`

> Guidance: New/changed endpoints, events, or signatures — request/response shape, error cases
> and status codes, idempotency, pagination, versioning & backward compatibility (who are the
> existing callers and how do you not break them).

[…]

### 5d. Key flows & components

> Guidance: The risky/non-obvious flows only (retry-after-partial-failure, concurrent update,
> cold start) — each as a short sequence. Where state lives, consistency boundaries, and the
> sync vs async boundaries and why.

[…]

## 6. Alternatives considered

> Guidance: This is what separates a design doc from a spec. For each alternative: what it was
> and what it optimized for, the axis on which it's worse, why it was rejected (tied to a
> specific requirement above), and the future condition that would make you revisit. Include
> "do nothing" if it's at all plausible.

### Alternative A: [name]
- **What:** [approach]
- **Tradeoffs:** [what it's better/worse at]
- **Why rejected:** [tied to a requirement/constraint]
- **Revisit if:** [the condition that would flip this decision]

### Alternative B: [name]
- […]

## 7. Cross-cutting concerns

> Guidance: The things that don't fit one component but will hurt you if ignored. Keep tight by
> default; scale up the ones that carry real risk for this change.

### 7a. Security & privacy

> Guidance: Sensitive data (PII, secrets) at rest / in transit / in logs; authN/authZ — who can
> call this and how it's enforced; any new attack surface or threat, named with its mitigation.

[…]

### 7b. Observability

> Guidance: The metrics, logs, traces, and alert thresholds that tell you it's healthy, and the
> dashboard on-call opens. Incident-question test: what's the *first question* during an incident,
> and can you answer it from what you're adding?

[…]

### 7c. Failure modes

> Guidance: What can fail (dependency down, queue backs up, bad input, partial write), how each is
> detected, the blast radius, and the mitigation/recovery. Note what degrades gracefully vs. fails
> hard, and whether failure is isolated or cascades.

| Failure | Detection | Blast radius | Mitigation / recovery |
|---|---|---|---|
| [e.g. provider down] | [success% drops, DLQ grows] | [that channel only; others unaffected] | [backoff retries; alert at DLQ > 1k; manual replay] |

### 7d. Migration / backfill `[delete unless you change existing data or cut over]`

> Guidance: Stepwise plan where each step is independently reversible, with a verification gate
> between steps (row counts, sampled diff) and the rollback at each stage.

[…]

### 7e. Rollout & rollback

> Guidance: How this reaches production safely (flag, %-ramp, canary, shadow). The rollback
> mechanism and how fast it is, whether it's safe at *every* stage (forward-compatible data),
> and the kill switch and exactly what it does.

[…]

## 8. Testing strategy

> Guidance: What's tested at each level (unit / integration / load / manual) and *why there* —
> targeting where the risk is, not a coverage number. Which risky behaviors get explicit
> integration or load coverage, and what you're deliberately not testing and why that's acceptable.

[…]

## 9. Open questions & risks

> Guidance: Mandatory — an empty section here is usually a lie. Open questions each with an owner,
> a needed-by date, and what they block. Risks as likelihood × impact, plus the mitigation or the
> trigger that tells you to act.

**Open questions**
- [question — owner — needed-by — what it blocks]

**Risks**
- [risk — likelihood/impact — mitigation or trigger]

## 10. Rollout plan & success metrics

> Guidance: Defines done and how you'll know it worked. Phases/milestones with the gate between
> them. Each success metric with its current baseline, target, and when you'll measure. What would
> tell you to roll back or rethink rather than push forward. (Ramp mechanics live in §7e.)

**Plan / milestones**
- [phase — entry/exit gate]

**Success metrics**

| Metric | Baseline | Target | Measured when |
|---|---|---|---|
| [metric] | [now] | [target] | [date / event] |

## 11. Appendix `[optional]`

> Guidance: Supporting material that would clutter the main flow — diagrams, detailed schemas,
> benchmark data, prior-art links, related RFCs, glossary, prototype links, design-review notes.

- [diagrams]
- [links: tracking issue, related RFCs, dashboards, prototypes]
- [glossary, if the domain has jargon a reviewer won't know]
