---
name: play-geothermal
description: Geothermal play types (convection- and conduction-dominated), the elements heat, permeability, fluid and recharge, structural setting and stress, and how the upstream products map to each. Load before linking the elements of a geothermal play.
---

# Geothermal plays

## Play types (Moeck 2014)

Convection-dominated (magmatic, plutonic, extensional domain) and conduction-dominated (intracratonic basin, orogenic belt, basement). The type sets which element is usually limiting: permeability and recharge in basins, heat and fluid in basement.

## Elements

| Element | Demonstrated by | Inferred from | Owner |
|---|---|---|---|
| heat | temperatures measured at depth in study-area wells | a geotherm calibrated regionally; heat flow | wells_petrophysics (temperature_profile), regional_geology (thermal_model) |
| permeability | core or test permeability; fracture data from image logs | facies and diagenesis; fault network | wells_petrophysics, sedimentology, structural_geology |
| fluid_recharge | pressure history or hydrochemistry showing recharge | a connected aquifer in the framework | wells_petrophysics, regional_geology |
| structural_setting | the fault network and stress field at the site | regional structural elements | structural_geology, regional_geology |
| cap_seal (supporting, convective systems) | a seal over the reservoir | seal facies | stratigraphy |
| stress_for_stimulation (supporting, low permeability) | measured stress and fault stability | the regional stress field | structural_geology, regional_geology |

## Timing (PT4)

Heat and recharge must be sustained over the project life: give `heat` and `recharge` events with `holds` in the events chart.

## Pitfalls

- Temperature alone treated as a play: without permeability and fluid there is no play.
- Connectivity between injector and producer assumed from one well.
- Depositional permeability assumed to survive deep burial and heating.

## Reporting

As for the other domains: the matrix, the weakest link, the discriminating evidence (usually permeability and recharge data).
