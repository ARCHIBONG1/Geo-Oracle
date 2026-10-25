---
name: petro-pressure-temperature
description: Formation pressure and temperature. It covers pretest QC, gradient legs and fluid densities, contacts from gradient intersections and excess pressure. It also covers Horner static temperatures and gradients, and the pressure_profile, fluid_contacts and temperature_profile products. Load before petro_pressure or petro_temperature.
---

# Pressure and temperature

## Pressure: `petro_pressure`

1. **Register the pretests**: `pressure_data` table, with md_m or tvdss_m, pressure (psi, bar or MPa), and where available `mobility_md_cp` and `quality`. Tests in MD need a well with a trajectory.
   **Stress tests are not pressures.** The tool sets a stress-test row aside before the fit. That is a row whose `test` (or type) column says LOT, XLOT, FIT or minifrac, or whose note says leak-off or formation integrity. The tool publishes it in the `pressure_profile` with status `stress test: <type>` (result `stress_tests`). A leak-off approximates the least principal stress. A FIT stops before leak-off and is only a lower bound. Hand the profile to structural_geology, whose `sg_stress_tensor` reads a LOT from it as Shmin. Never fit a stress test as a pore pressure: one LOT among a few pressure points can turn the fitted gradient negative.
2. **QC**: the tool excludes tests with a bad `quality` (tight, lost seal, supercharged, dry, unstable) or with mobility below `min_mobility` (default 0.5 md/cP). It gives the reason for each. Low-mobility tests read high: supercharging. Report every exclusion.
3. **Legs**: the tool splits the accepted points into 1 to `max_legs` straight gradient legs. BIC chooses the number. The tool merges adjacent legs of the same fluid on one line (one aquifer sampled in two sands).
   - Each leg gives a fluid density (gradient / 1.42233 psi/m per g/cm3) and a fluid type: gas below 0.3, oil 0.4-0.9, water above 0.97 g/cm3. The tool names a density that falls in an overlap as such.
   - **At least three points per leg.** The tool never gives a two-point "gradient" as a leg. Never give one in your findings.
4. **Contacts**: where adjacent legs intersect, with an uncertainty from the fit scatter.
   - These are **free-fluid levels** (zero capillary pressure). The log contact sits higher by the entry height. That difference is expected, not a contradiction.
   - If an intersection falls outside the gap between its legs (warned), the legs are mis-assigned, or the sands are separate compartments.
5. **Excess pressure**: with `brine_density` in the parameter set, relative to hydrostatic from 1 atm at MSL.
   - A consistent positive excess is overpressure.
   - Different water lines in different sands point at separate compartments. Report that as an interpretation for structural_geology or risk.

**Claims**: a contact from gradient intersection, with at least three good points per leg, is a measurement-based `interpretation`. It can be `supported`. A contact from logs alone stays `partially_supported`.

## Temperature: `petro_temperature`

- **Register the readings** (`temperature_data`: md_m or tvdss_m, temp, `kind` such as BHT or DST, `hours_since_circulation` for BHTs).
- **Several BHTs at one depth**, taken at different times after circulation: the tool extrapolates them by **Horner**. It fits T against log10((tc + dt)/dt), where tc = `circulation_hours` from the drilling report (a parameter with source). The intercept is the static temperature.
- **A single BHT is uncorrected and reads low**: the result notes it as a minimum. Give it as one in your findings. DST and production temperatures are close to static.
- **The gradient**: the tool fits it through the corrected points. Its `register_as` gives `temperature_gradient_c_per_km` and `surface_temp_c` for the parameter set. If they differ from what you used, rerun saturation with them.

## Products

- **`pressure_profile`** (CSV: tvdss_m, pressure_psi, excess_psi, status, test; stress tests carry status `stress test: <type>`) and **`fluid_contacts`** (JSON: legs and contacts), from `petro_pressure`.
- **`temperature_profile`** (CSV: tvdss_m, temp_c, method), from `petro_temperature`.

List each in `conclusions` exactly as `Product: <kind> — <ref>` from `result.products[].conclusion_line`.

**Figure**: `petro_pressure` draws the pressure-depth plot into `result.figures`. The plot shows the pressures, the fitted gradients with their fluids, the contacts and the hydrostatic line (with `brine_density`). `petro_render(kind="pressure_depth", pressure_profile=<ref>, fluid_contacts=<ref>)` draws it again from the published products.

<!-- end of skill: petro-pressure-temperature -->
