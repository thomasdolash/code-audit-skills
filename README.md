# Code audit skills

Twenty-two skills. Twenty-one audit a codebase for one named class of defect and report a
checkable artifact. `audit-sweep` runs the ones that apply to a target and merges the
findings into one ranked report.

Plain Markdown with YAML frontmatter, written for OpenCode. Any harness that discovers
`skills/<name>/SKILL.md` can load them.

## Install

Copy the directories into your skills path:

- OpenCode: `~/.config/opencode/skills/` for the user, `<project>/.opencode/skills/` for one
  project
- Other harnesses: the equivalent path, or point your loader here

Only `SKILL.md` matters. It is all text, with no scripts or dependencies.

## The shared shape

Every audit has the same four sections:

- **The failure that matters**: what the defect costs.
- **Procedure**: an ordered method that ends in evidence.
- **Fixing**: the safely-fixable findings are fixed, the structural ones confirmed first.
  `find-redundant-checks` differs, because deleting a test is not reversible: it
  quarantines, gets sign-off, then removes with an audit row carrying a restore path.
- **Report**: a table with named columns and buckets.

Two conventions make a report readable:

- A finding names **the break**: the realistic change that turns it red. A claim with no
  nameable break is the finding.
- A finding states its **strength**: measured, derived or unknown.

## The set

| skill | catches |
| --- | --- |
| `audit-sweep` | runs the audits that apply to a target and merges them into one ranked report |
| `find-state-machines` | a lifecycle whose states and exits are scattered; a non-terminal state with no wired exit |
| `find-scaling-costs` | work growing faster than its input; hidden quadratic scans, per-item queries, recomputation |
| `find-boundary-semantics` | off-by-one and representation mismatches at a boundary, including clock and time-base errors |
| `find-concurrency-ordering` | races, stale reads, TOCTOU, lost updates, missing happens-before |
| `find-failure-atomicity` | partial state and duplicate side effects after a failure, cancel, crash or retry, including retry policy |
| `find-invariant-violations` | properties that must hold across every operation, and the paths that break them |
| `find-data-flow-integrity` | data that keeps its type and loses its meaning across stages |
| `find-duplicate-code` | the same rule implemented more than once; copies that have already drifted |
| `find-liveness-defects` | work that can stay pending forever; starvation, livelock, deadlock |
| `find-numerical-instability` | finite-precision failures; cancellation, overflow, unstable reductions |
| `find-contract-inconsistencies` | components that agree on types but not on what an operation promises |
| `find-error-integrity` | failures that never surface; swallowed errors, faults returned as empty results |
| `find-resource-lifecycle` | a handle, lock or listener released on all but one path |
| `find-accepted-but-ignored` | input accepted and never acted on; a key nothing reads |
| `find-nondeterminism` | output that varies between runs although the input did not |
| `find-unbounded-growth` | anything sized by runtime rather than by the work in hand |
| `find-complex-functions` | functions doing more than one thing, or harder to follow than the job requires |
| `find-oversized-units` | files and modules past the size where one change can be understood in one place |
| `find-overengineering` | protective code that costs more than the case it protects; roll it back, keep the boundary checks |
| `find-verification-adequacy` | checks that cannot fail; the slop catalogue and per-case cost ranking |
| `find-redundant-checks` | checks that cost more than they catch; subsumption proved before anything is removed |


## License

MIT, see [`LICENSE`](LICENSE). Third-party notices, with every copyright holder, are in
[`NOTICE.md`](NOTICE.md); the four audits that adapt MIT text also carry their own
`## Attribution` section.
