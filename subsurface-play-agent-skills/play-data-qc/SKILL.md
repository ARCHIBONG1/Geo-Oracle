---
name: play-data-qc
description: How to read upstream products and claims (their ledgers, scope and transfer). How to register well outcomes, shows, seeps, geochemistry, tests and legacy wells, and turn upstream findings into evidence links. Load before reading any product or linking any element.
---

# Play data QC

## Claims are the main input

Geo Oracle's `upstream_findings` carry the id, classification, status and scope of every finding that bears on an element. Build a link from them directly: `{evidence: '<id>', scope: 'study-area' | 'regional' | 'analogue', kind: '<classification>', status: '<status>', against: false}`. The scope comes from the `[regional]` and `[analogue]` tags and from the regional products' transfer records. A finding from a study-area specialist without a scope tag is study-area. Write the scope on every link: a link without one counts as regional. Never upgrade a finding's classification (an interpretation is not an observation) or its status.

## Products

| Product | Owner | Read for | Link as |
|---|---|---|---|
| `trap_geometry` | structural_geology | closure, spill, relief, V1-V7, realizations | trap (interpretation; inferred at best unless V1 and V7 pass and the closure holds in every realization). V1 and V7 stay "not tested" unless `sg_closure_spill` ran with `model`, the structural_model on the same horizon: if they are not tested, ask structural_geology for that trap_geometry |
| `fault_seal`, `fault_stability` | structural_geology | SGR, column, critical pressure, verdict | seal or containment (interpretation; the stress is bounded, so inferred at best) |
| `stratigraphic_framework`, `correlated_tops`, `isopach_trends` | stratigraphy | seal continuity, C1-C7 | seal, caprock (interpretation) |
| `depositional_model`, `facies_log` | sedimentology | reservoir and seal facies, S1-S7, rung | reservoir, storage unit (observation for the described facies; interpretation for the environment) |
| `burial_history`, `thermal_model`, `regional_framework` | regional_geology | burial, heat flow, phases, transfer | maturation, preservation (regional scope unless calibrated locally) |
| `pressure_profile`, `temperature_profile` | wells_petrophysics | pressures, temperatures | seal (pressure differences), heat |

`play_read_product` returns the sidecar's ledger and scope. If a product's ledger fails, link it with status partially_supported at most, and put the failure in the note.

## Tables (`play_register_table`)

| Kind | Columns | Notes |
|---|---|---|
| `well_outcomes` | well, result (discovery, dry, shows, tight, not reached); year, target, stated_failure_reason, x, y, crs | The stated reason is the operator's view; the classification is yours |
| `shows`, `seeps`, `test_summaries` | see the tool | Direct study-area evidence for charge and migration |
| `source_rock_geochem`, `maturity_data`, `fluid_geochem` | TOC, Rock-Eval, Ro, Tmax, compositions | Study-area evidence for source and maturation (no tool computes from them yet; use the values as links) |
| `legacy_wells` | well, status, penetrates_caprock, integrity | Well containment: a well without an integrity record leaves it unknown |

## Reporting

Give tables as `[data, project_data]`, and products as `[data, upstream_specialist]` by reference. Cite claims by their full id.

<!-- end of skill: play-data-qc -->
