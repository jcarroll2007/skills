# Worked example

A complete ERD that follows every authoring rule. Match this level of grounding,
concision, and testability. Note the honest `[TBD: …]` markers and the non-empty
Open Questions — these are features, not gaps.

---

# Idempotent payment-webhook ingestion

**Status:** In review
**Author:** Claude (drafted), reviewed by A. Okafor
**Last updated:** 2026-06-21
**Reviewers:** Payments team, SRE on-call lead
**Related docs:** Payment provider webhook spec (linked in Appendix)

## Summary

Our payment provider occasionally delivers the same webhook event more than once. Because we process each delivery independently, duplicate events can cause duplicate refunds and double-counted revenue. This document defines the requirements for ingesting webhooks such that each unique event takes effect exactly once, regardless of how many times it is delivered. Solving this removes a known source of customer-facing financial errors and manual finance corrections.

## Problem

- **What's happening:** The provider retries webhook delivery on any non-2xx or slow response. Our handler treats each delivery as a new event, so a retried `refund.succeeded` can trigger a second refund.
- **Who's affected:** Customers (duplicate refunds/charges), the finance team (manual reconciliation), and support (handling complaints). In the last 90 days, finance logged 14 duplicate-refund incidents. [TBD: confirm count with finance — figure is from support tickets only.]
- **Cost of inaction:** Direct monetary loss per duplicate refund, plus ongoing manual reconciliation effort and erosion of customer trust.
- **Why now:** Refund volume is rising ahead of a planned promotion, increasing both retry frequency and the blast radius of each duplicate.

## Goals and non-goals

**Goals**
- Each unique provider event takes effect at most once, no matter how many times it is delivered.
- Duplicate deliveries are acknowledged successfully (so the provider stops retrying) without re-running side effects.
- Operators can tell, for any event, whether it was already processed.

**Non-goals**
- Redesigning how we integrate with the payment provider.
- Handling events the provider never sends (out of scope by definition).
- Deduplicating events *across* different providers — this doc covers one provider.
- Fixing the provider's retry behavior; we treat retries as a given.

## Requirements

**Functional**

| ID | Priority | Requirement | Acceptance criteria |
|----|----------|-------------|---------------------|
| F1 | P0 | Detect whether an incoming event has already been processed, using the provider's event ID | Replaying a previously processed event produces no new side effects, verified by an integration test that sends the same event twice |
| F2 | P0 | Return a 2xx to the provider for a duplicate delivery without re-running side effects | Second delivery returns 2xx; downstream refund count increments exactly once |
| F3 | P1 | Expose processed/not-processed status for a given event ID to operators | An operator can query an event ID and see its processing status and timestamp |
| F4 | P2 | Emit a metric each time a duplicate is detected | Duplicate-detection count is visible in the metrics dashboard |

**Non-functional**

| ID | Requirement | Target |
|----|-------------|--------|
| N1 | Ingestion latency must stay within the provider's delivery timeout | p99 response time < 2s (provider retries after 5s) |
| N2 | Deduplication must hold under concurrent delivery of the same event | Two simultaneous deliveries of one event result in exactly one set of side effects |
| N3 | Deduplication record retention | Records retained at least as long as the provider's retry window: [TBD: confirm provider's max retry horizon] |

**Constraints**
- We cannot change the provider's payload format or retry policy.
- The provider's event ID is the only identifier guaranteed unique per event.

## Success metrics

- Duplicate-refund incidents reported by finance drop to 0 within one month of launch (baseline: ~14 / 90 days).
- Manual reconciliation time spent on duplicates drops to ~0 (baseline: [TBD: get hours/week from finance]).
- Zero increase in webhook delivery failures attributable to this change.

## Assumptions and dependencies

- **Assumptions:** The provider's event ID is stable across retries of the same event. [If false, F1's dedup key is invalid and the approach needs rework.]
- **Dependencies:** Finance to confirm baseline incident and effort numbers; the provider's documentation for the exact retry window (N3).

## Risks and open questions

- **Risks:** If the dedup record store is unavailable, we must decide between rejecting events (provider retries, no data loss, but possible backlog) and processing them (availability, but dedup guarantee weakens). This trade-off needs an explicit decision.
- **Open questions:**
  - How long must dedup records be retained? — Payments team — depends on provider retry horizon `[TBD]`.
  - What is the correct behavior when the dedup store is down? — SRE + Payments — `[TBD]`.
  - Are there event types where "exactly once" is the wrong guarantee (e.g. intentional repeated events)? — Payments — `[TBD]`.

## Appendix

- Provider webhook delivery & retry specification: [link `[TBD]`]
- Glossary: *idempotent* — an operation that has the same effect whether applied once or many times.
