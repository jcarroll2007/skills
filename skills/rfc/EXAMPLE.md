# Worked example

A complete RFC that follows every authoring rule. Match this level of grounding,
concision, and numbered specificity. Note the real "do nothing" alternative, the
incident-question test in observability, the failure-modes table, and the non-empty
open questions with owners — these are features, not gaps.

---

# Async notification fan-out

## Metadata

| | |
|---|---|
| **Author(s)** | Claude (drafted), reviewed by D. Reyes |
| **Reviewers** | Notifications team, SRE on-call lead — **decider: J. Park (Notifications TL)** |
| **Status** | In review |
| **Created / Last updated** | 2026-06-21 |
| **Tracking issue / Epic** | PROJ-901 |
| **Target ship** | Q3, ahead of the reliability OKR deadline |

## 1. TL;DR / Summary

Users miss time-sensitive alerts because we fan out notifications synchronously inside the
checkout request path, so a slow SMS provider raises checkout p99 and drops sends. We propose
moving fan-out to an async queue with per-channel workers. The tradeoff is that delivery becomes
eventual (seconds) instead of best-effort-synchronous. **Asking for approval on the queue choice —
SQS vs. Kafka, see §6.**

## 2. Context / background & problem

Today, the checkout handler calls each notification provider inline before returning. When our SMS
provider's p99 spikes, checkout p99 follows, and a provider timeout drops the send entirely.

Support tickets tagged "missed notification" tripled QoQ (40 → 130/mo). This blocks the Q3
reliability OKR (checkout p99 < 800ms), which we currently breach whenever a provider degrades.
Constraint: the checkout service is owned by another team and we cannot expand its request budget;
existing email templates and provider SDKs stay as-is.

## 3. Goals & non-goals

**Goals**
- Decouple delivery from the request path so provider latency can't affect checkout p99.
- Allow adding a new channel (e.g., push) without changing caller code.
- Operators can tell, for any event, whether and to which channels it was delivered.

**Non-goals**
- A user-facing notification-preferences UI — tracked in PROJ-812 (not now).
- Cross-channel ordered delivery in v1 (not now).
- Migrating existing email templates — they stay as-is (not ever, under this doc).

## 4. Requirements

**Functional**

- Given a triggered event, the system MUST deliver to every channel the user has enabled.
- It MUST retry transient provider failures up to N times with exponential backoff.
- It MUST NOT deliver to a channel the user disabled *after* the event fired.
- It MUST acknowledge duplicate enqueues idempotently (dedupe at enqueue, not delivery).

**Non-functional**

| Dimension | Requirement |
|---|---|
| Performance / latency | Deliver within 30s p95 |
| Scale | Sustain 2k events/sec at peak (3× current), with 2× headroom |
| Reliability | At-least-once delivery; duplicates acceptable and deduped client-side |
| Security / privacy | Payloads contain email + order ID (PII); no PII in logs — see §7a |
| Cost | Added infra < $400/mo; no sign-off needed below $500/mo |

## 5. Proposed design

### 5a. High-level overview

Checkout publishes an event to a queue and returns immediately. Per-channel workers consume,
dedupe at enqueue, call the provider with backoff, and write a delivery record. A poison message
lands in a dead-letter queue (DLQ) for replay.

```
checkout ──enqueue──▶ [queue] ──▶ email worker ──▶ provider
                          │
                          ├──────▶ sms worker   ──▶ provider
                          └──────▶ push worker  ──▶ provider
                                      └─(failures)─▶ [DLQ] ──▶ replay tool
```

**Load-bearing decisions:** (1) dedup happens at *enqueue*, not delivery — deliberate, so a worker
crash mid-delivery can't double-send (see §6). (2) one worker pool *per channel*, so a slow SMS
provider can't starve email.

### 5b. Data model

Add a `delivery` table — source of truth for what was sent.

| Column | Type | Notes |
|---|---|---|
| `event_id` | uuid | dedup key; unique |
| `channel` | enum | email / sms / push |
| `status` | enum | queued / sent / failed |
| `updated_at` | timestamptz | |

- `idx_event_channel (event_id, channel)` serves the per-event status lookup.
- `idx_dlq_created` serves the replay tool's "failed since T" scan.
- Retention: 30 days, then archived.

### 5c. APIs / contracts

`POST /v2/notifications` — body `{event_id, user_id, channels[]}`, returns `202` + `{notification_id}`.
The `Idempotency-Key` header dedupes retries within 24h. `4xx` on an unknown channel. The existing
synchronous `/v1` path stays until checkout migrates (tracked in PROJ-840).

### 5d. Key flows & components

- **Retry after partial failure:** if 2 of 3 channels succeed, only the failed channel's message is
  retried; succeeded channels are not re-sent (per-channel delivery records gate this).
- **Concurrent delivery of one event:** the unique `event_id` dedup key means two simultaneous
  consumers produce exactly one delivery per channel; the loser's insert conflicts and is dropped.

## 6. Alternatives considered

### Alternative A: Kafka
- **What:** publish events to a Kafka topic; workers consume partitions.
- **Tradeoffs:** ordered, replayable streams that would also serve the analytics team; higher ops burden.
- **Why rejected:** we have no Kafka ops experience, and the ordering/replay features aren't needed
  here — at-least-once is sufficient (§4). The ops cost fails the time-to-ship constraint.
- **Revisit if:** analytics adopts Kafka, since shared infra changes the cost calculus.

### Alternative B: Do nothing (keep synchronous fan-out)
- **What:** leave fan-out in the checkout path; add provider timeouts.
- **Tradeoffs:** zero new infra; but checkout p99 stays coupled to provider p99.
- **Why rejected:** directly violates the Q3 reliability OKR (§2) and doesn't stop dropped sends.
- **Revisit if:** never — the coupling is the problem.

## 7. Cross-cutting concerns

### 7a. Security & privacy

Payloads contain user email + order ID (PII): encrypted at rest, redacted in logs, purged after 30d.
Worker→provider traffic over TLS; provider secrets pulled from vault, not env. Threat — queue
poisoning by a malformed payload; mitigated by schema validation at enqueue, with rejects to the DLQ.

### 7b. Observability

Metric `delivery_latency_seconds` (histogram, by channel). Alert: p95 > 60s for 5m → page. Dashboard
tracks enqueue rate, per-channel success %, and DLQ depth. The first incident question — "are we
dropping or just slow?" — is answerable from DLQ depth + success %.

### 7c. Failure modes

| Failure | Detection | Blast radius | Mitigation / recovery |
|---|---|---|---|
| Provider down | success% drops, DLQ grows | that channel only; others unaffected | backoff retries; alert at DLQ > 1k; manual replay tool |
| Queue backs up | enqueue rate ≫ consume rate | delivery delayed, not lost | autoscale workers; alert on consumer lag > 60s |
| Bad payload | schema validation fails at enqueue | single event | reject to DLQ; never reaches a worker |

### 7d. Migration / backfill

Not applicable — no existing data is rewritten. The cutover is handled as a rollout (§7e), not a backfill.

### 7e. Rollout & rollback

Flag-gated; ramp 1% → 10% → 50% → 100% over 4 days, watching success % + p95. Rollback = flip the
flag (no deploy); data is forward-compatible, so rollback is safe at any stage. Kill switch disables
the async path and falls back to synchronous send.

## 8. Testing strategy

- **Unit:** retry/backoff logic and the dedup key conflict path.
- **Integration:** enqueue → worker → provider stub, including the provider-timeout path.
- **Load:** 2k/sec sustained for 10m; assert p95 < 30s and an empty DLQ.
- **Not testing:** provider SDKs — we trust their tests.

## 9. Open questions & risks

**Open questions**
- Can we reuse the existing dedup table or do we need a new one? — @sam — needed by design review — blocks §5b.
- SQS vs. Kafka final call — @jpark — needed by design review — blocks the whole queue layer.

**Risks**
- Provider rate-limits us at peak (likelihood: med, impact: high). Mitigation: token-bucket per
  provider + DLQ. Trigger to act: 429 rate > 1% in canary.

## 10. Rollout plan & success metrics

**Plan / milestones**
- Phase 1: workers behind flag at 1% — gate: success % ≥ synchronous baseline.
- Phase 2: ramp to 100% — gate: p95 < 30s held for 48h.

**Success metrics**

| Metric | Baseline | Target | Measured when |
|---|---|---|---|
| "Missed notification" tickets | 130/mo | < 30/mo | 6 weeks after full rollout |
| Checkout p99 vs. provider p99 correlation | coupled | decoupled | next incident or scheduled game day |

## 11. Appendix

- Tracking issue: PROJ-901. Related: PROJ-812 (preferences UI), PROJ-840 (/v1 deprecation).
- Glossary: *DLQ* — dead-letter queue, where messages that fail processing are parked for inspection
  and replay. *Fan-out* — delivering one triggered event to multiple channels.
