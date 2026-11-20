---
name: find-duplicate-code
description: Find the same rule implemented more than once: a constant, table, formula, validation or protocol written in two places, a copied block that has since drifted, a second implementation of one idea under another name, and name the single place it should live. Use when a fix has to be made twice, when two locations disagree about the same thing, when reviewing copied code, or before extracting a shared helper. Merges only what changes together; leaves lookalikes that change for different reasons. Fixes the parts that are safely fixable.
license: MIT
---

# Find duplicate code

Duplication is the same **rule** written twice, whether that is a formula, a validation, a
constant list, a protocol step, or a derivation. The copies usually differ in their text
because someone adapted them, which is what makes the drift invisible.

The failure is that **the two copies diverge**, and one of them is wrong without anyone
noticing.

## The failure that matters

One rule in two places means every change must be made twice, and the second time is
forgotten. The copy that was not updated does not crash. It returns the old answer, quietly,
until someone notices a discrepancy far from either copy.

**Report the locations, the rule each one implements, whether they already disagree, and the
single place that should own it.**

Merging two things that merely resemble each other costs as much, because one abstraction
must then serve both and every future change to either needs a flag. Duplication is cheaper
than the wrong abstraction.

## Procedure

1. **Identify the rule, not the text.** State what the code computes as a sentence, "the
   frame's half-extent in cells", "which records this user may see", "the retry budget for a
   transient failure". Two blocks sharing that sentence are one rule; two that merely share a
   shape are not.

2. **Look where copies actually come from**, in this order:
   - a block copied between files and then adapted
   - the same constant, table, list or environment name in two places
   - a formula or derivation re-implemented: unit conversion, clamping, hashing, index
     arithmetic, padding
   - the same validation applied on both sides of a boundary
   - a second implementation of one protocol, format or state encoding
   - parallel chains over the same enum or type, two `switch`es that must both grow when
     a case is added
   - the same rule restated in code, in a document, in a config default, or in a test's
     expected value

3. **Compare the copies and report the drift, not the similarity.** Diff them semantically,
   asking what each computes and where they differ. **A meaningful difference between two
   copies of one rule is the strongest finding in this audit**, because one of them is
   already wrong. It is also the cheapest to settle, because both can be run against one
   input and the answers compared.

4. **Ask whether they change together.** Give every pair the same test. Is there a change
   that must be made in both, or a reason one would change alone? The same *rule* changes
   together; the same *shape* does not.

5. **Decide the owner.** The rule belongs where its inputs arrive and where it can be named,
   as a function, a constant, a table in one file, or a generated form of it. Do not create a
   module just to hold it. Name the file and symbol that will own it, and the call sites that
   stop carrying a copy.

6. **Check what duplication is legitimate** before flagging it: generated code, vendored
   third-party source, a language or protocol that mandates a twin (an encoder and its
   decoder, a definition shipped on both sides of a wire), intentionally independent test
   fixtures. Report these as expected, and move on.

## Fixing

- **Local, behavior-preserving consolidation: do it now.** Extract the constant or the
  helper and point both call sites at it; delete the second copy of a table; make one
  validation call the other. Establish that the copies are behaviorally equal first; where
  they already differ, that difference is a decision rather than cleanup.
- **Cross-module or cross-owner merges: confirm first.** Moving a rule into a shared module
  changes dependency direction and routes every future edit through it; show the owner, the
  call sites, and what the abstraction must now serve.
- **Never merge things that change for different reasons.** If a future change could apply
  to one copy and not the other, they are not duplication, and an abstraction over them is
  the defect `find-overengineering` exists to find.
- **A merge that needs a parameter to tell the callers apart is the wrong abstraction.**
  That parameter is the evidence the two cases were separate.

## Report

| rule | copy A | copy B | already differs? | owner |
| --- | --- | --- | --- | --- |
| frame half-extent in cells | `resolve_unit()` | `atom_size()` | yes: rounds differently at the edge | `units.cell_extent()` |
| visible-record predicate | `list_visible()` | `can_open()` | no | `visibility.allows()` |
| retry budget | `send()` | `flush()` | yes: one caps at 3, the other never stops | `retry.budget()` |

Then per finding: the rule as one sentence, where each copy lives, what a divergence would
cost, and the single owner with its call sites.

Close with: **consolidated**, **confirmed with the owner**, **already divergent**: the
strongest finding, saying which copy is wrong and how you know, and **not duplication**, a
legitimate twin named as such.

**Ask of every pair what change would have to be made in both places. If there is none, it is
a shape and not a rule.**