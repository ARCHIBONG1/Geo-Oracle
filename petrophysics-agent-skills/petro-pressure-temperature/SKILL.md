---
name: petro-pressure-temperature
description: Formation pressure and temperature - pretest QC, gradient legs and fluid densities, contacts from gradient intersections, excess pressure, Horner static temperatures and gradients, and the pressure_profile, fluid_contacts and temperature_profile products. Load before petro_pressure or petro_temperature.
---

# Pressure and temperature

## Pressure: `petro_pressure`

1. **Register the pretests**: `pressure_data` table, with md_m or tvdss_m, pressure (psi, bar or MPa), and where available `mobility_md_cp` and `quality`. Tests in MD need a well with a trajectory.
2. **QC**: tests with a bad `quality` (tight, lost seal, supercharged, dry, unstable) or with mobility below `min_mobility` (default 0.5 md/cP) are excluded, each with its reason. Low-mobility tests read high: supercharging. Report every exclusion.
3. **Legs**: the accepted points are split into 1 to `max_legs` straight gradient legs. The number is chosen by BIC, and adjacent legs of the same fluid on one line are merged (one aquifer sampled in two sands).
   - Each leg gives a fluid density (gradient / 1.42233 psi/m per g/cm3) and a fluid type: gas below 0.3, oil 0.4-0.9, water above 0.97 g/cm3. Overlaps are named as such.
   - **At least three points per leg.** A two-point "gradient" is not reported as one.
4. **Contacts**: where adjacent legs intersect, with an uncertainty from the fit scatter.
   - These are **free-fluid levels** (zero capillary pressure). The log contact sits higher by the entry height; that difference is expected, not a contradiction.
   - If an intersection falls outside the gap between its legs (warned), the legs are mis-assigned or the sands are separate compartments.
5. **Excess pressure**: with `brine_density` in the parameter set, relative to hydrostatic from 1 atm at MSL.
   - A consistent positive excess is overpressure.
   - Different water lines in different sands point at separate compartments. Report that as an interpretation for structural_geology or risk.

**Claims**: a contact from gradient intersection with at least three good points per leg is a measurement-based `interpretation` that can be `supported`. A contact from logs alone stays `partially_supported`.

## Temperature: `petro_temperature`

- **Register the readings** (`temperature_data`: md_m or tvdss_m, temp, `kind` such as BHT or DST, `hours_since_circulation` for BHTs).
- **Several BHTs at one depth**, taken at different times after circulation, are extrapolated by **Horner**: T against log10((tc + dt)/dt), where tc = `circulation_hours` from the drilling report (a parameter with source). The intercept is the static temperature.
- **A single BHT is uncorrected and reads low**: it is reported as a minimum. DST and production temperatures are close to static.
- **The gradient** is fitted through the corrected points. Its `register_as` gives `temperature_gradient_c_per_km` and `surface_temp_c` for the parameter set: rerun saturation with them if they differ from what you used.

## Products

- **`pressure_profile`** (CSV: tvdss_m, pressure_psi, excess_psi, status) and **`fluid_contacts`** (JSON: legs and contacts), from `petro_pressure`.
- **`temperature_profile`** (CSV: tvdss_m, temp_c, method), from `petro_temperature`.

List each in `conclusions` exactly as `Product: <kind> — <ref>` from `result.products[].conclusion_line`.
