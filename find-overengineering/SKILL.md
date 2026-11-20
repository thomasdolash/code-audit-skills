---
name: find-overengineering
description: Find protective code that costs more than the case it protects: a guard no caller can trip, a fallback for a case that cannot happen, an option nothing sets, a compatibility shim or migration for a product with no users yet. Roll the small ones back. Use when reviewing a diff or a subsystem for unnecessary guards, fallbacks, retries, validation, compatibility code or migrations, or when asked whether one is needed. Keeps every check on input the project does not control. Fixes the parts that are safely fixable.
license: MIT
---

# Find overengineering

A fallback for an input no caller can produce, three options where nobody sets two, a
compatibility layer for consumers that no longer exist and a migration framework for a
schema with one user all cost more than the case they defend against. The defense
is not free and not neutral. Every guard is a branch to test and a path to maintain, and a
silent fallback turns a loud failure into a quiet wrong answer.

## The failure that matters

Protective code that handles an edge case of an edge case usually hides the real failure
instead of handling it. The fallback masks a null, the retry masks a bug, and the shim
keeps a dead interface alive forever. It also rots. Nobody exercises it, because the case
it guards has never occurred, so nothing notices when it stops working.

A finding is not "this looks defensive". **For each piece of protection, report the case it handles, who controls that input, whether the case has ever happened, and what the protection costs.** Then recommend a rollback, a narrowing, or a reason to keep it.

**The hard boundary: checks on input the project does not control stay.** A user's file, a network response, a third party's payload and a device are real boundaries, and their validation is the job. This audit is about defense *inside* the trust boundary.

## Procedure

1. **Inventory the protection.** Guards and early returns with no reachable trigger,
   `catch` blocks that swallow, default arguments nobody passes, feature flags and options
   nothing sets, retries and timeouts for operations that can't fail that way,
   compatibility shims, migrations, and abstractions with one implementation.

2. **Name the case each one handles, and who controls that input.** An internal caller, the
   project's own config or its own database is inside the boundary and is a candidate.
   Anything from outside is not.

3. **Test the case's reachability before calling it impossible.** Construct it, grep the
   callers, look for the input that produces it, check the defect history for an incident.
   "Defensive" is not evidence that the case occurs, and neither is your inability to imagine
   it. Name what you checked.

4. **Price it.** Count the lines and branches, the states and options it adds, the tests it
   forces and the surface a reader must hold. Then name what it hides: a swallowed error, a
   wrong default, a failure that travels further before surfacing.

5. **Decide: roll back, narrow, or keep.** Roll back where nothing can reach it. Narrowing
   prefers one check at the boundary over N checks inside, one option over three and a loud
   failure over a quiet fallback. Keep where the case is real, reachable and outside the
   boundary, and say which of those it is.

6. **Confirm before structural rollback.** Removing a compatibility layer, a migration or a
   public option can break data already written or consumers already deployed; the small,
   clearly unreachable ones are yours to remove.

## Fixing

- **Small unreachable rollbacks: do them now.** Delete a guard no caller can trip, a
  fallback for a case that cannot occur, an option nothing reads (see
  `find-accepted-but-ignored` for the ones that *are* read and do nothing), a retry that
  only hides an error, an abstraction with one implementation and no near-term second.
- **Structural rollbacks: confirm first.** A pinned version, a protocol shim, a data
  migration, a compatibility path for deployed consumers, propose with the consumers named
  and let the owner decide.
- **Never roll back a boundary check.** Validation, escaping and authorization on untrusted
  input stay, whatever their cost.
- **Prefer deletion, and prefer narrowing to deleting.** One boundary check beats five
  interior ones; the smallest correct defense is the one that cannot drift.

## Report

Lead with the protection and its price:

| protection | case it handles | controls the input | ever observed | priced cost | disposition |
| --- | --- | --- | --- | --- | --- |
| `defaultConfig` fallback | a caller passing no config | internal | all callers pass config | 2 branches, 1 test | remove |
| retry around a local write | transient IO on the same disk | internal | never | hides the error, 3 s added | remove |
| `--legacy` flag | pre-2.0 consumers | external | none deployed | 1 path, 1 option | confirm first |
| input length check | a hostile payload | external | n/a | 1 guard | keep |

Then per finding: what it protects, why the case does not occur (or does), the price, and
the rollback, or the reason it stays.

Close with: **removed**, **narrowed**, **confirmed with the owner**, and **kept as a real
boundary**, so a reader can see the defense that survived and why.

## Attribution

Adapted from **dzhng/jevgrep** (https://github.com/dzhng/jevgrep), MIT, copyright (c) 2026
David Zhang. The class it audits and the rule that the small cases are rolled back are that
project's; this file states them in its own words and expands them into a procedure. The
repository README carries the full license notice; a copy must stay beside this file when it
is redistributed.
