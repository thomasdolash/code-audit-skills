---
name: find-nondeterminism
description: Find output that varies between runs although the input did not: iterating a map or set, hash order deciding sequence, an unstable sort, directory or glob listing order, a clock or random value entering something that should be fixed, a cache key missing an input. Use when the same input gives a different result, a test is flaky, a diff is noisy, or a build is not reproducible; when auditing sorting, grouping, caching, hashing and summarization. Fixes the parts that are safely fixable.
license: MIT
---

# Find nondeterminism

A nondeterminism defect is a **value or an order that depends on something other than the
input**. The cause can be hash order, a set's iteration, an unstable sort, the filesystem's
listing order, the clock, a global counter, or completion timing.

It rarely announces itself, and it turns a real defect into a flaky one.

## The failure that matters

The bug appears in one run and not the next, is dismissed as noise, and outlives every attempt
to reproduce it. Worse, any check that compares output across runs stops being able to fail
loudly, because the differences it reports are always blamed on ordering.

**For each value that varies, name the source of the variation, whether anything observes it,
and what would make it stable, or the reason the variation is harmless.**

## Procedure

1. **Identify the outputs compared across runs.** Serialized data, human diffs, checksums,
   cache contents, log text asserted in tests, generated files, summary or report text, and
   anything fed into a "no change" comparison.

2. **Find the order dependence.**
   - iteration over a map, dictionary, or set whose order is unspecified
   - a sort whose comparator admits ties, or a ranking with no tiebreaker
   - grouping that emits members in insertion or hash order
   - items processed in completion order rather than input order
   - the first match from a hash-backed structure
   - directory, glob, environment, or registry listing order

3. **Find the hidden state.** Wall clock or elapsed time inside a value that should be a
   function of the input; random or generated identifiers feeding output; a global mutable
   counter; a cache whose key omits a contributing input; parallelism that decides order;
   locale, encoding, or environment.

4. **Decide whether it is observed.** Nondeterminism matters when something compares, caches,
   asserts, signs, or persists the value, or when the variation *hides* a wrong answer by
   sometimes producing the right one. Say which case it is.

5. **Prefer the narrow fix**, such as a total order (stable sort plus a tiebreaker), a
   canonical serialization, an injected clock or seed, or a cache key that covers its inputs.
   Do not freeze the program to make one output stable.

## Fixing

- **Local and unambiguous: fix now.** Add the missing tiebreaker; iterate a map's keys in
  sorted order where the result is emitted or compared; use a stable sort where order is
  observed; make a cache key cover every input that changes its value; order parallel results
  by input index.

- **Confirm first when interfaces or cost change.** Injecting a clock or seed through
  callers, changing a public output order, or adding an ordering step to a hot path all reach
  beyond the function, the last one is also a `find-scaling-costs` question.

- **Preserve deliberate variation.** A nonce, a jitter, a random sampling order, or a
  deliberately time-dependent value is not a finding. Say so as a designed choice rather than
  leaving it in the list.

## Report

Lead with the findings:

| output | varies with | source | fix |
| --- | --- | --- | --- |
| serialized summary | run | map iteration order | emit sorted keys |
| merged diff | machine | parallel completion | order by input index |
| cached decision | caller | key omits one input | widen the key |
| test assertion | run | clock in the value | inject the clock |

Then, per finding: what varies, what observes it, and whether the variation can hide a wrong
result, a flake that sometimes passes is not benign.

Close with four buckets:

- **Fixed**: an order or a value is now stable; behavior otherwise unchanged.
- **Proposed**: needs an interface or cost decision; most impact first.
- **Benign**: varies and nothing observes it, stated as such.
- **Hides a defect**: the variation masks a real wrong answer; say what to look for next.

**A result that changes between runs is not a result until you can say why.**