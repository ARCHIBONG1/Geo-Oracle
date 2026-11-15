---
name: play-co2-storage
description: CO2 storage play elements, with containment kept separate (caprock, faults, wells). Also the pressure limit, trapping mechanisms by time, and how the upstream products map to each. Load before linking the elements of a CO2 storage play and before play_containment.
---

# CO2 storage plays

## Elements

| Element | Demonstrated by | Inferred from | Owner |
|---|---|---|---|
| storage_unit | net reservoir and porosity in study-area wells; described facies | facies model, correlation | wells_petrophysics, sedimentology |
| injectivity | core or test permeability; test rates | study-area porosity through a stated porosity-permeability relation (from analogue rock alone: analogue-supported) | wells_petrophysics |
| trapping_mechanism | a depth closure (structural) passing its ledger; a demonstrated pinch-out | a closure passing its ledger, as a trap_geometry link (inferred at most); with a failing ledger, set `claimed` hypothesised and name the failing test in `note` | structural_geology, stratigraphy |
| caprock_containment | capillary entry pressure of the caprock; continuity across the site | seal facies described and correlated | wells_petrophysics, stratigraphy, sedimentology |
| fault_containment | fault seal and stability at the planned pressure with measured stress | bounded stress (regional prior): inferred at most | structural_geology |
| well_containment | legacy-well inventory with integrity records | an inventory without records: unknown | supplied legacy_wells; literature_review |
| pressure_limit | critical pressure of faults and caprock fracture pressure against the planned increase | fault stability alone | structural_geology, wells_petrophysics |

Containment is never one status. `play_containment` keeps caprock, faults and wells apart, because each fails and is monitored differently. It reports the pressure margin per fault from `fault_stability` against the planned increase. Give `caprock_limit_dp_mpa`, the pressure increase the caprock can take: its fracture pressure (or the minimum horizontal stress from a minifrac or leak-off test) minus the initial pore pressure at the caprock, in MPa, with `caprock_limit_source` (the test or claim id): the margin is then read against the caprock too. A value of 0 or less is refused. Without it, the result and a warning say the caprock limit was not tested.

## Timing (PT4)

PT4 reads containment through time from the events chart. Give `play_events_chart` one `containment` event per period (injection, post-injection) and per barrier (caprock, faults, wells), each with `holds: true` or `holds: false` and the claim that supports it. Every event needs start_ma and end_ma: 0 for the present day and for planned periods. An event without `holds` tests nothing, and PT4 stays not tested unless another event fails. Without containment events, PT4 is not tested and the play cannot be supported.

## Trapping by time

Each trapping mechanism acts over its own time (IPCC 2005, Fig. 5.9 and Box 5.4):
- structural or stratigraphic, from injection;
- residual, during migration, over years to decades;
- dissolution, dominating from tens to hundreds of years;
- mineral, over a thousand years or longer.

A screening play rests on the first. The others need rock and brine data, and you present them as such.

## Pitfalls

- Capacity discussed without containment.
- Legacy wells in a depleted field overlooked. A well that penetrates the caprock without an integrity record leaves well containment unknown, and it caps the play.
- A fault-stability verdict on bounded stress reported as demonstrated. Until stress is measured, the verdict is inferred.

## Reporting

Give `containment_assessment` with the four statuses, and the storage play's matrix with the weakest link. The discriminating evidence usually names capillary data, well integrity and measured stress first.

<!-- end of skill: play-co2-storage -->
