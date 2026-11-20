---
name: find-numerical-instability
description: Find computations that become wrong or untrustworthy through finite precision: cancellation, overflow, underflow, ill-conditioning, unstable reductions, invalid numerical assumptions. Use when the code is deterministic and still gets the number wrong, when a total drifts as it grows, a threshold flips on rounding, or a result is right at small scale and wrong at large; when auditing floating-point code, scoring, geometry, statistics, GPU kernels or accumulators. Fixes the parts that are safely fixable.
license: MIT
---

# Find numerical instability

Mathematically equivalent formulas can behave very differently on a real machine. Finite
precision, operation order, range, and input conditioning decide whether a numeric result is
trustworthy.

Audit for the accuracy the domain requires, not for bitwise agreement between implementations.

## The failure that matters

Subtracting nearly equal large values erases useful precision. A sum can overflow before
normalization, small terms vanish in a long reduction, an epsilon comparison picks the wrong
branch, and GPU execution order moves a threshold decision.

**Report an input region and the mechanism by which the result goes wrong, with evidence
that the error breaches a stated tolerance or decision.**

## Procedure

1. **Define the numeric types and the domain's correctness criterion.** Record precision,
   permitted ranges, absolute and relative tolerances, overflow policy, reproducibility
   requirements, and any exact integer invariant. Bitwise parity is not assumed necessary.
2. **Trace the sensitive computation.** Follow conversions, intermediate ranges, summations,
   normalization, branching thresholds, and parallel reductions. Identify operations that
   amplify input or rounding error.
3. **Probe known failure patterns**, including catastrophic cancellation, subtractive
   comparisons, division by a near-zero value, repeated accumulation, exponentiation
   extremes, mixed precision, and integer overflow before a later cast.
4. **Distinguish instability from conditioning.** An ill-conditioned problem can be sensitive
   even under a stable algorithm. Where possible, compare against a trustworthy higher-precision
   reference, analytic bound, or domain invariant.
5. **Build a discriminating input.** Show the calculation, expected or bounded result, observed
   or derived finite-precision result, and the wrong downstream decision. Vary input
   magnitudes and reduction order rather than testing one arbitrary sample.
6. **Compare the credible remedies.** Algebraic reformulation, compensated sums, rescaling,
   robust predicates, stable reductions, and wider intermediates all cost runtime and
   semantics, so explain why the proposed method improves the stated criterion.

## Fixing

- **Small local numerical defects: fix now** when a proven overflow, a wrong cast order, or a
  standard stable formulation preserves intended semantics. Add tolerance-based regression
  tests with extreme inputs.
- **Precision or algorithm changes: confirm first.** Wider types, deterministic reductions,
  new tolerances, and revised threshold rules may affect speed, memory, portability, and
  accepted output.
- **Do not require mathematical parity.** Two implementations can both be correct within
  tolerance, or both wrong relative to intent.

## Report

| computation | input region | mechanism | failure against criterion | remedy |
| --- | --- | --- | --- | --- |
| `variance()` | large mean, tiny spread | cancellation | negative variance outside tolerance | centered/stable update |

State the reference used, precision, tolerance, and whether the issue is demonstrated,
bounded analytically, or only suspected. **A floating-point difference alone is not a
defect.**