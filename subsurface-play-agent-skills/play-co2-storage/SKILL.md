---
name: play-co2-storage
description: CO2 storage play elements, containment kept separate (caprock, faults, wells), the pressure limit, trapping mechanisms by time, and how the upstream products map to each. Load before linking the elements of a CO2 storage play and before play_containment.
---

# CO2 storage plays

## Elements

| Element | Demonstrated by | Inferred from | Owner |
|---|---|---|---|
| storage_unit | net reservoir and porosity in study-area wells; described facies | facies model, correlation | wells_petrophysics, sedimentology |
| injectivity | core or test permeability; test rates | porosity-permeability from analogue rock | wells_petrophysics |
| trapping_mechanism | a depth closure (structural) passing its ledger; a demonstrated pinch-out | a closure with a failing ledger | structural_geology, stratigraphy |
| caprock_containment | capillary entry pressure of the caprock; continuity across the site | seal facies described and correlated | wells_petrophysics, stratigraphy, sedimentology |
| fault_containment | fault seal and stability at the planned pressure with measured stress | bounded stress (regional prior): inferred at most | structural_geology |
| well_containment | legacy-well inventory with integrity records | an inventory without records: unknown | supplied legacy_wells; literature_review |
| pressure_limit | critical pressure of faults and caprock fracture pressure against the planned increase | fault stability alone | structural_geology, wells_petrophysics |

Containment is never one status: `play_containment` keeps caprock, faults and wells apart because each fails and is monitored differently, and reports the pressure margin per fault from `fault_stability` against the planned increase.

## Trapping by time

Structural or stratigraphic from injection; residual over years to decades; dissolution over centuries; mineral over millennia. A screening play rests on the first; the others need rock and brine data and are reported as such.

## Pitfalls

- Capacity discussed without containment.
- Legacy wells in a depleted field overlooked: a well that penetrates the caprock without an integrity record leaves well containment unknown and caps the play.
- A fault-stability verdict on bounded stress reported as demonstrated: it is inferred until stress is measured.

## Reporting

`containment_assessment` with the four statuses; the storage play's matrix with the weakest link; discriminating evidence usually names capillary data, well integrity and measured stress first.
