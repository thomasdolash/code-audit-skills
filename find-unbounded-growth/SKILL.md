---
name: find-unbounded-growth
description: Find anything whose size is set by how long the process runs rather than by the work in hand: a queue that can outrun its consumer, a cache with no eviction, a session, history or log store with no retention, temp files never reaped, registrations that repeat per call. Use when memory climbs over time, a process slows the longer it runs, a store keeps growing, a restart clears a problem that returns, or when auditing caches, queues, histories, logs and long-lived processes. Fixes the parts that are safely fixable.
license: MIT
---

# Find unbounded growth

A structure or store whose size is set by elapsed time or traffic rather than by the work in
hand grows without a bound: a queue with no backpressure, a cache with no eviction, a
history with no retention, or a registration that repeats. The acquisition is usually
correct; the bound is what is missing.

## The failure that matters

The growth is invisible in a short run and certain in a long one, so it survives every test.
It shows up as a process that degrades over hours, a store that grows without limit, or a
problem that a restart "fixes" and that then returns.

A finding is not "this collection is large". **For each accumulator, report what bounds it now and what bound is appropriate, or the construction that makes its total finite.** "It will not matter in practice" is true only with the bound that makes it so, so name the bound or the finding stands. Anything acquired and never released at all belongs to `find-resource-lifecycle`.

## Procedure

1. **Inventory the accumulators.** In-memory collections and buffers, queues, caches,
   session, history and event stores, indexes, log buffers, temp files and directories,
   retained objects (listeners, subscriptions, closures holding references), connection or
   worker pools, and anything appended inside a loop.

2. **Name what bounds each one.** A maximum size, an eviction policy, a TTL, a consumer
   that keeps up by construction, a retention job, or nothing. "Nothing" is the finding.

3. **Test the producer against the consumer** for queues and streams: if the consumer
   stalls, does the producer block, drop or grow? Which of those is intended, and is that
   what the code does? A bound that exists only because the consumer is never slow is not a
   bound.

4. **Check the retention path where one exists.** Is it in the same process, so it stops
   running under exactly the load that grows the store? Does it evict the right thing, and
   is the policy keyed on the property that matters, oldest or least valuable? Does it also
   reclaim the disk and any secondary index, or only the in-memory copy?

5. **Check what "bounded" rests on.** A counter reset on restart; a cap that is per
   instance while the instances are per request or per session; a TTL longer than the
   process's useful life; an index rebuilt and re-appended. Each looks like a bound and
   is not one.

6. **Distinguish the growth that is correct.** A list compiled once at start, a table whose
   size is genuinely the input's, and data discarded at the end of a request are all bounded
   by construction. Say so, and name the construction that bounds them.

## Fixing

- **Local and unambiguous: fix now.** Add the missing cap or eviction where the policy is
  obvious; drop a reference retained after its use; dedupe a registration that repeats per
  call; reap a temp artifact on the path that created it.

- **Confirm first when the policy is a choice.** Which entries to evict, how long to
  retain, whether to drop or block the producer, and whether retention moves to disk each
  change behavior, cost and failure modes. Propose the bound and who owns it before
  implementing.

- **Respect an existing retention convention.** Where the project already has a retention
  or eviction pattern, follow it rather than introducing a second scheme.

## Report

Lead with the findings:

| accumulator | grows with | bound now | bound needed |
| --- | --- | --- | --- |
| pending queue | producer rate | none | cap plus backpressure, or shedding |
| session store | runs per day | removed when it settles | retention window |
| exported temp dir | every export | none | reap on success |
| tool registry | process lifetime | index built at start | correct: bounded by construction |

Then, per finding: what accumulates, how fast, what observes the growth, and what breaks
first when it does: memory, handles, latency or disk.

Close with four buckets:

- **Fixed**: a bound now exists; behavior otherwise unchanged.
- **Proposed**: the policy is a decision; give the bound and who owns retention.
- **Bounded by construction**: state the construction.
- **Bounded only until restart**: say so; it is a delayed version of the finding.

**Every accumulator needs a bound, or a stated reason it cannot outlive its input.**
