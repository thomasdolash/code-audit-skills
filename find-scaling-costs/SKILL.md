---
name: find-scaling-costs
description: Find work that grows faster than its input requires: a hidden quadratic scan, a query or round trip per item, recomputation an index or cache would remove. Use when a code path, query, job or page gets slower as its data grows, times out on a large input, or degrades with volume; when asked about time or space complexity, Big-O, an O(n²) path or an N+1 query; or when a proposed optimization's complexity claim needs checking. Fixes the parts that are safely fixable.
license: MIT
---

# Find scaling costs

Cost grows faster than the input requires when a loop nests inside a loop, a query runs
once per item, a linear search repeats thousands of times, or a whole collection is
rebuilt after every change.

The expensive operation may be obvious, or it may sit three calls away. Cost belongs to
the complete execution path, not to the function you happen to be reading.

Name the growth and its cause, then say what the cost can become.

## The failure that matters

Code that is fine on small inputs can collapse at scale. A function that reads as one pass
is quadratic because a helper scans, and a fast query becomes expensive once it runs per
record.

**Name the input dimension, the derived cost and a credible replacement, or give a reason
the current cost is fine.** Both halves count. An O(n²) over five bounded elements is
rarely worth replacing, while an O(n log n) repeated in a hot path may be critical.

## Procedure

1. **Name the scaling dimensions before proposing anything.** Collection sizes (n, m),
   graph vertices and edges (V, E), request or event frequency (r), records and query
   counts, recursion depth, generated output size. Collapsing independent dimensions into
   one n hides the cost you are trying to state.

2. **Trace the complete path and how often it runs.** Follow calls through helpers,
   callbacks, and database or network boundaries, because a cheap function called O(n)
   times is not cheap. Express the total, including preprocessing, query frequency and
   amortised cost, and say which case you mean: worst, average or amortised.

3. **Derive the existing time complexity.** The growth is usually one of these:
   - a query or round trip per item (N+1), or one per record in a batch
   - recursion whose subproblems overlap and are recomputed
   - a full rebuild or invalidation where an incremental update would do
   - the same sort, filter, or aggregation repeated instead of done once
   - cost that multiplies across layers: an O(n) step inside an O(m) loop

4. **Decide whether the growth is necessary.** Compare against the real alternatives: an
   index or hash set, batching instead of per-item calls, a cache or memoisation, an
   incremental update, sorting once. Do not prefer a different algorithm because its
   Big-O looks better. Constant factors, input bounds, memory, update frequency and the
   cost of the change itself all count.

5. **Prove the replacement before recommending it.** For each finding, state the current
   cost and where it comes from; the proposed cost and why it holds; the extra space; the
   assumptions and input bounds it depends on; any change to observable behavior,
   ordering, duplicates, equality or error paths; and whether the work merely moves
   elsewhere.

6. **Separate what you measured from what you derived.** Static reading is enough for a
   clear defect, and preferred when the shape is unambiguous; when you do measure, prefer
   scaling measurements over single runs. Never state a complexity classification as
   established fact when it depends on an unknown helper, an external service or a data
   distribution. Name the dependency and say it is open.

## Fixing

- **Small, behavior-preserving optimizations: do them now.** Build the lookup set once
  instead of scanning per item, hoist an invariant out of the loop, batch a call that runs
  per record, drop a pass whose result is already in hand, sort or aggregate once instead
  of repeatedly. None of these change observable behavior, so fix them where they are.
- **Structural changes: confirm first.** Swapping a data structure, introducing a cache
  and its invalidation, changing a query strategy, or rewriting an algorithm moves memory,
  ordering, and ownership. Show the cost before and after, with the assumptions, and get
  agreement before rewriting.
- **Preserve behavior unless the behavior is the bug.** Keep observable ordering,
  duplicate handling, equality semantics, and error paths. Do not optimize a path you were
  not asked about.

## Report

Lead with the findings, worst first. A table is the clearest:

| location | scaling input | current | proposed | impact |
| --- | --- | --- | --- | --- |
| `processItems()` | n items | O(n²) | O(n) | high: repeated lookup |
| `loadRecords()` | n records | O(n) queries | O(1) queries | high: round trip per row |
| `updateView()` | n elements, r updates | O(rn) | O(r log n) | medium: full rebuild |

Then, per significant finding, give the hidden source of the cost, the proposed
replacement, its space and correctness trade-offs, and whether you measured it or derived
it.

Close with four buckets:

- **Fixed**: done, behavior-preserving, with what it was and what it is now.
- **Proposed**: worth changing but not safe to do unasked, most impact first.
- **Uncertain**: suspected, but blocked on profiling or on an input bound nobody has
  stated yet. Say what would settle it.
- **Not worth changing**: suboptimal and staying that way. Say why: bounded, rare, or
  the replacement costs more than it saves.

**Do not report a cost you have not established, and do not recommend a change whose
improvement you cannot argue.**