---
name: regional-thermal
description: Heat-flow statistics, conductive geotherms with sourced conductivities, calibration to corrected well temperatures, palaeoclimate and groundwater biases, and the thermal_model product. Load before reg_heat_flow_stats, reg_geotherm or reg_geotherm_calibrate.
---

# Thermal regime

## Heat flow: `reg_heat_flow_stats`

- Points of `ghfdb2024` (or a supplied table) within `radius_km`: median, p10-p90, distance-weighted mean, nearest point, points within 50 km. Filter by quality where the dataset carries it (e.g. `filters={"q_uncertainty": {"max": 20}}`).
- **Biases to name**: shallow onshore measurements carry palaeoclimate and groundwater-flow effects; offshore probe measurements can be biased by sedimentation; old compilations mix methods. Report the p10-p90 range, not just the median.
- No point within 50 km (the tool warns): the value is regional; T3 is not met at study-area scale.

## Geotherm: `reg_geotherm`

- 1-D steady-state conduction with layered conductivity (W/m/K) and heat production (microW/m3), from the surface (seabed) temperature. Every parameter is sourced: heat flow from the statistics or a calibration (cite `prov:`), conductivities from measured or published lithology values (cite the finding), surface temperature from the seabed or ground temperature.
- Typical conductivities (state them as assumptions, with a source): shale 1.2-2.0, sandstone 2.5-4.0, limestone 2.0-3.0, salt 5-6 W/m/K. Heat production: shales 1-2, sandstones 0.5-1, carbonates 0.2-0.5 microW/m3.
- The product `thermal_model` (depth, temperature, heat flow) is a regional prior for wells_petrophysics (temperature gradient) and subsurface_play (maturity timing); a 1-D steady-state model carries no lateral or transient effects, which the sidecar states.

## Calibration: `reg_geotherm_calibrate`

- Fits surface heat flow to corrected well temperatures from a petrophysics `temperature_profile` product (Horner static or DST). Give `water_depth_m` offshore so depths count from the seabed.
- The fitted heat flow depends on the conductivities assumed: report the pair, and compare the calibrated value with the regional heat-flow statistics (agreement supports transfer; disagreement points at local conductivity or transient effects).

## Reporting

- `[regional] heat flow median 61 mW/m2 (p10-p90 55-68, n=102 within 100 km, nearest 12 km)` as a measurement, `literature`, with the database release.
- `[transfer] … applies to the study area; T1 met, T2 n/a, T3 met (12 km), T4 unknown, T5 met (calibrated 60 mW/m2 at W1)` as an inference.
