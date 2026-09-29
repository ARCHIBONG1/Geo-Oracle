---
name: petro-saturation
description: Water saturation - the Rw hierarchy (water sample, SP, Pickett), temperature correction, choosing Archie or a shaly-sand model (Simandoux, Indonesia, Waxman-Smits, dual water), sourcing a, m, n, Rsh, Qv, Rwb, and reporting saturations and fluid claims. Load before petro_rw, petro_saturation or petro_net_pay.
---

# Water saturation

## 1. Rw first: `petro_rw`

Use the first method the data allow, and record which:

| Rank | Method | Needs | Notes |
|---|---|---|---|
| 1 | `water_sample` | A registered `water_analysis` table (Rw at a temperature, or salinity) | Returns Rw at 25 degC in `register_as` |
| 2 | `sp` | SP curve, water-based mud, a clean water-bearing bed (`md_range`), a thick shale for the baseline (`shale_md_range`), Rmf and its temperature (header or parameters), a temperature model | SSP = SP(clean) - SP(shale); Rw = Rwe from 0.85 Rmf. Approximate below 0.1 ohm.m (flagged). Not with oil-based mud |
| 3 | `pickett` | PHIE (run `petro_porosity`) and RT in a **proven** water-bearing interval; `a` and `m` (or `m` fitted, if porosity varies) | Rw is only as good as the "100% water" assumption and the m used |
| — | Regional value | Requested via `missing_data` from regional_geology | Never a textbook default |

Register the result exactly as returned in `register_as` (value, source, reference): `rw` together with `rw_temp_c`. A water sample outranks log-derived Rw. If they disagree by more than about 20%, that is a `contradiction`: sample representativeness versus a bed that is not clean or not fully water-bearing.

## 2. Temperature

Rw, Rmf and Rm change with temperature (Arps: R2 = R1 (T1 + 21.5) / (T2 + 21.5), degC). `petro_saturation` corrects Rw to every sample's formation temperature, so the parameter set needs `rw_temp_c` plus a temperature model:
- **`surface_temp_c` and `temperature_gradient_c_per_km`** (e.g. `regional`, with a range), or
- **`surface_temp_c` alone**, in which case the header BHT at log TD is used. Uncorrected BHT reads low, so the temperatures are minima: say so.

Only when the task states that Rw is already at formation temperature, use `rw_at_formation_temperature: true`.

## 3. Choose the model

| Condition | Model | Also needs |
|---|---|---|
| Clean sand or carbonate (VSH below about 0.1), intergranular porosity | archie | rw, a, m, n |
| Shaly sand, saline water, dispersed or structural clay | simandoux | + rsh (deep resistivity in a thick shale) |
| Shaly sand with fresh or brackish water | indonesia | + rsh |
| Cation-exchange capacity known from core | waxman_smits | + qv (meq/cm3); B from temperature and Rw (Juhasz) |
| Clay-bound water modelled explicitly | dual_water | + rwb (bound-water resistivity), PHIT and PHIE |
| Vuggy or fractured carbonate | archie with m from SCAL | m from core, not 2 |

**Archie in shaly rock overstates Sw**, because shale conduction is read as water, and pay is missed. Where VSH is above about 0.15, run a shaly-sand model too and report both. Their difference is the shale effect. The tool warns when Archie meets shale.

## 4. a, m, n and Rsh

- **a, m and n come from SCAL** (`scal`) if supplied. Otherwise a = 1, m = 2, n = 2 are allowed only as `assumption`, **with ranges** (e.g. m 1.8-2.2, n 1.8-2.2), so `petro_net_pay` can show their effect. m and n usually dominate the uncertainty in Sw.
- **Rsh** is read from a thick, in-gauge shale near the zone (`log_crossplot`, citing the interval).

## 5. Run and read: `petro_saturation`

- **Output**: writes `SW`, `BVW` (= SW x PHIE), `RW` (the temperature-corrected Rw) and `TEMP` into a new well file.
- **Per-zone statistics**: add `tops` to get SW's 10th, 50th and 90th percentiles per zone.

## 6. Fluid claims

Saturation from logs is a `measurement` (`derived`). The fluid (gas, oil, water) is an `interpretation`:
- **Supported**: when pressures, tests, samples or core agree.
- **Partially supported**: when logs are consistent (neutron-density crossover, resistivity, low Sw) but uncalibrated.
- **Name the alternatives**: low-resistivity pay (laminated or conductive minerals) against wet shaly sand; fresh water against hydrocarbons; tight rock against hydrocarbons.

## 7. Report

- `measurements`: "Sw (Simandoux; Rw 0.0755 ohm.m at 25 degC from the DST-1 water sample, corrected to 48-49 degC; a = 1, m = 2, n = 2 assumed) median 0.25 in SAND_B above 1142 m MD", with the provenance id.
- `assumptions`: every parameter with its source.
- `limitations`: e.g. "m and n not measured".
