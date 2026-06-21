# Worked example

A complete PRD that follows every authoring rule. Match this level of grounding,
concision, and prioritization. Note how the brackets are rare and load-bearing, and
the problem comes before any mention of the solution.

---

# Scheduled Send (team messaging app)

| | |
|---|---|
| **Author** | J. Rivera |
| **Status** | In review |
| **Last updated** | 2026-06-21 |
| **Stakeholders** | Eng: M. Chen · Design: P. Adeyemi · Data: L. Novak |
| **Links** | [Figma], [research deck], [JIRA epic] |

## TL;DR

We let people compose a message now and have it delivered at a chosen time. It removes the pressure to message colleagues at odd hours across time zones, and is the most-requested feature from our enterprise accounts this year.

## Problem

Our heaviest users work on distributed teams spanning 3+ time zones. They routinely finish work and want to hand off to a teammate who is offline, but they face a bad choice: send now and ping someone at 2 a.m., or hold the thought and risk forgetting it.

- In Q1 research, distributed-team users named "messaging people at the wrong time" as their #1 friction. [NEEDS INPUT: exact % from the survey]
- "Scheduled send" is the top write-in request in the feedback tool, ahead of the next item by a wide margin. [NEEDS INPUT: vote counts]
- Competitors ship this; in two lost-deal notes, its absence was cited as a reason. [ASSUMPTION: two notes is a signal, not proof — GTM to confirm scale.]

If we do nothing, off-hours notifications keep eroding the "respectful async" positioning we lead with in enterprise sales, and the feature gap keeps showing up in deals.

## Goals and non-goals

**Goals**
- Let users send messages at a time they choose, reliably, without thinking about the recipient's clock.
- Reduce off-hours notifications sent by schedulers.

**Non-goals**
- Recurring/repeating scheduled messages.
- Auto-suggesting send times based on recipient activity ("send when they're online").
- Scheduling for channels with more than [NEEDS INPUT: cap?] members.

## Success metrics

| Metric | Baseline | Target | How measured |
|---|---|---|---|
| % of weekly active senders who schedule ≥1 msg | 0% | 8% in 90 days | product analytics |
| Off-hours notifications from adopters | [NEEDS INPUT] | −25% | notification logs |
| Scheduled-message delivery success | n/a | ≥99.9% | delivery pipeline |

## Users and key use cases

**Primary user:** an IC or manager on a distributed team, ending their day while teammates are offline.

1. Finish a request at 6 p.m., schedule it to land at 9 a.m. in the recipient's morning.
2. Draft an announcement Friday, schedule for Monday so it isn't buried over the weekend.
3. Review and reschedule something already queued.

## Requirements

**Must have**
- User can pick a date and time at compose, in their own time zone, and send.
- A scheduled message is clearly marked as pending and shows its send time.
- User can view, edit, reschedule, and cancel any pending message before it sends.
- If sending fails at the scheduled time, the user is notified and the message is not silently dropped.

**Should have**
- Quick presets ("Tomorrow 9 a.m.," "Monday morning").
- Show the send time in the recipient's time zone as a secondary label while composing.

**Could have (later)**
- Recurring sends.
- Per-channel limits on how far ahead one can schedule.

## Experience and key flows

**Happy path:** compose → open the schedule control → pick a preset or custom time → message moves to a "Scheduled" view → at send time it posts and the user gets a lightweight confirmation.

**Edge cases that matter:**
- Recipient leaves the channel before send → [ASSUMPTION: message still sends; flag if they're no longer a member.]
- App is offline at send time → server-side scheduling, so delivery does not depend on the client being open.
- User edits a message after scheduling → edit applies to the pending version.

## Open questions, assumptions, risks and dependencies

- **Open:** What's the max look-ahead window — days, or months? Decision owner: PM + Eng.
- **Assumption:** [ASSUMPTION: scheduling runs server-side; client-only would fail the offline case and is out.]
- **Risk:** Delivery reliability is the whole promise — a missed send is worse than no feature. Mitigation: dedicated retry + alerting before GA.
- **Dependency:** Notifications team owns the delivery pipeline we'd hook into.

## Rollout and milestones

1. **Internal dogfood** — employees only; validate reliability and the edit/cancel flows.
2. **Beta** — opt-in for enterprise design partners; watch delivery success and adoption.
3. **GA** — gated on ≥99.9% delivery success sustained over the beta and no P0s open.
