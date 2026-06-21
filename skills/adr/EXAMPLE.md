# Worked example

A complete ADR that follows every authoring rule. Note the imperative title that states the
decision, the forces captured *as they were*, the honest negative consequences (lock-in, a
possible future migration), and the brief alternatives that link back to the RFC instead of
re-arguing it. This ADR records the outcome of the RFC in the `rfc` skill's worked example.

---

# ADR-0007: Use SQS for notification fan-out

## Metadata

| | |
|---|---|
| **ADR** | ADR-0007 |
| **Status** | Accepted |
| **Deciders** | J. Park (Notifications TL), SRE on-call lead |
| **Date** | 2026-06-21 |
| **Supersedes / Superseded by** | — |

## Context & forces

We are moving notification fan-out off the synchronous checkout path onto a queue (RFC PROJ-901).
That requires picking a queue technology, and two real options were on the table: SQS and Kafka.
The forces in tension:

- We need at-least-once delivery at ~2k events/sec peak. We do **not** need ordering or replayable streams.
- The team has run SQS in production for two years; we have no Kafka operational experience.
- The analytics team has floated adopting Kafka but has not committed.
- The reliability OKR deadline is this quarter, so time-to-ship is a hard constraint.

## Decision

We will use **SQS** for notification fan-out: one queue with a dead-letter queue per channel,
deduping at enqueue via `event_id`.

## Consequences

**Positive**
- Ships within the quarter on infra we already operate — no new ops capability to build.
- Per-channel DLQs give isolated failure handling and a manual replay path out of the box.

**Negative / costs**
- No ordered or replayable event stream. If we later need an event log (e.g. for analytics),
  SQS won't serve it and we'll face a migration.
- If analytics adopts Kafka anyway, we'll be running two messaging systems — duplicated ops surface.

**Neutral**
- Delivery becomes eventual (seconds), which the product has accepted (RFC §4).

## Alternatives considered

- **Kafka** — ordered, replayable, would also serve analytics; rejected because we have no Kafka
  ops experience and those features aren't needed here, failing the time-to-ship constraint.
- **Do nothing (synchronous fan-out)** — rejected: it's the problem the RFC exists to fix.

## Links

- Realizes RFC PROJ-901 (async notification fan-out).
- Revisit if analytics commits to Kafka — reopen as a new ADR superseding this one.
