---
name: find-failure-atomicity
description: Find operations that leave partial state or duplicate side effects after an exception, cancellation, timeout, crash or retry: a charge applied twice, a file published before its index, an uncapped retry against a failing dependency. Use when a failure leaves half-applied state, when a retry applies an effect twice, or when auditing multi-step updates, transactions, retries and backoff, filesystem or database writes, queues, external APIs, cleanup, restart recovery, or exactly-once and idempotency claims. Fixes the parts that are safely fixable.
license: MIT
---

# Find failure atomicity defects

An operation is failure-atomic at a specified boundary when failure does not leave an
externally inconsistent partial result. Some operations provide rollback; others use
durable steps and compensation. "Atomic" must be defined against the actual observer and
persistence model.

Probe the gaps between effects as well as the `catch` blocks.

## The failure that matters

A function updates memory and then fails before persisting; a retried request applies a
charge twice; a file is published before its index is updated; cancellation skips an
irreversible side effect. The success path is correct, but failure exposes contradictory
state.

**Report an interruption point, the partial state it leaves, and the promised recovery
behavior it violates.**

## Procedure

1. **Define the operation's commit boundary and observers.** List what must appear
   all-or-nothing to callers, subsequent requests, other processes, or after restart. Do
   not assume full rollback if the intended contract is eventual reconciliation.
2. **List every effect in order.** Include in-memory mutation, files, DB writes, network
   calls, queue acknowledgments, events, and external services. Mark which are reversible,
   durable, and idempotent.
3. **Place an interruption after each effect.** Consider exceptions, timeout, cancellation,
   lost response, process termination, and retry. Trace what survives and what cleanup
   actually runs; `finally` does not run after every kind of crash.
4. **Follow recovery and retry end to end.** Check whether recovery can distinguish "not
   applied" from "applied but acknowledgment lost." Verify deduplication keys, commit
   markers, outbox/inbox records, or compensations against actual persistence boundaries.
   **Then check the retry policy itself:** whether the work is safe to repeat at all,
   whether attempts are capped and budgeted, whether jitter spreads a fleet of clients,
   whether a non-retryable failure is retried anyway, and whether a timeout and a retry can
   both fire for one logical operation. An uncapped retry against a failing dependency
   turns an incident into an outage, and no idempotency key fixes a retry storm.
5. **Construct the smallest failure witness.** Show state before, effects completed,
   interruption, state afterward, and the next retry/restart result. Do not assert
   exactly-once semantics where only at-least-once delivery is provided.
6. **Select the narrow repair.** It may be reordered publication, an existing transaction,
   a durable idempotency key, or a compensating step. Confirm that moving an effect does
   not create a new failure window.

## Fixing

- **Localized broken cleanup or commit order: fix now** when the contract is clear and the
  correction cannot change external protocol semantics. Add a fault-injection regression
  test at the failing gap.
- **Transaction/protocol redesign: confirm first.** Introducing new durable state,
  idempotency keys, two-phase workflows, or compensation changes ownership and recovery
  semantics.
- **Do not call an operation atomic merely because it uses a lock.** Concurrency exclusion
  and crash consistency are different guarantees.

## Report

| operation | failure point | surviving partial state | violated promise | repair |
| --- | --- | --- | --- | --- |
| `publish()` | after file rename, before index commit | visible file, absent index | publication consistency | commit protocol |
| `charge()` | retry after a timeout, before the response | one charge applied twice | exactly-once claim | idempotency key, or no retry on that error |

Include the interruption timeline, observed persistence assumptions, retry outcome, and
test status. Separate confirmed partial-state defects, unverified durability assumptions,
and acceptable eventual states.