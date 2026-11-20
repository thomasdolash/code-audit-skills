---
name: find-resource-lifecycle
description: Find resources acquired and not released on every path: a handle, connection, lock, listener, subscription, thread or temp file still held when the code returns early, throws or is canceled; double release; use after release; teardown that never propagates to the owner. Use when something leaks, when handles or connections run out, a file stays open, a listener keeps firing after its component is gone, or a pool is exhausted; when auditing acquisition and cleanup, destructors, pools and cancellation.
license: MIT
---

# Find resource lifecycle

A resource-lifecycle defect is an **acquisition whose release does not happen on every path
out of the scope**. The release may happen twice, or too early, or not at all when the owner
disappears.

Locks, file handles, connections, subscriptions, timers, tasks, temp files and native buffers
all share one shape, in which something is taken and something must give it back.

## The failure that matters

Leaks are invisible until exhaustion, and the symptom appears far from the cause. A process
slows over hours, a lock deadlocks the second time, a subscriber fires after its component is
gone, a build fails on "too many open files".

**For each acquisition, list the paths that release it and the paths that escape, or give a
reason its lifetime is bounded.** A resource whose release is guaranteed by the runtime, or
whose count is capped, is not a finding.

## Procedure

1. **Inventory the acquisitions.** Handles and descriptors, sockets and connections,
   transactions, locks and semaphores, subscriptions and listeners, timers, threads and
   tasks, temp files and directories, GPU or native buffers, and any allocation with an
   explicit owner.

2. **For each, list the paths out of the scope.** Success return, early return, throw or
   panic, cancellation, timeout, a callback that never fires, a caller that abandons the
   future, process exit. State which path releases it. The path that neither releases nor
   hands ownership on is the finding.

3. **Look for the shapes:**
   - release written after a body that can return or throw, with no scope guard,
     `defer`, `finally`, or context manager
   - acquire inside a loop or a retry, releasing only on the happy iteration
   - a lock held across a call that can block, re-enter, or call back into the holder
   - a subscription registered per mount, render, or request, with no teardown that
     matches its registration
   - a resource released on the error path but leaked on the success path, or the reverse
   - double release, where the second is not idempotent
   - use after release, including a callback that captures the closed handle

4. **Check that release follows ownership.** When the owner is canceled, destroyed, or
   restarted, does release still run? Are errors from release itself swallowed and
   forgotten? Is release attempted on paths where acquisition never succeeded?

5. **Bound what cannot be guaranteed.** Where release depends on a path nobody can force,
   such as a remote peer or a crash, is there a cap, whether pool size, max open, TTL, or a
   reaper? Things that are only ever acquired, never released, are `find-unbounded-growth`.

## Fixing

- **Fix the local, unambiguous pairings now.** Move release onto every path out of the
  scope; release before returning from a failed branch; take the resource after the guard
  clauses rather than before; make release idempotent where a double release is possible;
  close what you open inside the same block.

- **Confirm first when lifetime or ownership changes.** Adding a pool or a reaper, widening
  or narrowing a lock's scope, propagating cancellation, or making destruction synchronous
  all change ordering and concurrency. Show what is held and for how long.

- **Preserve behavior unless the behavior is the bug.** Do not widen a lock that is
  deliberately held across a sequence, and do not change when a resource is released without
  saying so.

## Report

Lead with the findings:

| resource | acquired in | released on | escapes at |
| --- | --- | --- | --- |
| file handle | `open()` | success, throw | early return at the empty-input branch |
| subscription | `onMount()` | nothing | no teardown exists |
| connection | `withClient()` | success | timeout path |
| lock | `apply()` | every path | none: but held across a blocking call |

Then, per finding: what is held, how many can be held at once, which path leaks, and what the
caller or owner does when the resource runs out.

Close with four buckets:

- **Fixed**: release now happens on every path; behavior otherwise unchanged.
- **Proposed**: needs an ownership or lifetime decision; most impact first.
- **Bounded**: capped or genuinely process-lifetime. State the bound, or name it as missing.
- **Not a finding**: owned by the runtime or the caller, released by construction.

**Every acquisition needs an answer to one question: which path releases this?**