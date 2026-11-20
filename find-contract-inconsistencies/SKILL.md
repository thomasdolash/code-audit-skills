---
name: find-contract-inconsistencies
description: Find components that agree on types but not on what an operation promises: a precondition callers assume and the callee never establishes, a timeout one layer reads as cancellation and another as retryable, a specification allowing duplicates an implementation silently removes. Use when auditing integration boundaries, migrations, feature behavior, backward compatibility, protocol versions, search/sort/index preconditions, external inputs, or locally-correct functions that are wrong in the real call path. Fixes the parts that are safely fixable.
license: MIT
---

# Find contract inconsistencies

Components can agree on types and still disagree about **what an operation promises**. One
layer reads a timeout as cancellation and another as a retryable failure, a specification
allows duplicates that an implementation silently removes, and an algorithm is correct only
under a precondition its real callers never establish.

Compare observable obligations across producers, consumers, and authoritative
requirements, and trace each precondition backward until it is established, or until a
reachable entry path breaks it.

## The failure that matters

Two reasonable implementations encode incompatible definitions of success, ownership,
ordering, error meaning, or accepted input, or a helper assumes sortedness, uniqueness, or
non-emptiness that no caller guarantees. The tests for each component pass, but the full
workflow violates the user-visible contract.

**Report the conflicting promises, or the precondition nobody establishes, with their
sources and a reachable behavior that exposes the disagreement.**

## Procedure

1. **Pick one operation or interface and find its contract authorities.** These are the
   requirements, explicit API docs, schemas, interoperability rules, callers, integration
   tests, and observed legacy behavior. Record source age and authority, because old behavior
   can be a defect rather than a specification.
2. **Write the behavioral contract in observable terms.** Cover valid inputs, output meaning,
   ordering, duplicates, side effects, atomicity, error handling, retry semantics,
   compatibility, and version boundaries. Add any algorithmic prerequisite
   (`sorted by comparator C`), environmental prerequisite (`transaction active`), or caller
   promise (`IDs unique within scope S`). Separate required behavior from implementation
   detail.
3. **Compare each layer's interpretation, and follow every entry path backward.** Trace
   producer → transport/storage → consumer and alternate implementations; trace public
   APIs, callbacks, retries, alternative constructors, deserialization, cached inputs, and
   feature branches. Do not stop at the immediate caller when its own inputs are unverified.
4. **Identify where each prerequisite is established, and what can invalidate it.** For a
   constructor invariant, a schema constraint, a sort, a validation, a lock, or a strong type,
   verify that it uses the same comparator, units, identity domain, and lifetime as the
   consumer. Filtering, mutation, merging, partial updates, concurrency, type conversion, and
   reuse after reconfiguration can erase a guarantee that was true a moment ago.
5. **Resolve apparent contradictions cautiously.** Tests may encode a bug, prose may be
   outdated, and newer contracts may intentionally supersede older ones. Majority behavior and
   mathematical parity are not the authority.
6. **Build an end-to-end witness.** Use an input that distinguishes the interpretations, or a
   valid system input whose call chain is missing the prerequisite, and follow it through the
   real path to the incorrect observable result. A possible call in isolation is not
   enough if the system cannot reach it. If the authority is genuinely unresolved, report a
   decision needed instead of declaring one implementation wrong.
7. **Name the owner of the contract and the place it should be enforced.** Say which layer
   must validate, normalize, reject, translate, or preserve the semantics, and which tests
   should anchor the shared interpretation. Establish the property once at the trusted
   boundary, or strengthen the API contract or the type, rather than scattering redundant
   checks.

## Fixing

- **Unambiguous local mismatches: fix now.** Correct a return-code mapping, documented error
  behavior, a comparator mismatch, a forgotten compatibility path, or validation missing at
  an untrusted ingress, when there is a clear authoritative contract. Add an
  integration-level regression test through the real entry point.
- **Contract changes and migrations: confirm first.** Redefining success, changing wire
  meanings, dropping legacy support, accepting narrower input, introducing sorting, or
  altering ownership requires explicit agreement, so show the failing path and the intended
  contract first.
- **Correctness outranks implementation parity, and do not mask broken upstream intent.**
  Matching an existing result is not compliance if that result violates intent, and sorting
  inside every consumer may hide a producer defect and change semantics, so prove which layer
  owns the guarantee.

## Report

| operation / consumer | authority or precondition | implementation or call path assumes | distinguishing case | status |
| --- | --- | --- | --- | --- |
| `submitBatch()` | partial success identified per item | caller treats any 2xx as all accepted | one rejected item | confirmed mismatch |
| `locate()` | sorted by `rank,id` | import → merge → locate, merge uses `rank` only | two equal ranks | precondition violated |

Include the source and scope of each promise, or the first point where a precondition fails,
the end-to-end trace or a reproducible input, and the smallest compliant change. Mark
assumptions as **established**, **violated**, or **unknown**, and separate **confirmed
contract violations**, **conflicting authorities**, and **intentional version differences**.
Never turn "not documented" into "incorrect" without a reachable violation.