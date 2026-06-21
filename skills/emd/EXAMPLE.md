# Worked example

A complete EMD that follows every authoring rule. Match this level of grounding,
concision, and milestone discipline. Note how each milestone delivers standalone value,
how precision decays honestly from Milestone 1 to Milestone 4, the explicit decision
checkpoints, and the non-empty risks with pivot/kill criteria — these are features, not gaps.

---

# Payments reliability — from best-effort processing to an auditable, exactly-once pipeline

## Metadata

| | |
|---|---|
| **Author** | Claude (drafted), reviewed by A. Okafor |
| **Status** | In review |
| **Last updated** | 2026-06-21 |
| **Reviewers** | Payments team, SRE on-call lead, Finance |
| **Related docs** | Payments reliability PRD (linked in Appendix); Milestone 2 RFC — *Dedup at the ingestion boundary*; Milestone 2 ERD (optional) — *Idempotent payment-webhook ingestion* |

## Summary

Our payments pipeline processes each provider event on a best-effort basis, with no durable
record of what happened and no automated way to detect drift between our state and the
provider's. The result is duplicate refunds, double-counted revenue, and manual
reconciliation by Finance. This initiative moves us, over four milestones, to a pipeline
where every state-changing event is recorded once in an append-only ledger and discrepancies
are detected and corrected automatically. We sequence the work so each milestone removes a
class of error on its own: first we can *see* the problem, then we stop duplicates, then we
make the ledger the source of truth, then we self-heal. Revenue is rising ahead of a planned
promotion, so the cost of each unreliable event is growing.

## Context and motivation

Today the pipeline has no memory: a handler runs, mutates state, and forgets. When the
provider retries a delivery — which it does on any non-2xx or slow response — we re-run the
side effect. We cannot answer "did we already process this event?" without manual log
spelunking, and we cannot prove to Finance that our numbers match the provider's. Finance
logged 14 duplicate-refund incidents in the last 90 days. `[ASSUMPTION: that figure is from
support tickets only; the true count is likely higher. Finance to confirm.]` Each incident
is direct monetary loss plus reconciliation effort plus eroded trust. A single RFC can't fix
this because the problem is structural and spans ingestion, storage, and reconciliation — it
needs to be sequenced, not solved in one change.

## Goals and non-goals

**Goals**
- Every unique state-changing event takes effect exactly once and is durably recorded.
- Finance can reconcile our ledger against the provider without manual log work.
- Drift between our state and the provider's is detected automatically, and routine cases are corrected without human intervention.

**Non-goals**
- Redesigning how we integrate with the payment provider.
- Supporting multiple payment providers — this covers the one we have.
- Replacing Finance's accounting system; we feed it, we don't become it.

## Success criteria

- Duplicate-refund incidents drop to zero across a full promotion cycle.
- 100% of state-changing events are present in the ledger (measured by sampled audit against provider records).
- Manual reconciliation time drops by `[TBD: Finance to set a target against current baseline]`.

## Current state → target state

**Current:** Stateless handlers mutate balances directly on each delivery. No event store, no
dedup, no reconciliation. Observability is limited to application logs.

**Target:** Inbound events are deduplicated at ingestion, then appended to a durable, ordered
event log that is the source of truth for downstream state. A reconciliation service
periodically compares the ledger to the provider and either auto-corrects known drift classes
or escalates. `[ASSUMPTION: an append-only log is the right backbone; this is revisited at the
Milestone 2 checkpoint before we commit to building it.]`

## Guiding principles and constraints

- **No downtime and no data loss during migration.** Each milestone ships behind a flag and is reversible.
- **Backward compatible until the ledger is authoritative.** Existing consumers keep working until Milestone 3 cuts over.
- **Measure before changing.** We don't ship a fix to a number we can't currently see.

## Milestones

### Milestone 1 — Observe and baseline

- **Objective:** Instrument the existing pipeline so we can measure duplicate and lost events. Delivers a dashboard and a real baseline number on its own — valuable even if the initiative stopped here, because it turns an anecdotal problem into a measured one.
- **Exit criteria:** Dashboard shows duplicate-event rate and a daily reconciliation gap vs. the provider; baseline numbers agreed with Finance.
- **Scope:** In — instrumentation, dashboards, baseline report. Out — any change to processing behavior.
- **Key technical bets:** Pure observability; no behavior change. We deliberately do this first so every later milestone has a before/after measurement, and so we size Milestones 2–4 against real data rather than guesses.
- **Dependencies:** None. This is the entry point.
- **Sizing:** Small (~1–2 weeks).
- **Risks:** Low. Main risk is that instrumentation reveals the problem is larger than scoped, which would re-shape later milestones — an acceptable thing to learn early.
- **RFC:** None — small enough to proceed from this doc.
- **Decision checkpoint:** Does the measured duplicate rate justify the rest of the initiative? If it's negligible, stop here.

### Milestone 2 — Idempotent ingestion

- **Objective:** Deduplicate inbound events so each unique event takes effect at most once, regardless of how many times the provider delivers it. Removes the duplicate-refund class of error on its own.
- **Exit criteria:** Replayed deliveries acknowledge successfully without re-running side effects; duplicate-event rate on the Milestone 1 dashboard drops to ~0.
- **Scope:** In — dedup at the ingestion boundary. Out — durable storage of event history (that's Milestone 3).
- **Key technical bets:** Solve dedup at ingestion, before any state mutation, so later milestones build on a clean stream. We stop here rather than bundling storage because idempotency is independently valuable and independently testable.
- **Dependencies:** Milestone 1 (need the dashboard to prove the rate dropped).
- **Sizing:** Medium (~3–4 weeks).
- **Risks:** Dedup-record store availability becomes a new dependency in the hot path; addressed in the RFC.
- **RFC:** *Dedup at the ingestion boundary* (linked in Related docs). An ERD also exists for this milestone — *Idempotent payment-webhook ingestion* — but that's optional; most milestones carry only an RFC.
- **Decision checkpoint:** Did duplicates actually go to zero? Confirm the dedup approach holds under promotion-level load before building durable storage on top of it.

### Milestone 3 — Durable event ledger

- **Objective:** Persist every state-changing event in an append-only log and make it the source of truth for downstream balances. Gives Finance a reconcilable record.
- **Exit criteria:** All state changes are derivable from the ledger; downstream state is rebuilt from the ledger in a staging cutover and matches production.
- **Scope:** In — the event log, the cutover of downstream consumers to read from it. Out — automated correction of drift (Milestone 4).
- **Key technical bets:** Append-only log as backbone, cut over behind a flag with a fallback to the current path. The boundary sits here because storage is only worth building once the inbound stream is clean (Milestone 2).
- **Dependencies:** Milestone 2 (a clean, deduplicated stream to record).
- **Sizing:** Large (~6–8 weeks). `[ASSUMPTION: sizing assumes one provider's event volume; revisit after Milestone 1 baseline.]`
- **Risks:** Cutover is the highest-risk step in the initiative; mitigated by flag + reversible rollout and the no-data-loss principle.
- **RFC:** `[TBD: to be written before Milestone 3 starts]`.
- **Decision checkpoint:** Does the ledger reconcile cleanly against the provider in staging before we make it authoritative in production?

### Milestone 4 — Reconciliation and self-healing

- **Objective:** Automatically detect drift between the ledger and the provider and correct routine cases, escalating the rest. Removes ongoing manual reconciliation.
- **Exit criteria:** `[TBD: define which drift classes auto-correct vs. escalate, with Finance]`.
- **Scope:** Coarse — this milestone is intentionally under-specified until Milestones 1–3 reveal the real drift patterns.
- **Key technical bets:** `[TBD: depends on what drift classes the ledger surfaces in practice.]`
- **Dependencies:** Milestone 3 (needs an authoritative ledger to reconcile against).
- **Sizing:** `[TBD: cannot size responsibly until we see real drift data from Milestone 3.]`
- **Risks:** Auto-correction acting on bad data could compound errors rather than fix them; the milestone will need strong guardrails defined in its RFC.
- **RFC:** `[TBD]`.
- **Decision checkpoint:** Is the residual manual reconciliation load after Milestone 3 large enough to justify automation, or is escalation-only sufficient?

## Sequencing and dependencies

Strictly sequential on the critical path: 1 → 2 → 3 → 4, because each milestone consumes the
previous one's output (measurement, then a clean stream, then a durable record, then
reconciliation against it). The only parallelizable work is dashboard refinement from
Milestone 1, which can continue alongside Milestone 2.

## Cross-cutting concerns

- **Migration:** Every behavior change ships behind a feature flag with a defined rollback. No big-bang cutovers.
- **Observability:** The Milestone 1 dashboard is extended at each milestone to prove the targeted error class actually dropped.
- **Security/compliance:** The ledger stores financial event data; retention and access controls are `[TBD: confirm with security and Finance]`.
- **Backward compatibility:** Existing consumers read the old path until the Milestone 3 cutover; the old path stays available as a fallback through that milestone.

## Risks and open questions

- **Risks:** The largest bet is that an append-only ledger is the right backbone (Milestones 3–4 depend on it). If Milestone 2 reveals dedup is far harder than expected under load, the storage design may need to absorb dedup responsibility, re-cutting the boundary between 2 and 3.
- **Pivot/kill criteria:** If Milestone 1 shows a negligible duplicate/drift rate, stop after Milestone 1. If the Milestone 3 staging cutover can't reconcile cleanly, hold before production cutover and re-scope.
- **Open questions:**
  - What is the true baseline incident count? — Finance — `[TBD]`.
  - What ledger retention does compliance require? — Security + Finance — `[TBD]`.
  - Which drift classes are safe to auto-correct? — Payments + Finance — `[TBD, resolved at Milestone 3 checkpoint]`.

## Out of scope and future

- Multi-provider support and a generalized event backbone beyond payments — deferred to a future initiative.
- Real-time reconciliation (this initiative targets periodic).

## Appendix

- Payments reliability PRD: [link `[TBD]`]
- Provider webhook delivery & retry specification: [link `[TBD]`]
- Glossary: *append-only log* — a store where events are only ever added, never modified, so the full history is the source of truth. *Drift* — divergence between our recorded state and the provider's.
