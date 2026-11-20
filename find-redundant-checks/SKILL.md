---
name: find-redundant-checks
description: Find checks that cost more than they catch: cases subsumed by another, near-duplicates under different names, cases whose claim is already guarded at a cheaper level, tests of a feature that no longer exists, and setup patterns that run everywhere. Use when a suite takes too long for what it proves, when asked which tests can go, when a level duplicates another, when the same claim is checked twice, or before deleting, demoting or rewriting cases. Coverage equality is not redundancy; nothing is removed without a subsumption proof and a restore path.
license: MIT
---

# Find redundant checks

A check can spend time without buying fault detection. It may guard the same claim twice,
guard a claim two levels above where it would be caught just as well, test code that no
longer exists, or belong to a suite where every case pays for the same setup.

> **Coverage equality is not redundancy.** Two cases can execute the same code and assert
> completely different things: a different oracle, different inputs, a different edge.
> Coverage says what ran, never what was checked. The only proof that one case subsumes
> another is that the survivor catches the same faults.

## The failure that matters

Deleting the wrong case is irreversible in practice, because "it is in version history" is
not a recovery plan; nobody watches for the defect a deleted case would have caught. The
mirror failure is as common. A suite is declared redundant and cut to a fast set that no
longer catches anything.

A finding is not "these look duplicated". **For each case, report the claim it makes, the break that reddens it, and where that claim stays covered afterwards, or say explicitly to keep it.** A case that has never failed is not thereby worthless. It usually guards low-churn, stable code. Investigate churn and risk, and never delete on the strength of a green record.

## Procedure

1. **Measure the corpus.** Take per-case cost from the runner's own machine-readable report,
   and hand the command back rather than running it, per the standing rule. Attribute a
   shared process's setup to the cases that pay it, rank by seconds saved, and work from the
   top; the ranking is the worklist, and "looks slow" is never the measure.

2. **Map the claims**, one row per case file: level (unit / integration / request / system /
   end-to-end), the production code it exercises, each distinct claim on its own line, the
   real libraries it drives, the doubles it uses, its measured cost, and the proposed move.

3. **Name the break** for each claim, the realistic production change that reddens it. No
   nameable break is what makes a case a candidate. Spend cases where the project actually
   goes wrong: complicated conditionals, money, security, code with a defect history.

4. **Classify, one disposition per case, never one bucket:**

   | disposition | means | move |
   | --- | --- | --- |
   | **Redundant** | subsumed, same claim AND the survivor catches the same breaks | merge or delete, but only after a mutation check confirms subsumption |
   | **Obsolete** | tests a feature, path or flag that no longer exists | delete, after verifying the target is really gone |
   | **Low-value** | never failed AND trivial, no meaningful assertion | quarantine and review; low value is not zero value |
   | **Keep** | uniquely covers something, has caught a real defect, or guards a high-risk path | keep, whatever the overlap |

5. **Cluster near-duplicates by similarity, not filenames.** Hash the files and diff
   same-named pairs first, then compare normalized structure (token or AST shingling) **and**
   the measured cost profile. A cluster is a set of candidates for review, never a delete.

6. **Check the level.** A claim belongs at the cheapest level that keeps its failure mode.
   In-memory logic belongs in unit cases; a high-level case keeps only what nothing lower
   can catch, such as wiring, configuration or one happy path per critical flow. Where a
   high-level case and a lower one check the same claim, the expensive one is the candidate.
   Assert at a public boundary, so a behavior-preserving refactor does not redden it; where
   the project hands work to a library, the claim is the data handed over, with one smoke
   case that the library accepts it.

7. **Tier on risk and defect history, runtime last.** The fast tier is the critical-path
   guards plus the cases that have caught real defects; runtime only breaks ties. Encode
   tiers as a selection, a profile, a marker, a tag, never by moving files, and account
   for every case in some tier.

## Fixing

- **The default verdict is lean.** A few slow cases are not a problem; do not invent cuts. If
  the suite is already lean for its risk, say so and stop.
- **Apply the first move that fits.** **Delete** (no nameable break, or the same claim at the
  same or a cheaper level), **Demote** (a real claim above its cheapest level, write it
  lower and drop the expensive check), **Merge** (near-duplicates sharing arrange/act become
  one parameterised case), **Rewrite** (a real claim in a fragile or over-mocked case),
  **Fix flake**, **Keep**.
- **Guardrails.** Never remove the only guard of a claim the project owns: every removal
  names where its claim stays covered, or why the claim is not ours. A keep-list stands
  unless the owner says otherwise: regression cases tied to a real defect, contract cases at
  a boundary, error text or format consumers rely on, property and invariant cases, and
  anything on money, auth or permissions. Speed never buys a weaker claim; making a case
  fast by removing what it proves is a deletion and needs a deletion's evidence. No new
  tooling (a mutation runner, a per-test coverage tool) without asking.
- **Order the work:** levers that change no case first (configuration, environment,
  parallelism), then demotions and rewrites, replacements written and passing FIRST, then
  deletions, each landing on a suite where its replacement already passes, then tiers last,
  so a tier holds only cases that survived the cuts. One commit per batch; a red batch is
  reverted and diagnosed from the reverted state, then retried smaller.
- **Deletion is destructive, so it has a ladder.** Quarantine first: skip the case with the
  reason and an audit-row id, or mark it deprecated, so it stops running but stays
  restorable. Then the owner approves the cohort; an agent never deletes tests on its own
  reading. Then an observation window runs with the cohort quarantined and watches for the
  defect it would have caught; an escaped defect means it was not redundant, and the case is
  restored. Only then delete in small batches, each linked to its audit row.
- **The audit record is committed**, one row per removal: case path, disposition and reason,
  the surviving case that supersedes it, the measured cost saved, the approver with revision
  and date, and the one-line restore path. **A row that cannot be filled is a case that
  cannot be deleted.**

## Report

Lead with the corpus, ranked by seconds saved:

| case | cost | claim | disposition | where it stays covered |
| --- | --- | --- | --- | --- |
| `device_suite` (14 cases) | 9 to 14 s each | data handed to the driver | demote | same claims without a device |
| `test_parse_b` | 0.2 s | parser rejects a bad token | redundant | `test_parse_a`, same oracle |
| `test_empty` | 0.1 s | empty input yields empty | keep | only case for it |
| `test_totals` | 0.1 s | the sum is correct | keep | has caught a real defect |

Then, per finding: the claim, the break it would catch, the survivor that catches the same
break, and the seconds saved.

Close with four buckets:

- **Removed**: with its audit row and restore path.
- **Quarantined**: excluded, still collected, awaiting the observation window.
- **Proposed**: a batch awaiting the owner's decision.
- **Keep**: stated so the reading is visible; large or slow but coherent is a finding that
  is not a cut.

**Before anything is removed:** every redundant row carries a mutation result, or it stays
a quarantine with an open row and the report says so rather than claiming a deletion; nothing
was removed in the quarantine pass; the tier selection returns only its cohort and every case
belongs to one; and the log has no blank `superseded_by`, cost or approval field.

**Cut a suite by proving subsumption, never by reading coverage.**

## Attribution

Adapted from two MIT projects:

- **petrkindlmann/qa-skills** (https://github.com/petrkindlmann/qa-skills), copyright (c) 2026
  Petr Kindlmann, the corpus-level audit, the coverage-equality rule and the
  destructive-safety ladder.
- **svyatov/supermatt** (https://github.com/svyatov/supermatt), copyright (c) 2026 Matt
  Pocock and Leonid Svyatov, the move set and the guardrails.

The repository README carries the full license notice, naming every holder; a copy must stay
beside this file when it is redistributed.
