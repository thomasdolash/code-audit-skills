---
name: find-error-integrity
description: Find failures that never surface: an exception swallowed in a catch, a fault converted to an empty list or a default so the caller sees success, a cause dropped when an error is re-wrapped, an error path that cannot be reached or is never exercised. Use when a failure is mysterious, when something silently does nothing or returns empty, when the logs say success where the outcome was wrong, or when auditing error handling, catch blocks, fallbacks, logging and retries. Fixes the parts that are safely fixable.
license: MIT
---

# Find error integrity

An error-integrity defect is a **failure that reaches nobody**. The handler swallows it,
downgrades it to a benign-looking value, or strips the information someone would need in
order to act on it.

The source of the failure and the shape of the correct fix vary; what they share is that the
system continues as though nothing went wrong.

## The failure that matters

A fault disguised as an ordinary empty result is worse than a crash. The caller proceeds with
wrong data, nothing is logged, and the symptom appears far from the cause, as a missing row, a
silent no-op, or a default that looks legitimate.

**For each failure source, say what the caller sees when it fails and what they would need to
know, or give the reason the degradation is deliberate.** A designed fallback is not a
finding. An accidental one that nobody can distinguish from success is.

## Procedure

1. **Enumerate the failure sources before judging any handler.** Consider throwing calls, I/O,
   parsing, network and external services, allocation, cancellation, invariants, and user
   input.

2. **Follow each failure to its consumer.** Where does it end, a log line, a return value, a
   retry, a default, a user-visible message? Name the fate of every one. A failure whose fate
   you cannot name is the finding.

3. **Look for the disguises:**
   - a catch that logs and continues, or does nothing at all
   - a failure mapped to `null`, `[]`, `false`, `0`, or `-1` the caller reads as data
   - an unknown value collapsed into an enum default
   - a failed call absorbed by a null-coalescing or defaults operator
   - a rejection never awaited, or an error discarded to a placeholder
   - a fallback that erases the difference between "absent" and "failed"
   - re-wrapping that drops the cause, or flattens a typed error to a string
   - a catch so broad it takes programming errors as recoverable data

4. **Separate recoverable from fatal.** A transient network fault and a broken invariant need
   different fates, and collapsing both into data is how the second becomes invisible. Say
   which of the two the handler is treating the failure as.

5. **Check reachability and exercise.** Is the handler reachable, or does an earlier guard
   already guarantee it cannot run? Does any test drive it? An error path no test drives is
   where the surprises are.

## Fixing

- **Fix what is local and unambiguous now.** Propagate a failure that is currently discarded;
  restore the cause when re-wrapping; make a default distinguishable from a failure where the
  caller could act; log at the point of loss with the original error; delete a handler that
  cannot be reached.
- **Confirm first when the contract changes.** Turning a degradation into a throw, altering
  what callers receive, changing retry semantics, or adding a new failure mode to a public
  path all reach outside the function.
- **Preserve behavior unless the behavior is the bug.** A deliberate fallback, a cache miss
  treated as absent, an optional feature unavailable, is correct and stays.

## Report

Lead with the findings, worst first:

| location | can fail | currently reaches | should reach |
| --- | --- | --- | --- |
| `parseConfig()` | malformed value | caller sees `{}` | an error naming the key |
| `saveAll()` | one write fails | loop continues | propagate, with which item |
| `reload()` | request fails | swallowed by empty catch | log with cause, then retry |
| `loadPrefs()` | file absent | default profile | unchanged: intentional |

Then, per significant finding: what can fail, the fate it currently has, what the caller needs
instead, and whether a test drives that path.

Close with four buckets:

- **Fixed**: the failure now surfaces, behavior otherwise unchanged.
- **Proposed**: needs a contract or caller change; most impact first.
- **Deliberate degradation**: intentional and staying. State what distinguishes it from a
  fault, or propose the distinction if nothing does.
- **Not reachable**: the handler cannot run; delete or justify.

**A failure nobody can see is not handled; it is discarded.**