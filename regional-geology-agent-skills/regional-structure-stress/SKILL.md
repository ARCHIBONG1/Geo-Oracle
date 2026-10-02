---
name: regional-structure-stress
description: Regional fault elements and trends, SHmax orientation and stress regime from stress indicators, domain boundaries, and what can be carried into the study area. Load before reg_fault_elements, reg_trend_statistics or reg_shmax.
---

# Structure and stress

## Fault elements: `reg_fault_elements`

- Faults whose nearest point lies within `radius_km` of the AOI, from `gem_faults` or a supplied fault layer: length, trend (axial, first to last vertex), slip type, distance. Length-weighted trend statistics follow.
- **Coverage, not absence**: zero faults within the radius means the dataset has none mapped there. The GEM database holds active faults; older, inactive structures are absent by design. Say which.
- The product `structural_elements` is regional: a consumer uses it as the expected structural grain, not as mapped faults in the AOI.

## Trends: `reg_trend_statistics`

Axial statistics (0-180 degrees): mean direction, concentration R (1 = all parallel), circular SD. With R below about 0.5, report a spread or several modes, not one trend.

## SHmax: `reg_shmax`

- **Only A-C quality indicators** are used (World Stress Map ranking: A within 15 degrees, B 20, C 25); D and E are counted as excluded. Weights A 1.0, B 0.75, C 0.5.
- The result gives the axial mean, circular SD, counts by quality and type, regime counts, and the mean per distance bin. **Read the distance bins**: a stable azimuth from 0-50 km to 100-200 km supports transfer; a change points at a domain boundary.
- A circular SD above 30 degrees (the tool warns) means a range, not an azimuth.
- Regime from the regime counts: a mix of NF and SS with few TF is reported as such, never resolved by majority.

## Transfer

- T1: is there a basin-bounding fault, salt wall or terrane boundary between the indicators and the AOI? Check the fault elements.
- T3: the nearest A-C indicator's distance; indicators inside the AOI make it `[study-area]` evidence.
- T4: local salt, intrusions or recent faulting can rotate stresses; unknown unless the local specialists say.
- Hand to structural_geology as the `stress_field` product with your T1-T5 record; fault-stability analysis in the AOI is theirs.

## Reporting

- `[regional] SHmax N150E +/- 8 (n=197 A-C indicators within 150 km; nearest 5 km; regime SS 70 %, NF 30 %)` as a measurement, `literature`, `prov:…; World Stress Map 2016 release`.
- `[transfer] …` as an inference with T1-T5.
- Rose diagram with `reg_render(kind=rose)`: count and quality filter are printed.
