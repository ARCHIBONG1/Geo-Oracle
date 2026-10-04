---
name: prospect-target
description: Deriving a well target from the closure's cells under the licence, keep-out and existing-well constraints, the depth and its uncertainty, and what to do when no cell satisfies them. Load before pt_target_point.
---

# Target definition

## `pt_target_point`

Only for a prospect: the tool reads the readiness record and refuses otherwise. The candidate cells are the closed cells (above the scenario's contact if one is given); each constraint is applied in turn and the number of cells it removes is recorded, so a refusal names the constraint that bites.

| Constraint | Table | Rule |
|---|---|---|
| licence | `licence_outline` | inside one of its polygons; without a licence table the target is unconstrained by licence, which the result says and the findings must repeat |
| keep-out | `keepout_zones` | outside each polygon by its `buffer_m` |
| existing wells | `existing_wells` with `min_well_distance_m` | at least that far from each |

The objective is the crest by default (the largest column); `deepest_closed` places the target at the down-dip limit. The depth uncertainty is the larger of the structural pick uncertainty and half the crest's spread across the realizations.

## When no cell satisfies the constraints

The candidate stays a prospect, R6 fails, and the binding constraint is the next request: the licence to its holder, the keep-out to its owner, the well spacing to the operator. A target is never placed by relaxing a constraint silently.

## Coordinates

Every coordinate carries its CRS, taken from the constraint tables; the closure's own CRS is the seismic survey's and must be the same, which the data-QC step checks. A target without a CRS is not a target.

## Reporting

The target as a `measurement`: `<candidate>, <scenario>: target at X, Y (<CRS>), <depth> m TVDSS +/- <uncertainty>, <n> cells satisfy every constraint`. Then `pt_readiness` with the result as `target`, so R6 is recorded and the maturity set by the tool. The `target_card` product carries the trail of constraints.
