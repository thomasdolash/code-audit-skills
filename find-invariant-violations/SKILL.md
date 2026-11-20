---
name: find-invariant-violations
description: Find properties that must hold across every operation and locate the paths that break them: an item lost while partitioning, two owners claiming one resource, a completed job still counted as pending, a reused index that no longer names the same object. Use when you can name a property that must always hold, conservation, uniqueness, one owner, totals that reconcile, and suspect a path that breaks it; when results are subtly wrong; or when auditing algorithms, partitioning, resource accounting or ownership. Fixes the parts that are safely fixable.
license: MIT
---

# Find invariant violations

An invariant is a property that must hold at a defined boundary, whether before and after
an operation, at commit, or throughout a lifecycle. It may concern counts, membership,
ownership, ordering, or relationships between structures. The fact that every function
returns successfully does not prove the invariant survives.

Find the invariant independently of the current implementation, then look for a reachable
operation that breaks it.

## The failure that matters

The algorithm returns plausible results while silently violating a system-level truth. An
item is lost during partitioning, two owners claim one resource, a completed job still
counts as pending, or a mapping contains an index that no longer identifies the same
object.

**Report the property, where it must hold, and a reachable counterexample, or state
plainly that preservation remains unproven.**

## Procedure

1. **Derive the properties from intent, not from the code being audited.** Consult
   requirements, external contracts, callers, and domain rules. Examples: `output multiset
   = input multiset`, every live record has exactly one owner, and committed totals
   reconcile. Do not invent an invariant merely because it would simplify an
   implementation.
2. **Choose its boundary and quantify it.** Does it hold at all intermediate steps, only
   between public calls, or only after successful commit? Specify valid inputs and
   exceptions. Temporary internal violations may be intentional.
3. **Identify the base case and every mutating operation.** Follow creation, update,
   deletion, move, split, merge, reset, rollback, and recovery, including mutations in
   helpers. An invariant is preserved only if initialization establishes it and each
   reachable transition preserves or restores it at the required boundary.
4. **Attack the weakest paths.** Empty and singleton inputs, duplicates, collisions,
   repeated application, partial failure, conditional skips, early returns, and boundary
   values often reveal violations. Trace both data and counts, not only control flow.
5. **Construct a minimal counterexample.** Name the starting valid state, operation
   sequence, resulting state, and exact property broken. Use a small test or trace when
   available; otherwise mark the result as reasoned, not executed.
6. **Check whether the property is genuinely required.** Distinguish an incorrect result
   from a valid alternative ordering, intentional deduplication, or an intermediate state
   that is repaired before exposure. Do not treat a missing assertion or lack of a formal
   proof as a confirmed bug.

## Fixing

- **Small, behavior-preserving corrections: do them now.** Fix the lost update, missed
  bookkeeping branch, wrong set operation, or omission that demonstrably violates the
  intended property. Add a focused regression test asserting the property.
- **Structural changes: confirm first.** Changing ownership, data models, transactional
  boundaries, or algorithm structure can alter observable behavior. Show the
  counterexample and propose the smallest correct repair before rewriting.
- **Preserve intent over current output.** If the existing output is wrong, a fix need not
  reproduce it. Do not impose stricter invariants than the domain requires.

## Report

Lead with the broken or unproven properties:

| invariant | required boundary | violating path | evidence | impact |
| --- | --- | --- | --- | --- |
| Each input appears once | after `partition()` | duplicate-key branch | 2-item counterexample | high: lost work |

Then give the minimal trace, whether it was executed, what was fixed, and what requires
design approval. Separate confirmed violation, unproven preservation, and property not
actually required.

A property with no counterexample and no violated contract is not a confirmed defect.