---
name: find-verification-adequacy
description: Find checks that cannot fail: a test asserting the mock it just configured, a tautology, an expectation that re-expresses the implementation's own formula, a case that reddens on a behavior-preserving refactor, coverage with nothing asserted about the outcome. Use when a green suite is failing to catch the defects that reach production, when asked whether the tests prove anything, after fixing a bug, or when a change needs a regression test. Names the test that would fail without the fix; never runs the suite.
license: MIT
---

# Find verification adequacy

A check can pass whether or not the code is right. It asserts what was arranged rather than
what the code produced, re-uses the code's own formula as its oracle, or never reaches the
case that matters. A green suite is evidence only if its checks are capable of failing.

## The failure that matters

A test nobody can make fail guards nothing. It gives a green light that outlives the
behavior it claims to protect. Worse, it displaces the test that would have caught the
bug: the case looks covered, so nobody writes the real one. This is how a suite grows
without its fault detection moving. It gets slower while it stops proving anything.

A finding is not "coverage is low". **For each check, name the transformation of the code that would make it fail, or conclude that none exists.** When fixing a bug, it is the input that fails before the fix and passes after. Without that, the fix is unproven.

## Procedure

1. **Get the cost before reading the suite.** Take per-case timing from the runner's own
   machine-readable report, not from a stopwatch and not from your own run; per the
   standing rule, hand the timing command back rather than running it. Rank by measured
   duration plus a shared process's setup share, never by "looks slow", and work from the
   top. The cases that cost the most while proving the least are the first findings.

   Where the runner records **per-test coverage**, its greedy half is also available:
   repeatedly keep the case with the most newly-covered lines per unit of runtime until
   the coverage floor is met, then hand the plan to review. Never treat the result as a
   deletion list; see `find-redundant-checks`.

2. **State the claim each check makes**, as a property of behavior, then ask what defect
   would make it fail. No answer is the finding. This works on any framework and any
   language, without running anything.

3. **Read the slow tail against these shapes:**
   - asserting the value the mock was configured to return, or that a stub was called
   - an assertion that restates its own input, or re-derives the expectation by calling
     the code under test
   - **an expectation that re-expresses the implementation's own formula**, so an error
     common to both cancels out; the witness is a value derived by hand from the
     authored inputs
   - asserting existence, presence, non-emptiness or a changed count where the claim is a value
   - asserting a constant the code under test never touches
   - **a large golden blob with no targeted assertion**: it pins formatting, not behavior
   - **a bound with no absolute value, containment asserted in one direction only, or two
     variants asserted merely to differ where order or direction is the claim**
   - asserting call counts or internal ordering instead of the outcome
   - expecting an error type without establishing the condition that produces it
   - a `try`/`except` or `catch` that swallows the failure the test meant to observe
   - mocking the unit under test, so the double is verified and the code is not
   - **a case that re-implements the production loop in its own body**, so it stays green
     when the production function is disabled
   - **driving a full render, encode or round trip to check a number the project's own
     data decides**: that claim belongs on the data, with one smoke case that the real
     library accepts it
   - asserting only that nothing was raised, or nothing at all beyond being executed

4. **Check the cases the claim needs.** The boundary inputs (empty, single, last,
   overflowing), the failure path, and, if a regression was recently fixed, the exact
   input that reproduced it.

5. **Check isolation:** shared fixtures, module-level state, singletons, clock or
   randomness, filesystem or network dependence, and ordering between tests. A pair that
   passes alone and fails together is an ordering defect, not a flake. Same-revision
   divergence is what makes a flake; one bad run does not.

6. **Write the missing case as a sentence the user can act on**: the input, the expected
   outcome, and which current behavior it would fail on. That sentence is the deliverable.

## Fixing

- **Repairing a check is the fix here.** Tighten one that asserts the arrangement into one
  that asserts the outcome; replace a re-derived expectation with a value derived by hand;
  add the boundary or failure case that is missing; isolate a test that shares state. Write
  the case when it is asked for.
- **Hand the running back.** Per the standing rule, do not run the suite on your own
  initiative. Describe the command, the expected outcome and what a failure would mean, and
  let the user run it.
- **Never weaken a check to make it pass.** Deleting or loosening an assertion to reach
  green is the defect this audit exists to find. A case that has never failed is not
  thereby worthless: investigate its churn and risk before recommending its removal.
- **Removal is not this skill's move.** Where a case cannot fail *and* its claim is already
  guarded elsewhere, the disposition, the subsumption proof and the destructive safety
  ladder are `find-redundant-checks`.

## Report

Lead with the checks that cannot fail, costliest first:

| check | measured cost | claim | can it fail? | fix |
| --- | --- | --- | --- | --- |
| `test_save()` | 12.4 s | the record round-trips | no: asserts the mock's return | assert against the store |
| `test_reject()` | 0.2 s | malformed input is rejected | no: no assertion on the outcome | assert the error, and the input that causes it |
| `test_total()` | 0.1 s | the sum is correct | yes, at one input | add the empty and overflow cases |
| `test_end_to_end()` | 31 s | the flow works | only if the server is up | it is not hermetically isolated |

Then, per significant finding: the claim, why the current check cannot fail, and the input
plus expected outcome that would.

Close with four buckets:

- **Repaired**: the check now fails when the code is wrong.
- **Proposed**: the case to add, with its input and expected outcome.
- **Cannot fail as written**: recommend repairing it, and say whether its claim is
  guarded elsewhere (which makes it `find-redundant-checks`' business, not a quiet delete).
- **Not covered**: a behavior with no check at all.

**Ask of every check what change to the code would make the check fail.**

## Attribution

Adapted from **JoseAntonioNuevo/test-suite-doctor**
(https://github.com/JoseAntonioNuevo/test-suite-doctor), MIT, copyright (c) 2026 Jose
Antonio Nuevo. The slop-pattern catalogue and the per-case cost ranking are that project's
work; this file recasts them into the audit shape above. The repository README carries the
full license notice; a copy must stay beside this file when it is redistributed.
