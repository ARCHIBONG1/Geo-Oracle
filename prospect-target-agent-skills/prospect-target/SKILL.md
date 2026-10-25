---
name: prospect-target
description: How to derive a well target from the closure's cells under the licence, keep-out and existing-well constraints; the depth and its uncertainty; and what to do when no cell satisfies them. Load before pt_target_point.
---

# Target definition

## `pt_target_point`

Only for a prospect or a target: the tool reads the readiness record and refuses a lead or a concept. The candidate cells are the closed cells (above the scenario's contact if one is given). The tool applies each constraint in turn, and records the number of cells it removes. If no cell satisfies them, the tool returns `found: false` with the binding constraint, and R6 fails.

| Constraint | Table | Rule |
|---|---|---|
| licence | `licence_outline` | inside one of its polygons; without a licence table the target is unconstrained by licence, which the result says and the findings must repeat |
| keep-out | `keepout_zones` | outside each polygon by its `buffer_m` |
| existing wells | `existing_wells` with `min_well_distance_m` | at least that far from each |

The objective is the crest by default (the largest column). `max_column` chooses the same cell as `crest`, because the column is largest at the crest. `deepest_closed` places the target at the down-dip limit. The depth uncertainty is the larger of the structural pick uncertainty and half the crest's spread across the realizations.

## When no cell satisfies the constraints

The candidate stays a prospect, and R6 fails. The binding constraint is the next request: the licence to its holder, the keep-out to its owner, the well spacing to the operator. Never place a target by relaxing a constraint silently.

## Coordinates

Every coordinate carries its CRS, taken from the constraint tables. The closure's own CRS is the seismic survey's, and it must be the same. The data-QC step checks this. A target without a CRS is not a target.

## Reporting

State the target as a `measurement`: `<candidate>, <scenario>: target at X, Y (<CRS>), <depth> m TVDSS +/- <uncertainty>, <n> cells satisfy every constraint`. Then run `pt_readiness` with the result as `target`, so that R6 is recorded and the tool sets the maturity. The `target_card` product carries the trail of constraints.

<!-- end of skill: prospect-target -->
