---
name: find-liveness-defects
description: Find work that can stay pending forever through legal transitions and individually bounded steps: a task repeatedly deprioritized, a retry that never advances, an item a busy queue never reaches. Use when something hangs or never finishes, when a queued item never runs, or when auditing queues, retries, schedulers, locks, wait graphs, polling, worker progress, async workflows, starvation, livelocks, deadlocks or nontermination. Fixes the parts that are safely fixable.
license: MIT
---

# Find liveness defects

Safety asks whether the system does something wrong. Liveness asks whether it eventually
makes required progress. A workflow can have only legal transitions and a cheap
per-iteration cost and still never complete a job.

Look for the missing progress guarantee; an infinite-looking loop is only one symptom.

## The failure that matters

A task is repeatedly deprioritized, two workers wait on each other, a retry cycles
without advancing, or a queue is continually serviced while one item never runs. Valid
states and bounded operations do not imply eventual completion.

**Report the work that must progress, the assumptions under which progress is promised,
and a reachable stalled execution that defeats it.**

## Procedure

1. **Define the progress obligation.** "Every accepted job eventually completes or fails,"
   "every held resource is eventually released," or "each retry eventually terminates under
   a finite error budget." State environmental assumptions, fairness, service
   availability, and any legitimate indefinite waits.
2. **Identify wait and scheduling dependencies.** Build a wait-for graph for locks/resources
   and trace queues, priorities, retries, backoff, timers, wakeups, and event delivery.
   Include cancellation and shutdown behavior where they affect progress.
3. **Look for cycles without progress.** Deadlock (actors blocked), livelock (actors active
   without useful advancement), starvation (some work perpetually bypassed), and unbounded
   retry. Keep these distinct from a state-machine's missing legal transition.
4. **Find a decreasing measure or fairness argument.** For loops, show which quantity moves
   toward completion; for queues, show why each eligible item eventually receives service.
   If the claim depends on fair scheduling, label it conditional rather than absolute.
5. **Construct a witness schedule.** Give the smallest state and event sequence that can
   repeat or remain blocked indefinitely. Consider whether external guarantees (timeouts,
   queue fairness, lock semantics) exclude the witness.
6. **Choose the progress mechanism.** Bounded retries, fair selection, deadlines, wakeup
   signaling, cycle breaking, or explicit terminal failure may resolve the defect;
   evaluate resulting safety and ordering changes.

## Fixing

- **Local progress gaps: fix now** when a missed wakeup, missing terminal branch, or
  bounded retry omission has a clear existing contract. Add a deterministic
  schedule/termination test.
- **Scheduler and protocol changes: confirm first.** Fairness policies, retry budgets,
  lock ordering, and timeout behavior change semantics. Show which progress promise is
  being restored.
- **Do not label long-running work a hang without a progress criterion.** An intentionally
  persistent server loop is not nontermination by itself.

## Report

| obligation | blocking cycle/schedule | assumed guarantee | evidence | consequence |
| --- | --- | --- | --- | --- |
| all accepted tasks serviced | high-priority arrivals always preempt L | no fairness guarantee | repeating queue trace | task starvation |

Report the precise liveness condition, witness or proof gap, and any required environmental
fairness assumption. Distinguish demonstrated no-progress paths, conditional progress, and
intentional waiting.