---
name: play-geothermal
description: Geothermal play types (convection- and conduction-dominated) and the elements (heat, permeability, fluid and recharge, structural setting and stress). Also how the upstream products map to each. Load before linking the elements of a geothermal play.
---

# Geothermal plays

## Play types (Moeck 2014)

The play types are convection-dominated (magmatic, plutonic, extensional domain) and conduction-dominated (intracratonic basin, orogenic belt, basement). The type sets which element is usually limiting: permeability and recharge in basins, heat and fluid in basement.

## Elements

| Element | Demonstrated by | Inferred from | Owner |
|---|---|---|---|
| heat | temperatures measured at depth in study-area wells | heat flow at the site (a geotherm calibrated only regionally: analogue-supported) | wells_petrophysics (temperature_profile), regional_geology (thermal_model) |
| permeability | core or test permeability; fracture data from image logs | facies and diagenesis; fault network | wells_petrophysics, sedimentology, structural_geology |
| fluid_recharge | pressure history or hydrochemistry showing recharge | a connected aquifer in the framework | wells_petrophysics, regional_geology |
| structural_setting | the fault network and stress field at the site | the site's fault network interpreted (regional structural elements alone: analogue-supported) | structural_geology, regional_geology |
| cap_seal (supporting, convective systems) | a seal over the reservoir | seal facies | stratigraphy |
| stress_for_stimulation (supporting, low permeability) | measured stress and fault stability | fault stability at the site (the regional stress field alone: analogue-supported) | structural_geology, regional_geology |

A petrothermal (enhanced or engineered) play has no natural fluid recharge. The template still holds `fluid_recharge` as critical: state it as engineered (the planned circulation), with its basis, and say in `limitations` that the template has no petrothermal path.

## Timing (PT4)

Heat and recharge must be sustained over the project life. In the events chart, give `heat` and `recharge` events with `holds`.

## Pitfalls

- Temperature alone treated as a play. Without permeability and fluid, there is no play.
- Connectivity between injector and producer assumed from one well.
- Depositional permeability assumed to survive deep burial and heating.

## Reporting

As for the other domains: the matrix, the weakest link, the discriminating evidence (usually permeability and recharge data).

<!-- end of skill: play-geothermal -->
