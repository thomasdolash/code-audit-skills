---
name: find-state-machines
description: Find implicit state machines: one lifecycle whose state, transitions, and ownership are scattered across flags, effects, callbacks, and async work, and make them explicit, guaranteeing every non-terminal state has a way out. Use when a spinner never stops, a record sits in processing, a flag is never cleared, or an object ends up in a state nothing exits; when auditing lifecycle coordination, impossible states, or a proposed stateful refactor.
license: MIT
---

# Find state machines

A lifecycle that exists only as scattered flags is one machine nobody can name. A
connection, a job, a session, an upload or an editor moves through states that live in
three files: a flag in one, a callback in another, an `isLoading` that never clears. The
states are real, and no single place describes them.

Make the machine explicit, and give every state a way out.

## The failure that matters

A state with no guaranteed exit is the defect. A failure branch returns without clearing
the flag, a cancel path skips its cleanup, a timeout was never written, or a process dies
mid-flight and nothing resumes the work. The happy path exists, which is why the defect
survives. A state you cannot leave becomes a hang, a leak or a corrupt record.

**Name the exit for every non-terminal state.**

## Procedure

1. **Name the lifecycle and list its states** before reading for defects. Write "idle,
   connecting, live, draining, closed" rather than "the loading stuff".
2. **Say where each state lives**: a field, a flag, a row, an effect dependency, a closure.
   Two copies of one state will diverge, so the scattering is itself a finding.
3. **List every transition and what triggers it.** Look for transitions that exist in the
   code and in no mental model, and for states that are reachable and never named.
4. **For each non-terminal state, name the event that leaves it:**
   - success
   - failure
   - cancel
   - timeout
   - restart, or recovery on the next load

   One missing exit is a finding. All of them missing is a defect, not a risk.
5. **Check that each exit is wired, not merely present.** A `catch` that logs and returns
   without clearing the state is not an exit. Follow it to where the state changes.
6. **Look for impossible states**: two flags that must never both be true, a "closed" that
   still receives events. Prefer one value that cannot represent the impossible combination
   over a guard on every read.

## Fixing

- **Small behavior-preserving fixes: do them now.** Add the missing timeout, clear the flag
  in the failure branch, cancel the subscription on teardown, resume after a restart. These
  are defects, and they are fixed where they are.
- **Restructures: confirm first.** Collapsing scattered flags into one state, or moving
  ownership, changes behavior and touches call sites. Show the states with their exits and
  agree the change before rewriting.
- **Preserve behavior unless the behavior is the defect.** Do not clean up a lifecycle you
  were not asked to change.

## Report

Lead with the states and their exits, as a table:

| state | lives in | exits wired |
| --- | --- | --- |
| connecting | `_socket`, `uploadState` flag | success yes, failure yes, cancel no, timeout no, restart no |

Then state what you fixed, and what you would restructure with the reason.

## Attribution

Adapted from **dzhng/jevgrep** (https://github.com/dzhng/jevgrep), MIT, copyright (c) 2026
David Zhang. The description and the central requirement are that project's text; this file
restructures them. The repository README carries the full license notice; a copy must stay
beside this file when it is redistributed.
