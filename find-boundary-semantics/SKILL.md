---
name: find-boundary-semantics
description: Find off-by-one and representation mismatches where two valid local meanings meet: inclusive versus exclusive ranges, units, time bases and clock sources, indices, nullability, encodings, sentinel values, integer conversions. Use when the failure sits at an edge, the first or last item, an empty input, an overflow, the first non-ASCII character, or an interval that comes out negative after a clock step; when integrating APIs, slicing, serialization, protocols, chunking or cross-language data. Fixes the parts that are safely fixable.
license: MIT
---

# Find boundary semantics defects

Many failures occur where two valid local representations encode different meanings. A
count is mistaken for a final index, milliseconds for seconds, UTF-8 bytes for characters,
a missing field for an explicit null, or an inclusive endpoint for an exclusive one.

Name the representations on both sides, then prove the conversion.

## The failure that matters

Most inputs pass. The first, last, empty, overflowing, non-ASCII, or timezone-crossing
case fails because an interface loses a distinction that the downstream code needs.

**Report two incompatible interpretations of the same value, the conversion path between
them, and a failing boundary example.**

## Procedure

1. **Locate representation boundaries.** Array slices, database/API payloads, file formats,
   language FFI, time services, numeric narrowing, and producer, consumer pairs. Select
   boundaries that actually carry relevant data.
2. **Write the contract on both sides.** Specify units, origin, range inclusivity,
   coordinate system, encoding, timezone, signedness, null/missing distinction, sentinel
   meaning, and whether a number is an ID, offset, or count. **For anything time-shaped,
   name the clock and the base separately:** what reads the time, a wall clock or a
   monotonic source, the base it is expressed in (epoch, local date, ticks), its
   resolution, and whether the value is an instant, a duration, or a date. An interval
   taken from a wall clock is the same defect as seconds used as milliseconds, and it
   fails the same way, rarely and far from the defect.
3. **Trace conversions step by step.** Examine rounding, truncation, clamping,
   normalization, casts, defaulting, and parse/serialize behavior. Check conversions in
   helpers as well as direct call sites.
4. **Probe discriminating examples.** Empty range, single element, exact endpoint,
   maximum/minimum, zero, negative values where valid, Unicode multi-byte text, absent
   versus null, and unknown enum values where supported. For a time boundary add: a clock
   step or NTP correction inside an interval, the instant a DST offset changes, a leap
   second where the platform counts one, sub-second truncation, and an expiry or timeout
   that spans the change.
5. **Establish intent before declaring a mismatch.** An inclusive range may be
   intentionally converted to half-open; a sentinel may be valid in only one layer. Show
   where information is actually lost or misinterpreted.
6. **Find the narrowest explicit interface.** Named units, well-defined types, conversion
   helpers, or contract tests can prevent recurrence. Avoid broad conversion rewrites
   without tracing every consumer.

## Fixing

- **Local conversion errors: fix now.** Correct an endpoint adjustment, unit factor,
  overflow check, or null mapping where the public contract is clear. Add table-driven
  tests around the boundary.
- **Representation and protocol changes: confirm first.** Changing serialized fields,
  index bases, timestamps, or cross-service types can break other clients.
- **Keep domain semantics intact.** Do not standardize everything on one representation
  merely because it is convenient.

## Report

| boundary | producer meaning | consumer meaning | witness | consequence |
| --- | --- | --- | --- | --- |
| slice → kernel | `[start,end)` | `[start,end]` | `start=end=3` | extra element |
| interval → wall clock | elapsed from a step-corrected source | a monotonic duration | an NTP step mid-interval | a negative or absurd duration |

Include the exact value before and after conversion, the responsible call path, and
whether the failing case was executed. Mark representation ambiguities without an
observable violation as **contract questions**, not confirmed defects.