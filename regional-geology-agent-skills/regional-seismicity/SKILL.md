---
name: regional-seismicity
description: Earthquake catalogue summaries, magnitude of completeness, b-values and their limits, and the seismicity_summary product for risk work. Load before reg_catalogue_summary, reg_completeness or reg_b_value.
---

# Seismicity

## Order of work

1. `reg_catalogue_summary(source, aoi, radius_km)`: counts, largest events with distance and depth, depth distribution, time window. Radius 300 km for a regional view, 50-100 km for local hazard.
2. `reg_completeness`: Mc by maximum curvature and the goodness-of-fit test (Wiemer & Wyss 2000); the recommended Mc is the larger. **Always before a b-value.**
3. `reg_b_value(mc=…)`: maximum likelihood (Aki 1965) with the Shi & Bolt (1982) uncertainty. **Refused below 50 events above Mc**: then report counts and the largest events only, and say a b-value is not supported.

## Reading the result

- The catalogue snapshot's own limits come from the catalogue entry: its bounding box, minimum magnitude and start date (e.g. `usgs_comcat_<region>`, "M>=2.0 since 1970 in bbox …"). Completeness varies in time: a regional network improves; say which window the b-value uses.
- Few events within 50 km of the AOI can mean quiescence or poor detection: report both possibilities unless the catalogue's completeness settles it.
- Depth percentiles: fixed default depths (e.g. 10 km) in a catalogue are a reporting artefact, not a measurement; say so when depths cluster at one value.

## Transfer and consumers

- Seismicity is regional by nature: T3 is the distance of the nearest events and the density within 50 km.
- `seismicity_summary` goes to risk_uncertainty (induced-seismicity screening for CO2 storage and geothermal) and structural_geology (active faulting). For CO2 and geothermal tasks, report it even when not asked.

## Reporting

- `[regional] 6,727 events M>=1.5 within 300 km (1970-2026); Mc 2.5; b 0.99 +/- 0.01; largest M 4.8 at 120 km` as measurements with `prov:` ids and the catalogue name and snapshot version.
- Limitations: catalogue window and bounds, network changes, depth artefacts.
