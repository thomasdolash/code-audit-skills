---
name: find-accepted-but-ignored
description: Find input that is accepted and then has no effect: a config key absent from the list that validates it or the code that reads it, a flag nothing consults, a parameter a function receives and ignores, an unknown field dropped in silence, an option documented but unimplemented. Use when a setting seems to do nothing, when a "supported" option changes nothing, after adding or renaming a config key, or when auditing configuration parsing, API parameters, feature flags and option objects. Fixes the parts that are safely fixable.
license: MIT
---

# Find accepted but ignored

An accepted-but-ignored defect is input the system **takes and never acts on**, so the caller
believes it took effect.

The value parses, validates, and is then unused, or is validated against a set that no longer
matches what the code downstream will accept.

## The failure that matters

A value that is silently inert is worse than one that is rejected. The user tunes the knob,
believes it, and the behavior is unchanged; no layer says so, and no test can fail because
nothing was expected to happen. The same shape appears at every boundary, whether config,
flags, query parameters, options objects, or the fields of a parsed document.

**For each input that carries intent, name the place it should take effect and whether it
does, or the reason it is deliberately inert.** A reserved key awaiting an implementation is
fine if it is marked as one.

## Procedure

1. **List the inputs that carry intent.** Configuration keys and environment variables,
   command-line flags, request parameters and bodies, option objects, feature flags, headers,
   and the fields of any parsed document or manifest.

2. **Trace each input from where it is accepted to where it takes effect.** Name the
   consumer, the branch, call, or setting it reaches. An input with no consumer past its
   parsing site is the finding.

3. **Look for the shapes:**
   - parsed, stored, and never read; or read into a default the code then ignores
   - validated against a set that does not include the value actually in use, so the
     validator accepts what the consumer will silently discard
   - a documented option absent from the implementation entirely
   - the same setting written from two sources, with one silently winning
   - a field dropped by a parser that strips unknown fields without reporting them
   - a flag read once at startup while the documentation implies it is live
   - a parameter accepted and then shadowed by a local or a constant

4. **Check the accepted set against the real domain.** A validator built from a stale list is
   the same defect wearing a check, because it accepts a value the downstream model, provider,
   or schema will refuse, and the refusal is swallowed.

5. **Say which inert inputs matter.** One that is documented and user-settable is a defect.
   One that is internal, unreachable, or explicitly reserved is not, but it should say so in
   place, so the next reader does not repeat the audit.

## Fixing

- **Small and unambiguous: fix now.** Remove a key nothing reads; report or reject a field the
  parser drops; wire a parsed value to the branch that clearly meant to use it; align a
  validator's set with the consumer's real domain.
- **Confirm first when behavior changes for existing input.** Making a previously accepted
  value an error, or giving effect to a setting people already set, changes runtime behavior
  for every existing configuration.
- **Preserve deliberate inertness.** Do not delete a reserved key someone may set; mark it as
  inert instead, in the same place it is declared.

## Report

Lead with the findings:

| input | accepted at | should affect | effect now |
| --- | --- | --- | --- |
| engine variant | config load | the engine call | none: the engine declares no such variant |
| retry count | request options | the retry loop | ignored: the loop reads a constant |
| unknown fields | body parse | nothing | stripped silently, no report |
| timeout | constructor | request deadline | honored |

Then, per finding: the input, where it is accepted, where it should take effect, and what the
user would observe either way. Where a value is validated, say what the validator checks
against and whether that source is current.

Close with four buckets:

- **Fixed**: the input now takes effect, or now says it cannot.
- **Proposed**: needs a behavior decision for existing configurations.
- **Deliberately inert**: reserved or internal, stated as such in place.
- **Documented but unimplemented**: the documentation promises what the code does not do.

**If a value can be set, it either does something or it says it cannot.**