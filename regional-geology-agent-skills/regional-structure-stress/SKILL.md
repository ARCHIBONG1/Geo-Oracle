---
name: regional-structure-stress
description: Regional fault elements and trends, SHmax orientation and stress regime from stress indicators, domain boundaries, and what can be carried into the study area. Load before reg_fault_elements, reg_trend_statistics or reg_shmax.
---

# Structure and stress

## Fault elements: `reg_fault_elements`

- The tool returns the faults whose nearest point lies within `radius_km` of the AOI, from `gem_faults` or a supplied fault layer: length, trend (axial, first to last vertex), slip type, distance. Length-weighted trend statistics follow.
- **Coverage, not absence**: zero faults within the radius means that the dataset has none mapped there. The GEM database holds active faults. Older, inactive structures are absent by design. Say which.
- The product `structural_elements` is regional. A consumer uses it as the expected structural grain, not as mapped faults in the AOI.
- The tool draws a rose of the fault trends and returns it in `result.figures`. The rose counts each fault once. The trend statistics are length-weighted.

## Trends: `reg_trend_statistics`

Axial statistics (0-180 degrees): mean direction, concentration R (1 = all parallel), circular SD. If R is below about 0.5, report a spread or several modes, not one trend.

## SHmax: `reg_shmax`

- **Only A-C quality indicators** are used (World Stress Map ranking: A within 15 degrees, B 20, C 25). The tool counts D and E as excluded. Weights: A 1.0, B 0.75, C 0.5.
- The result gives the axial mean, circular SD, counts by quality and type, regime counts, and the mean per distance bin. **Read the distance bins**. A stable azimuth from 0-50 km to 100-200 km supports transfer. A change points at a domain boundary.
- A circular SD above 30 degrees (the tool warns) means a range, not an azimuth.
- Take the regime from the regime counts. Report a mix of NF and SS with few TF as such, and never resolve it by majority.

## Transfer

- T1: is there a basin-bounding fault, salt wall or terrane boundary between the indicators and the AOI? Check the fault elements.
- T3: the nearest A-C indicator's distance. Indicators inside the AOI make it `[study-area]` evidence.
- T4: local salt, intrusions or recent faulting can rotate stresses. T4 is unknown unless the local specialists say.
- Hand the field to structural_geology as the `stress_field` product, with your T1-T5 record. Fault-stability analysis in the AOI is theirs.

## Reporting

- `[regional] SHmax N150E +/- 8 (n=197 A-C indicators within 150 km; nearest 5 km; regime SS 70 %, NF 30 %)` as a measurement, `literature`, `prov:…; World Stress Map 2016 release`.
- `[transfer] …` as an inference with T1-T5.
- `reg_shmax` draws the rose of the A-C indicators itself (`result.figures`), with the mean and the circular SD marked. The rose counts each indicator once. The mean is quality-weighted. Do not draw a rose of typed azimuths for it.

<!-- end of skill: regional-structure-stress -->
