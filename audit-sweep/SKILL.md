---
name: audit-sweep
description: Run the code-audit set over a target and deliver one ranked report. Use when asked to review a codebase, a subsystem or a diff for defects, when starting work in unfamiliar code, after a large change, or before a release. Picks the audits that apply to the project, splits the work the way the project suggests, and ranks the combined findings worst first by impact. Reports; fixes follow on approval.
license: MIT
---

# Audit sweep

The `find-*` skills each cover one class of defect. This runs the ones that apply and
merges the result into a single report, so the whole set is covered without every skill
reporting separately.

## The set

Twenty-one audits in seven groups. Read the group first: it decides what a project needs.

**Failure and state**

| audit | catches |
| --- | --- |
| `find-error-integrity` | a fault the caller never sees |
| `find-failure-atomicity` | partial state or a doubled effect after a failure |
| `find-resource-lifecycle` | a resource released on every path but one |
| `find-state-machines` | a state with no exit wired |
| `find-liveness-defects` | work that can stay pending forever |

**Concurrency and time**

| audit | catches |
| --- | --- |
| `find-concurrency-ordering` | a race, a stale read, a missing happens-before |
| `find-nondeterminism` | output that varies between runs |

**Data and interfaces**

| audit | catches |
| --- | --- |
| `find-boundary-semantics` | two valid meanings meeting at a boundary |
| `find-contract-inconsistencies` | a promise one layer never keeps |
| `find-data-flow-integrity` | a value that loses its meaning in transit |
| `find-invariant-violations` | a property broken across operations |
| `find-duplicate-code` | one rule written twice, and drifting |
| `find-accepted-but-ignored` | input accepted and never acted on |

**Numbers**

| audit | catches |
| --- | --- |
| `find-numerical-instability` | a result that drifts, a threshold that decides wrongly |

**Growth and cost**

| audit | catches |
| --- | --- |
| `find-scaling-costs` | one path that grows with its input |
| `find-unbounded-growth` | a store that grows with runtime |

**Code shape**

| audit | catches |
| --- | --- |
| `find-complex-functions` | a function doing too much |
| `find-oversized-units` | a file or module doing too much |
| `find-overengineering` | protection that costs more than the case it guards |

**Checks**

| audit | catches |
| --- | --- |
| `find-verification-adequacy` | a check that cannot fail |
| `find-redundant-checks` | a check that costs more than it catches |

## The project decides the shape

Read the target before choosing anything. What it is determines how the sweep runs:

- **Which audits apply.** No tests means no audit from **Checks**. No concurrency means no
  audit from **Concurrency and time**. No persistence means no atomicity audit. Say which
  you dropped, and why, one line each.
- **How the work splits.** By module, by audit, by subsystem, whatever the tree and the
  request suggest. One pass or many. Serial or parallel. A small tree does not need lanes.
- **How much is read.** A diff review reads the diff and what it reaches; a whole-repo
  review reads the tree.

Do not impose a roster, a lane count or a phase order. State your choice in a line.

## Consolidate

- **One owner per finding.** The audits overlap. When two catch the same defect, the
  narrower remedy owns it.

  | defect | owner |
  | --- | --- |
  | a failure nobody can see | `find-error-integrity` |
  | partial state after a failure | `find-failure-atomicity` |
  | a value that loses its meaning in transit | `find-data-flow-integrity` |
  | one rule written twice | `find-duplicate-code` |
  | two valid meanings at one boundary | `find-boundary-semantics` |
  | a promise one layer never keeps | `find-contract-inconsistencies` |
  | no way out of a state | `find-state-machines` |
  | work that never completes | `find-liveness-defects` |
  | growth per call | `find-scaling-costs` |
  | growth per runtime | `find-unbounded-growth` |
  | a check that cannot fail | `find-verification-adequacy` |
  | a check that costs more than it catches | `find-redundant-checks` |

- **Rank worst first:** correctness and data loss, then liveness, then accounting, then
  cost. Say why that order holds for this project.
- **Each finding:** where it is, what breaks, how you know, and at what strength. A finding
  with no nameable break is itself the finding.
- **Close with the gaps:** what you did not read, and why.

## Fix, after

Report first. Fix on approval, worst first, one editor per file. Do not edit during the
sweep.
