---
name: find-concurrency-ordering
description: Find wrong outcomes caused by concurrent or reordered operations: races, stale reads, TOCTOU, lost updates, missing happens-before. Use when the result is wrong only sometimes and happy-path tests never see it, or when working with threads, async tasks, callbacks, message queues, worker pools, shared state, cancellation or GPU/CPU synchronization. Fixes the parts that are safely fixable.
license: MIT
---

# Find concurrency and ordering defects

Two paths can be correct in isolation and wrong when their events interleave or complete
out of order. A race is a reachable schedule whose result violates a required ordering or
state guarantee; the presence of async code is not the defect.

Identify the shared resource and the required order, then find a witness schedule.

## The failure that matters

An older request overwrites newer results, two workers claim the same job, validation
reads a value before another task changes it, or a buffer is consumed before its producer
finishes. Happy-path tests run the events in the expected order and miss the defect.

**Report a concrete interleaving with the wrong observable outcome, or state the
synchronization assumption you could not resolve.**

## Procedure

1. **List execution actors and shared resources.** Threads, event-loop tasks, processes,
   GPU streams, callbacks, queues, and external writers. Distinguish actually concurrent
   operations from code serialized by a guaranteed dispatcher.
2. **Name the required relation.** Must updates be linearizable, ordered by request
   generation, mutually exclusive, or merely eventually consistent? Identify the observer
   and contract before calling reordering wrong.
3. **Map synchronization and happens-before edges.** Locks, awaits, transactions,
   barriers, message order, atomics, ownership transfers, and callback sequencing. Check
   what each primitive truly guarantees, including visibility and cancellation.
4. **Enumerate critical interleavings.** Focus on read-modify-write, check-then-act,
   concurrent creation, completion versus cancellation, late responses, and teardown
   versus callback. Trace the shortest witness schedule event by event.
5. **Check stale state and ownership.** Closures may capture obsolete generations; locks
   may protect one field but not its paired data; buffer reuse may race with consumers.
   Verify that the fix closes the demonstrated schedule rather than moving it.
6. **Validate reachability.** Use controlled barriers, deterministic scheduling, or
   instrumentation where practical. If runtime ordering is undocumented or external,
   report the dependency as uncertain rather than inventing a race.

## Fixing

- **Small ordering guards: fix now** when the intended winner is explicit, such as
  discarding obsolete responses by generation or guarding a missing paired mutation. Add a
  test that forces the bad schedule.
- **Ownership and synchronization redesign: confirm first.** New locks, lock order,
  transactions, worker topology, and actor boundaries can create deadlocks or change
  throughput and semantics.
- **Do not serialize everything by default.** Protect the actual invariant with the
  smallest sufficient mechanism.

## Report

| actors | shared resource | witness order | wrong result | synchronization gap |
| --- | --- | --- | --- | --- |
| requests A/B | current search results | A starts → B starts → B ends → A ends | stale A displayed | no generation check |

Provide numbered event schedules, what ordering was assumed, and how a proposed fix
prevents *that* schedule. Distinguish confirmed races, potential races blocked on
scheduler guarantees, and benign reordering.