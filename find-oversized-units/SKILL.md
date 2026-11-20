---
name: find-oversized-units
description: Find files and modules that have grown past the size where one change can be understood in one place: an implementation file far over the preferred range, a module several agents edit in unrelated sections, a unit whose parts change for different reasons. Use when asked about file or module size, when a change means reading far more code than it should, when review or merge conflicts cluster in one file, or before adding a feature to an already oversized file. Recommends named boundaries, extracts as it goes, and confirms a wider split before making it.
license: MIT
---

# Find oversized units

One change should be understandable in one place. A unit that breaks that rule holds more
code than the change needs to touch: a file past the preferred range, a module where every
agent edits a different section, or a unit whose parts change for unrelated reasons.

Size alone is not the defect. A cohesive 550-line module can beat eight 70-line modules you
must open together. The defect is responsibility concentration. The unit is large enough
that changing one behavior requires understanding several unrelated ones.

Find the concentration, name the boundaries, and say what a split would and would not buy.

## The failure that matters

Large units cost more than readability. They cost the context an agent must consume to work
in them, overlapping edits between parallel worktrees, review difficulty, and the slow
accumulation of unrelated responsibilities in one module.

A finding is not "file too long". **For each finding, name the responsibilities present, whether they separate cleanly, and the boundaries you would draw, or give an explicit reason the size is right.** A big declarative table, a generated file, and a vendored dependency are not findings. A coherent 900-line operator registry is not the same as a 900-line file that initializes a device, allocates resources, compiles shaders, and schedules work.

## Procedure

1. **Inventory the files mechanically before judging any of them.** For each: path,
   physical lines, substantive lines (excluding blanks and comments), approximate
   responsibility, and recent growth. Rank by substantive lines.

   Thresholds, for hand-maintained implementation code only:

   | file size | reading |
   | --- | --- |
   | under 150 | small: fine if it is a meaningful unit |
   | 150 to 350 | preferred range |
   | 350 to 500 | acceptable: flag only mixed responsibilities |
   | 500 to 800 | review: say whether a split would help |
   | 800 to 1200 | refactoring expected unless there is a clear exception |
   | over 1200 | finding: needs a justification or a concrete split |
   | over 2000 | architectural: check whether subsystems accumulated |

   None of these apply to generated code, vendored dependencies, large declarative
   tables, or test fixtures. Exclude tests and data from the count, or say which you
   counted.

2. **Apply the boundary test: one reason to change.** Ask who would ask for this unit to
   change. If two different requesters drive it (distinct actors, subsystems or external
   contracts), that is where the boundary is; the line count is only what made you look.
   If everything in the file changes together for one reason, it is cohesive however long
   it is; say so and stop.

3. **Judge each flagged unit architecturally.** For every file over the review threshold:
   - does it hold more than one clearly separable responsibility?
   - can those parts be named, and given real module boundaries?
   - would extraction reduce coupling, or move code behind more indirection?
   - are several people or agents editing unrelated sections of it?
   - can a split preserve the current API, ownership, and behavior?
   - is it large because of data or test declarations rather than control flow?

4. **Recommend boundaries, not a number.** Name the modules you would extract and what
   each would own. "Split this file" is not a finding. Where the whole unit is
   interdependent, say so and stop.

5. **Check change concentration where history is available.** A 600-line file touched by
   many independent tasks in a week is usually more urgent than a stable 1500-line file of
   declarative metadata. Report high-churn oversized files separately, because that is
   where a split pays first.

6. **Distinguish new growth from inherited.** Say which size this change introduced and
   which was already there. Do not penalise a file you did not grow, and do not let a task
   worsen one either: a feature added to an already oversized file should extract the
   responsibility it needs where that is practical.

   Function length, nesting, and branch count are a different audit, `find-complex-functions`.

## Fixing

- **Extract as you go when the task would otherwise worsen a file.** A feature added to an
  already oversized file takes its responsibility out with it where that is practical, so
  the file does not grow, and the extraction is part of the work rather than a follow-up.
- **An audit's splits are structural, so they are confirmed first.** Every finding here
  moves code across module, API, or ownership lines; that is what separates this from the
  audits whose findings are local. Name the boundaries, get agreement, then split, and do
  not leave the review half-applied by starting one.
- **Internal modules first.** Move code without changing the API, ownership, or behavior.
  Create a separate crate or package only when dependency direction, reuse, or ownership
  isolation justifies it; a large file is not a reason.
- **Never split to hit a target.** Cohesion decides. If the parts cannot be named and do
  not separate without indirection, the finding is "large but coherent", with the reason.

## Report

Lead with the inventory, largest first:

| file | substantive lines | responsibilities | assessment |
| --- | --- | --- | --- |
| `src/store` | 7,400 | indexing, query planning, persistence, migrations | split: four separable concerns |
| `src/ui/workspace` | 2,900 | layout, canvas, inspector, assets | split: boundaries below |
| `src/rules` | 1,800 | declarations, registration, verification | partly coherent; registration separates |
| `src/config` | 620 | one declarative table, twelve lines of logic | no action: data, not control flow |

Then, per finding: the responsibilities present, the boundaries you would draw, what the
split buys, and what it costs: indirection, moved files, API reach.

Close with four buckets:

- **Split recommended**: named boundaries, and whether the APIs survive.
- **Large but coherent**: over a threshold and staying whole, with the reason.
- **Inherited, not worsened**: already oversized; this change did not grow it.
- **Not a finding**: generated, vendored, declarative, or test data.

**The measure is not the number of files.** Ask instead whether this change reduces the
amount of code someone must understand to change one behavior.
