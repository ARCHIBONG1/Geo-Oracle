---
name: prospect-data-qc
description: Reading upstream products (closures, play matrices, ledgers and companion tables), registering the supplied constraint and property tables, and turning upstream claims into inputs with provenance. Load before reading any product or registering any table.
---

# Prospect data QC

## Products

| Product | Owner | Read for | Watch |
|---|---|---|---|
| `trap_geometry` | structural_geology | domain (depth or time), closed area, crest, spill, relief, the validity record V1-V7, the realization count and ranges, the cell table | a closure in time is unusable; V1 and V7 untested means R2 is untested, not failed |
| `element_status_matrix`, `play_concepts` | subsurface_play | the element statuses to inherit, the weakest element, the play's own status | the candidate must sit in the play named in the matrix |
| `fault_seal`, `fault_stability` | structural_geology | the column the seal holds; the critical pressure | both rest on bounded stress unless stress was measured: inferred at best |
| `stratigraphic_framework`, `zonation` | stratigraphy | whether the reservoir is present at the closure, the seal's continuity | the C1-C7 ledger |
| `depositional_model`, `facies_log` | sedimentology | reservoir and seal facies, the rung reached | an environment at the association rung is not a reservoir description |
| `pressure_profile`, `temperature_profile` | wells_petrophysics | conditions at the contact | |

`pt_read_product` returns the sidecar. A product whose ledger fails is used with the failure stated, and the element that rests on it is lowered, never raised.

## The closure's companion table

A closure names its cell table in the sidecar (`closure_cells`). It is read through the producing agent's namespace, `@structural-geology-agent/tables/<file>`, and it is what makes a gross rock volume possible: without it the tool refuses, and the request is for the closure republished with its cells.

## Supplied tables (`pt_register_table`)

| Kind | Columns | Notes |
|---|---|---|
| `zone_summary` | zone, well, net_to_gross, porosity, sw, and their p10/p90 where given | the source of the volume inputs; quote each as `{product: <this table>, field: <column>, value: ...}` |
| `pvt` | fluid, pressure_mpa, temperature_c, bo or bg, density | at reservoir conditions; a formation volume factor at other conditions is the wrong number |
| `licence_outline`, `keepout_zones` | name, x, y, buffer_m | a CRS is required; a coordinate without one cannot be checked |
| `existing_wells` | well, x, y, td_tvdss_m | for the minimum-distance check |

## Claims as inputs

An upstream finding becomes an input as `{range: [p10, p50, p90], source: '<its full id>'}`, never as a bare number, and never with its classification or status upgraded. Where a finding gives one value and no range, say so: a single value is a p50 with no spread, and that is a limitation worth reporting.
