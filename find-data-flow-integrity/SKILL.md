---
name: find-data-flow-integrity
description: Find data that keeps its type but loses its meaning as it moves: a position mistaken for an ID, a filtered index reused against the original list, two records aliasing one mutable object, a batch that drops its remainder. Use when a value is right where it is produced and wrong where it is used, when a result lands on the wrong record or goes missing, or when auditing planners, ETL pipelines, slices, batching, index maps, serialization or multi-stage processing. Fixes the parts that are safely fixable.
license: MIT
---

# Find data-flow integrity defects

Data may retain the same type and look valid while its meaning changes between stages. A
position becomes an ID, a filtered index is reused against the original list, two records
alias one mutable object, or a batch omits a remainder.

Track what every value denotes as well as what it contains.

## The failure that matters

The output passes schema checks but is attached to the wrong entity, misses records,
duplicates work, or attributes results to the wrong source. Individual transformations
appear correct because the defect is in the handoff.

**Report the complete lineage to the consumer that receives the wrong value and the point
where the semantic guarantee is lost.**

## Procedure

1. **Choose one data product and define its identity domain.** Distinguish stable IDs,
   array offsets, view-local indices, handles, keys, and positional order. State whether
   duplicates and reordering are meaningful.
2. **Draw the transformation chain.** Trace producers, maps, filters, sorts, grouping,
   slicing, serialization, transfers, caches, and final consumers. Record the shape and
   semantic domain at each boundary.
3. **Check preservation rules at every edge.** Which operations preserve membership,
   order, identity, multiplicity, source attribution, and ownership? Which intentionally
   change them? Verify explicit remapping when indices or handles change domain.
4. **Examine aliasing and lifetime.** Shared buffers, mutable views, reused objects,
   in-place operations, and async consumers can make an earlier value change after it was
   validated. Confirm copy versus view semantics from actual APIs.
5. **Follow exceptional and partial paths.** Failed batches, skipped items, empty
   partitions, retries, deduplication collisions, and out-of-order results often expose a
   missing mapping or a cardinality mismatch.
6. **Demonstrate the mismatch.** Use a tiny input with distinguishable records; annotate
   each stage and show the first incorrect value or association. If transformations are
   intentionally lossy, compare against the documented loss rule rather than assuming a
   bijection.

## Fixing

- **Local identity/remapping errors: fix now.** Carry the correct stable key, repair an
  index map, preserve provenance metadata, or handle an omitted tail where semantics are
  clear. Add a multi-stage regression test.
- **Structural changes: confirm first.** Replacing positional references with IDs,
  changing buffer ownership, or restructuring the pipeline affects every downstream
  consumer. Propose the contract for each edge first.
- **Preserve permissible transformations.** Do not accidentally forbid legitimate sorting,
  aggregation, deduplication, or lossy conversion.

## Report

| source → consumer | broken guarantee | first bad edge | witness | impact |
| --- | --- | --- | --- | --- |
| inputs → slices → results | stable item identity | filtered offset reused as source ID | IDs A,B,C with B removed | result misattribution |

Include a short lineage trace, the expected mapping, the observed or derived mapping, and
whether the defect was confirmed by execution. Keep identity defects distinct from general
API-contract disagreements and cost-only concerns.