---
name: prospect-data-qc
description: How to read upstream products (closures, play matrices, ledgers and companion tables), register the supplied constraint and property tables, and turn upstream claims into inputs with provenance. Load before reading any product or registering any table.
---

# Prospect data QC

## Products

| Product | Owner | Read for | Watch |
|---|---|---|---|
| `trap_geometry` | structural_geology | domain (depth or time), closed area, crest, spill, relief, the validity record V1-V7, the realization count and ranges, the cell table, the realised cells (`realization_cells`) | a closure in time is unusable; V1 and V7 untested means R2 is untested, not failed: ask structural_geology for a trap_geometry from sg_closure_spill with model=<the structural_model on the same horizon>, so V1 and V7 are in its validity record |
| `element_status_matrix`, `play_concepts` | subsurface_play | the element statuses to inherit, the weakest element, the play's own status | the candidate must sit in the play named in the matrix |
| `fault_seal`, `fault_stability` | structural_geology | the column the seal holds; the critical pressure | both rest on bounded stress unless stress was measured: inferred at best |
| `stratigraphic_framework`, `zonation` | stratigraphy | whether the reservoir is present at the closure, the seal's continuity | the C1-C7 ledger |
| `depositional_model`, `facies_log` | sedimentology | reservoir and seal facies, the rung reached | an environment at the association rung is not a reservoir description |
| `pressure_profile`, `temperature_profile` | wells_petrophysics | conditions at the contact | |

`pt_read_product` returns the sidecar. If a product's ledger fails, use the product with the failure stated. Lower the element that rests on it, and never raise it.

## The closure's companion table

A closure names its cell table in the sidecar (`closure_cells`). Read the table through the producing agent's namespace, `@structural-geology-agent/tables/<file>`. The table is what makes a gross rock volume possible. Without it, the tool refuses, and the request is for the closure republished with its cells.

## Supplied tables (`pt_register_table`)

The columns before the semicolon are required; the others are optional.

| Kind | Columns | Notes |
|---|---|---|
| `zone_summary` | zone; well, net_to_gross, porosity, sw, and their p10/p90 where given | the source of the volume inputs; quote each as `{product: <this table>, field: <column>, value: ...}` |
| `pvt` | fluid; pressure_mpa, temperature_c, bo or bg, density_kg_m3, source, sample | at reservoir conditions; a formation volume factor at other conditions is the wrong number |
| `licence_outline` | name, x, y; vertex (the point order) | one row per vertex, at least three per polygon; the `crs` argument is required, because a coordinate without a CRS cannot be checked |
| `keepout_zones` | name, x, y; vertex, buffer_m, reason | as the licence; `buffer_m` is the distance the target keeps outside the polygon (0 when absent) |
| `existing_wells` | well, x, y; td_tvdss_m, status | for the minimum-distance check; the `crs` argument is required |

## Claims as inputs

An upstream finding becomes an input as `{range: [p10, p50, p90], source: '<its full id>'}`. Never give it as a bare number, and never upgrade its classification or status. If a finding gives one value and no range, say so: a single value is a p50 with no spread, and that is a limitation worth reporting.

<!-- end of skill: prospect-data-qc -->
