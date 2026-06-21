---
name: adr
description: Use when recording a technical or architectural decision and its reasoning as a durable, standalone record — one accepted/superseded choice, not a deliberation. Triggers when the user asks for an ADR, an architecture/decision record, or a decision-log entry, wants to capture "why we chose X" so a future engineer understands it, or needs to distill a settled RFC or design discussion into a permanent record.
---

# ADR — Architecture Decision Record

## Overview

An ADR is a short, immutable record of **one** technical decision: the context that
forced it, the choice made, and the consequences accepted. It captures *why* durably,
so an engineer months later understands the decision without re-deriving it. It is the
record, not the deliberation — an RFC argues *which* option and circulates for comment;
the ADR pins down *what was decided and why* in half a page and stays put.

ADRs are **one immutable file per decision**, not a single living doc. They accumulate as a
numbered **decision log** — `docs/adr/0001-*.md`, `0002-*.md`, … — where each record stops
changing once accepted and a later decision *supersedes* an earlier one rather than editing it.

| Doc | Job | Lifecycle |
|---|---|---|
| RFC | Deliberate the approach and alternatives before committing | Evolves in review, then frozen |
| **ADR** | **Record one decision and its consequences, durably** | **Immutable once accepted; superseded, never edited** |

## When to use

- The user asks for an ADR, decision record, or decision-log entry.
- A decision worth remembering has been made — a choice between real options with lasting
  consequences (data store, framework, API contract, a boundary, a convention) — and the
  *why* needs to live where the next engineer will find it.
- A settled RFC or design discussion needs its outcome distilled into a permanent record.

**Skip it** when the choice is trivial, obvious, or trivially reversible with no lasting
consequence — a code comment or PR description is cheaper.

**Not** for: deliberating a contested or hard-to-reverse design (use `rfc`); requirements
(use `erd`); product framing (use `prd`).

## Workflow

1. **Gather grounding.** Collect the decision, the constraints behind it, and any RFC/ticket.
   Ask the user for anything a section needs but you don't have — don't fill gaps with guesses.
2. **Create a new file** in the decision log (`docs/adr/NNNN-short-title.md`) — never append to an
   existing ADR. Take the next unused number (`ADR-NNNN`) and **copy `TEMPLATE.md`**. Keep section order.
3. **Fill it section by section**, applying the authoring rules below.
4. **Match `EXAMPLE.md`** for brevity, the honest negatives, and the imperative title.
5. **Strip all `> Guidance:` lines** and replace every bracket before sharing.

## Authoring rules

1. **One decision per record.** Recording two choices → write two ADRs. A record that decides
   several things can't be cleanly superseded later.
2. **Immutable once accepted.** Don't rewrite an accepted ADR to reflect a new choice — write a
   new one and mark this `Superseded by ADR-NNNN`. The record *is* the history; editing it
   destroys the value.
3. **Capture the forces, not just the verdict.** Context states the constraints and tradeoffs in
   tension *at the time*. A reader must see why a reasonable person chose this, not just that you did.
4. **State consequences honestly — including the bad ones.** Every decision costs something: what
   you're now locked into, what got harder, what you'll revisit. An ADR with only upsides is marketing.
5. **Be brief.** Half a page to a page. It's a record, not a design doc — if it sprawls into
   alternatives and diagrams, the deliberation belongs in an RFC; link it and record only the outcome.
6. **Write at decision time, in the present.** "We will use X." Capture it while the context is
   fresh, not reconstructed from memory months later.
7. **Title the decision, not the topic.** "Use Postgres for the ledger", not "Database choice" — a
   reader scanning the log should read the decision itself.

## Conventions (use literally)

- Number sequentially and immutably: `ADR-0007`. Never reuse or renumber.
- Status: `Proposed` → `Accepted` → `Superseded by ADR-NNNN` / `Deprecated`.
- `[NEEDS INPUT: …]` — a required fact or decision you weren't given; never a guess.

## Sections (in `TEMPLATE.md`)

Metadata (number / status / deciders / date) → Title → Context and forces → Decision →
Consequences (positive / negative / neutral) → Alternatives considered *(brief)* → Links.

## Common mistakes

- Editing an accepted ADR to change the decision → write a new one; mark this superseded.
- Several decisions in one record → split them.
- Consequences list only the wins → name what got worse and what you're locked into.
- A vague topic title ("Caching") → state the actual decision.
- Growing into a full design doc → that's an RFC; record only the outcome here.
- Backfilling from memory long after → the forces get sanitized; write it at decision time.

Before finishing, re-read Context and Consequences and ask: *could a new engineer, months from
now, explain why we chose this and what it cost — without asking anyone?* If not, that's the fix.
