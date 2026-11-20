---
name: find-complex-functions
description: Find functions that do more than one thing or are harder to follow than their job requires: a body you cannot hold in one read, deeply nested conditionals, a long chain that should be three named steps, one that mixes high-level policy with low-level parsing. Use when a function is hard to follow or review, when asked about function length, nesting, cyclomatic complexity or local variables, or before adding another branch to one already difficult to read. Decomposes the parts that are safely decomposable.
license: MIT
---

# Find complex functions

A complex function asks the reader to hold more than its job requires. It carries several
responsibilities, branches inside branches, statements that mix levels of abstraction, or
more locals than anyone can track.

Length is a symptom of that, not the defect. A 90-line flat dispatch over a table is easy; a
40-line body with four nested loops is not. **Find what the reader is being asked to hold, and name the steps that should be separate.**

## The failure that matters

Complexity is paid by whoever next changes the function, not by whoever wrote it. It shows
up in a change that must be understood end to end, a branch added in the wrong place, or a
bug that survives review because the reviewer stopped reading at line 60.

A finding is not "function too long". **For each finding, describe what the body does, how many things that is, and the steps you would extract, or give an explicit reason the shape is right.** A long function implementing one cohesive algorithm can be correct.

## Procedure

1. **Measure the body, then read it.** Record physical lines, substantive lines (excluding
   blanks and comments), the longest nesting depth, and the number of local variables.
   Thresholds, for hand-maintained code:

   | function size | reading |
   | --- | --- |
   | under 20 | ideal: the body reads as one step |
   | 20 to 50 | normal range |
   | 50 to 80 | review: say whether the parts separate |
   | over 80 | a split is normally expected |
   | over 200 | high-priority complexity finding |

2. **Count what the line count stands in for.** For each flagged function:
   - **Responsibilities.** If the honest description needs "and", it is two functions.
     A stronger signal is two different callers, actors or reasons driving it, which is the seam.
   - **Branch paths.** Count the if, loop, match arm and catch. A handful is fine; paths
     that multiply across nested branches are the signal, and they are what makes a short
     function hard.
   - **Nesting depth.** Depth lowers the acceptable length: the ceiling falls as complexity
     and indentation rise. Three levels is the usual tolerance; past it, restructure rather
     than continue. The same 60 lines is fine at depth 1 and wrong at depth 4.
   - **Local variables.** Past roughly seven to ten, the reader is tracking more than they
     can hold; that is a finding on its own.
   - **Levels of abstraction.** Mixing high-level policy with low-level parsing, indexing,
     or pointer arithmetic in one body is a defect regardless of length.
   Lines still matter where those counts miss: a nested chain of calls can be one
   statement, zero branches, and sixteen lines that must be read in order.

3. **Name the steps you would extract, and which technique fits.** Four carry most of the
   work:
   - **Extraction**: move a nested block into a named helper that owns one concern.
   - **Guard clauses and early returns**: reject the invalid first, so the body runs at
     one level of indentation instead of inside three conditionals.
   - **A flatter data structure**: replace nested traversal with one pass over an index,
     map, or pre-built list.
   - **A library**: where the nesting is a hand-rolled algorithm (sorting, grouping,
     matching), the code you delete is the win.
   The rule is the same for chains rather than loops, such as promises, observables and
   pipelines: compose them, and never nest a subscription or a pipeline inside another.
   Then say what each helper would be called and what it would own. "Refactor this
   function" is not a finding, and neither is a split that only moves lines behind names.
   If the extracted step still needs the whole body's context, it is not a step.

4. **Check the argument list and return shape.** Parameter count and a wide tuple or
   `bool` return are the same defect from the interface side: the function is doing
   several jobs, or its result is ambiguous. Note it even when the body is short.

5. **Preserve behavior when you propose a split.** Extraction must keep ordering,
   short-circuiting, error propagation, mutation visibility, and any side effect's
   timing. A decomposition that changes when a lock is released or a write happens is a
   rewrite, not an extraction.

## Fixing

- **Small, behavior-preserving extractions: do them.** Pull a nested block into a named
  helper, replace a deep `if` pyramid with guard clauses and early returns, hoist a mixed
  low-level computation out behind a name, split one function into the two steps its own
  comment already describes. None of these change observable behavior.
- **Structural changes: confirm first.** Changing a signature, moving ownership, merging
  or re-splitting modules, or replacing a state machine with types changes callers. Show
  the shape before and after first.
- **Preserve behavior unless the behavior is the bug.** Keep ordering, short-circuiting,
  error paths, and mutation timing. Do not decompose a function you were not asked about.

## Report

Lead with the findings, worst first:

| function | substantive lines | depth | branch paths | assessment |
| --- | --- | --- | --- | --- |
| `loadAll()` | 210 | 5 | 21 | split: four stages, each already a comment |
| `applyRules()` | 68 | 4 | 14 | guard clauses first, then split the two passes |
| `parseTable()` | 95 | 1 | 3 | large but flat; leave |
| `run()` | 24 | 3 | 9 | short and dense: extract the retry loop |

Then, per finding: what the body does, the steps you would extract with the names you
would give them, and what the split costs: indirection, moved state, changed call sites.

Close with four buckets:

- **Decomposed**: done, behavior-preserving, with what moved where.
- **Proposed**: worth extracting but not safe to do unasked, most impact first.
- **Large but flat**: over the size threshold and staying whole, with the reason.
- **Not a finding**: generated, or a table-like body with no branching.

**A short function is not automatically a good one, and a long flat one is not automatically bad.** Branch count, depth and mixed levels of abstraction are the defects; length is how they are usually paid for. Three levels of nesting is a tolerance line rather than a law, and it exists so the steps are reconsidered when the depth climbs.
