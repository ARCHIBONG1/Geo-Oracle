---
name: petro-pressure-temperature
description: Formation pressure, temperature and thermal properties. It covers pretest QC, gradient legs and fluid densities, contacts from gradient intersections and excess pressure. It also covers Horner static temperatures and gradients, thermal conductivity, heat capacity, heat production and heat flow from logs, and the pressure_profile, fluid_contacts, temperature_profile and thermal_properties products. Load before petro_pressure, petro_temperature or petro_thermal.
---

# Pressure and temperature

## Pressure: `petro_pressure`

1. **Register the pretests**: `pressure_data` table, with md_m or tvdss_m, pressure (psi, bar or MPa), and where available `mobility_md_cp`, `quality` and a `test` column. Tests in MD need a well with a trajectory.
   - **Absolute or gauge**: the tool takes psia. For gauge readings, set `pressure_reference="gauge"`: the tool adds 14.696 psi (`pressure_basis`).
   - **Stress tests are not pressures.** The tool sets a stress-test row aside before the fit. It reads the test name (LOT, XLOT, FIT, DFIT, PIT, SRT, minifrac) from the `test`, `type` or `kind` column. It reads phrases (leak-off, formation integrity, step rate) from those columns and the note or comment columns only. "No leak-off" is not a LOT. The tool publishes each in the `pressure_profile` with status `stress test: <type>` (result `stress_tests`, each with its `reading`).
   - **Readings**: a leak-off approximates the least principal stress; XLOT, minifrac and DFIT closure give it. A FIT stops before leak-off: a lower bound only. A PIT is a LOT or a FIT: read the record. An SRT gives the fracture extension pressure: an upper bound, not Shmin.
   - Hand the profile to structural_geology. Its `sg_stress_tensor` reads LOT, XLOT, minifrac and DFIT rows as Shmin. It reports a FIT or a PIT as a lower bound and an SRT as an upper bound. Give the record of each PIT in your findings. Never fit a stress test as a pore pressure: one LOT among a few pressure points can turn the fitted gradient negative.
2. **QC**: the tool excludes a test whose `quality` names a failure. The failure words are tight, lost seal, no seal, supercharged, dry, unstable, failed, aborted, invalid, poor and no stabilisation. A negated word passes, also with up to two words between, for example "not supercharged", "no seal failure" or "no sign of supercharging". The tool also excludes a test with mobility below `min_mobility` (default 0.5 md/cP). It gives the reason for each. Low-mobility tests read high: supercharging. Report every exclusion. Tests with no quality and no mobility are `unscreened`, not good: say so.
3. **Legs**: the tool splits the accepted points into 1 to `max_legs` straight gradient legs. BIC chooses the number. The tool merges adjacent legs of the same fluid on one line (one aquifer sampled in two sands).
   - Each leg gives a fluid density (gradient / 1.42233 psi/m per g/cm3) and a fluid type: gas below 0.3, oil 0.4-0.9, water above 0.97 g/cm3. The tool names a density that falls in an overlap as such.
   - **At least three points per leg.** The tool never gives a two-point "gradient" as a leg. Never give one in your findings.
4. **Contacts**: where adjacent legs intersect. `uncertainty_m` comes from the full regression variance of both legs, so it grows with the distance from the points.
   - These are **free-fluid levels** (zero capillary pressure). The log contact sits higher by the entry height. That difference is expected, not a contradiction.
   - If an intersection falls outside the gap between its legs (warned), the legs are mis-assigned, or the sands are separate compartments.
5. **Excess pressure**: with `brine_density` in the parameter set, relative to a hydrostatic from 1 atm at `hydrostatic_reference_elevation_m` (default 0, MSL). Offshore, MSL is right. Onshore, register the water-table elevation with its source. The tool warns when `ground_elevation_m` is above 0 and no reference is registered.
   - A consistent positive excess is overpressure.
   - Different water lines in different sands point at separate compartments. Report that as an interpretation for structural_geology or risk.

**Claims**: a contact from gradient intersection, with at least three good points per leg, is a measurement-based `interpretation`. It can be `supported`. A contact from logs alone stays `partially_supported`.

## Temperature: `petro_temperature`

- **Register the readings** (`temperature_data`: md_m or tvdss_m, temp, `kind` such as BHT or DST, `hours_since_circulation` for BHTs). A blank `kind` is `unclassified`, not a BHT: give the kind.
- **Several BHTs at one depth**, taken at different times after circulation: the tool extrapolates them by **Horner**. It fits T against log10((tc + dt)/dt), where tc = `circulation_hours` from the drilling report (a parameter with source). The intercept is the static temperature.
- **A single BHT is uncorrected and reads low**: the result notes it as a minimum. Give it as one in your findings. DST and production temperatures are close to static.
- **The gradient**: the tool fits it through the points, with `gradient_se_c_per_km` and `residual_sd_c`. Its `register_as` gives `temperature_gradient_c_per_km` and `surface_temp_c` for the parameter set. The source is `test` only when every point is corrected or a stated non-BHT kind. Otherwise it is `assumption`, with low and high (`register_as_basis`): the high side allows for uncorrected BHTs and unclassified points that read 10 degC low.
  - When only BHT or unclassified points exist, the high case also passes through a registered `surface_temp_c` (in the `parameter_set`).
  - A range that the points cannot set is degenerate. The entry then has no low or high, `register_as_degenerate` names it, and a warning follows. Set low and high by hand, or register `surface_temp_c` with its source and run the tool again.
  - Register the entries as given. If they differ from what you used, rerun saturation with them.
- **Microbial window**: when the task asks for it (for example a hydrogen play), give the formation temperature with its basis and range. The window limits come from the play work, not from this skill.

## Thermal properties: `petro_thermal`

**Geothermal.** Temperature is a primary deliverable with an error band. `petro_thermal` gives log-derived estimates, at laboratory conditions by default. A core measurement outranks them. Without core, their claims stay `partially_supported` at most. Heat flow needs a registered gradient.
- **Inputs**: VSH and PHIT (or PHIE, warned) with a composition: `mineralogy` `multimineral` (V_* and PHI_MM), `fractions` (F_QUARTZ, F_CALCITE, F_DOLOMITE, F_ANHYDRITE, F_HALITE from `matrix_id`; for evaporites run `petro_lithology` with `matrix_minerals`, for example quartz, anhydrite, halite) or `matrix` (one mineral, an assumption). The default `auto` takes the first of these that the well has. The shale end member is `tc_shale_w_mk` (default 1.8 W/(m K), illite; warned).
- **Shale water**: VSH is wet shale. Register `phi_shale` with its source: the clay-bound water is then part of the pore water. Without `phi_shale`, the pore water is PHIE and the conductivity reads high. Without PHIE too, the conductivity reads low. The tool warns in both cases.
- **Outputs**: TC (geometric mean), with TC_LOW and TC_HIGH (harmonic and arithmetic Wiener bounds); RHOC (volumetric heat capacity at 25 degC); DIFF (diffusivity = TC / RHOC); HPROD (heat production). The `thermal_properties` product holds them.
- **Heat production**: `spectral` (K, TH, U with RHOB; Rybach) is preferred. It needs curves the loader read as spectral K, TH and U. A file curve with family `other` is refused in `spectral`, and `auto` then uses GR. `gamma_ray` (total GR; Bucker & Rybach 1996) is lower confidence. The GR line is not for igneous or ultramafic rock. The tool warns when the median GR is above 300 API.
- **Heat flow** = gradient x the harmonic-mean conductivity of the interval (layers in series). It needs a registered `temperature_gradient_c_per_km` measured over the same interval, or `zone_gradients_c_per_km` from a temperature log. Uncorrected BHTs read low, so a gradient from them usually understates the heat flow. It assumes conduction only.
  - `interval.heat_flow_bounds_mw_m2` is an envelope (`heat_flow_bounds_basis`), not P10/P90.
  - The tool records each `zone_gradients_c_per_km` value in `result.assumptions` as unsourced. The zone `heat_flow_class` is conditional: state the source of the gradient.
  - A zone with no samples returns None, with `heat_flow_note` or `tc_harmonic_note`.
- **Conditions**: by default, values are at laboratory conditions (25 degC, water-filled pores). Rock conductivity falls with temperature, and pore-water conductivity rises. For formation conditions:
  - `temperature_correction` `sedimentary` (0-300 degC) or `crystalline` (basement, 0-500 degC) takes the solid mean to formation temperature (Vosteen & Schellschmidt 2003). Halite stays at 25 degC. Above 150 degC the error is +8/-34 %. Outside the range the value is NaN.
  - `pore_fluid` `water`, `oil`, `gas` (methane), `co2` or `h2` fills PHIE x (1 - SW) with that fluid, or PHIT x (1 - SW) when SW is SwT; the rest is water. It needs SW. Only `oil`, `gas`, `co2` and `h2` need a pore pressure (`reservoir_pressure_mpa`, else `brine_density` for a hydrostatic estimate). Oil needs `oil_api`; its values are dead oil at 1 atm, about 10 %. The water is pure water: salt lowers its conductivity a few per cent.
  - Temperature: `surface_temp_c` with `temperature_gradient_c_per_km`, else the TEMP curve, else a header BHT (a minimum). `result.formation_conditions` states each basis. RHOC keeps the solids at 25 degC.
- **Not available** (`not_available`): well-test and PTS analysis, fracture permeability, pressure correction of the solid conductivity, temperature correction of the solid heat capacity. List them in `limitations` when the task needs them. Put the data that would allow them (for example a test or a core measurement) in `missing_data`.

## Products

- **`pressure_profile`** (CSV: tvdss_m, pressure_psi, excess_psi, status, test; stress tests carry status `stress test: <type>`) and **`fluid_contacts`** (JSON: legs and contacts), from `petro_pressure`.
- **`temperature_profile`** (CSV: tvdss_m, temp_c, method), from `petro_temperature`.
- **`thermal_properties`** (CSV: md_m, tvdss_m, tc_w_mk, tc_low_w_mk, tc_high_w_mk, rhoc_mj_m3k, diffusivity_m2_s, hprod_uw_m3), from `petro_thermal`.

List each in `conclusions` exactly as `Product: <kind> — <ref>` from `result.products[].conclusion_line`.

**Figure**: `petro_pressure` draws the pressure-depth plot into `result.figures`. The plot shows the pressures, the fitted gradients with their fluids, the contacts and the hydrostatic line (with `brine_density`). `petro_render(kind="pressure_depth", pressure_profile=<ref>, fluid_contacts=<ref>)` draws it again from the published products.

<!-- end of skill: petro-pressure-temperature -->
