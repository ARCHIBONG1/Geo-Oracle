---
name: petro-saturation
description: Water saturation. It covers the Rw hierarchy (water sample, SP, Pickett), temperature correction, and choosing Archie or a shaly-sand model (Simandoux, Indonesia, Waxman-Smits, dual water). It also covers sourcing a, m, n, Rsh, Qv, Rwb, and reporting saturations and fluid claims. Load before petro_rw, petro_saturation or petro_net_pay.
---

# Water saturation

## 1. Rw first: `petro_rw`

Use the first method the data allow, and record which:

| Rank | Method | Needs | Notes |
|---|---|---|---|
| 1 | `water_sample` | A registered `water_analysis` table (Rw at a temperature, or salinity) | Returns Rw at 25 degC in `register_as`. Salinity is read in Arps (1953) Table 5 at 25 degC. Above a column's last tabulated concentration, the tool scales the nearest column by the table's own column ratio; each sample's `basis` says so. The later correction to formation temperature uses Arps eq 6, not Table 5 |
| 2 | `sp` | SP curve, water-based mud, a clean water-bearing bed (`md_range`), a thick shale for the baseline (`shale_md_range`), Rmf and its temperature (header or parameters), a temperature model | Header Rmf needs a resistivity unit (ohm.m, OHMM, OHM or OHMS) and header RMFT a temperature unit (degC, degF, 'degree C', 'degree F' or K): a blank or unknown unit is refused, so register `rmf` or `rmf_temp_c`. `source_of_rmf_temp` states how the header was read. SSP = SP(clean) - SP(shale); Schlumberger chart SP-1: Kc = 61 + 0.133 T(degF); Rmfeq = 0.85 Rmf when Rmf at 75 degF exceeds 0.1 ohm.m, otherwise chart SP-2 read in reverse (saline mud, flagged); Rweq converted to Rw along the digitised solid NaCl lines of chart SP-2 at formation temperature (above 0.11 ohm.m the chart's single 'NaCl at 75 degF' line; beyond 2 ohm.m Rw = Rweq, flagged). Formation temperature must be within 75-500 degF and Rweq above NaCl saturation, else the tool refuses. The result states the basis of each step (`rmfe_basis`, `rw_basis`). NaCl waters only. The tool refuses oil-based, synthetic-based and other non-aqueous mud (OBM, SBM, NAF, invert, water-in-oil, diesel, ester, olefin, paraffin, LAO, IO). It accepts water-based labels (WBM, water, KCl, oil-free, oil-in-water, oil emulsion WBM). A label in no list is not refused: the tool warns that it is not recognised. Confirm the mud type before you use that Rw |
| 3 | `pickett` | PHIE (run `petro_porosity`) and RT in a **proven** water-bearing interval (a test there recovered formation water and no free hydrocarbon, or the pressures in it give a water gradient; with a residual oil show the Rw is a high limit). Find the tests in the rows of a registered `well_tests` table. Name the test or the pressure points in the Rw reference; `a` and `m` (or `m` fitted, if porosity varies) | Rw is only as good as the "100% water" assumption and the m used. A fitted m is in `register_as` beside Rw |
| — | Regional value | Requested via `missing_data` from regional_geology | Never a textbook default |

With `pickett`, the tool draws the Pickett plot of the interval into `result.figures`. Iso-saturation lines need `n` in the parameter set. Without it, the plot shows only the Sw = 1 line.

Register the result exactly as returned in `register_as` (value, source, reference): `rw` together with `rw_temp_c`, and the fitted `m` when the Pickett fit returns it. That Rw holds only with that m. `petro_saturation` warns when a later run uses another m. A water sample outranks log-derived Rw. Run each Rw source the data allow. To compare them, give one `report_at_temp_c` (degC, above 0 and below 374) to every `petro_rw` call. The result then also gives `rw_at_report_temp` (`value`, `temp_c`, `from_temp_c`, `note`). Arps eq 6 carries it from 25 degC for a water sample, and from the interval formation temperature for SP and Pickett. It is for comparison only: register `rw` and `rw_temp_c` from `register_as`. If the sources disagree by more than about 20%, that is a `contradiction`. It is between sample representativeness and a bed that is not clean or not fully water-bearing.

## 2. Temperature

Rw, Rmf and Rm change with temperature. The relation is Arps 1953, in degF as printed: R2 = R1 (T1 + 6.77) / (T2 + 6.77). The tool converts degC exactly. Chart Gen-6 states the relation valid 20-100 degC. `petro_saturation` corrects Rw to every sample's formation temperature. So the parameter set needs `rw_temp_c` plus a temperature model:
- **`surface_temp_c` and `temperature_gradient_c_per_km`** (e.g. `regional`, with a range), or
- **`surface_temp_c` alone**: then the tool uses the header BHT at log TD. Uncorrected BHT reads low, so the temperatures are minima: say so. The tool reads the BHT unit (degC, degF, K). It refuses a blank or unknown unit, a BHT not above `surface_temp_c`, or no finite TVDSS at TD. Then register `surface_temp_c` and `temperature_gradient_c_per_km`.

Use `rw_at_formation_temperature: true` only when the task states that Rw is already at formation temperature.

## 3. Choose the model

| Condition | Model | Also needs |
|---|---|---|
| Clean sand or carbonate (VSH below about 0.1), intergranular porosity | archie | rw, a, m, n |
| Shaly sand, saline water, Rsh known and Qv not | simandoux (the modified, total-shale form: shale term Vsh Sw / Rsh) | + rsh (deep resistivity in a thick shale) |
| Shaly sand with fresh or brackish water | indonesia | + rsh |
| Cation-exchange capacity known from core | waxman_smits (returns SwT, the water fraction of PHIT) | + qv (meq/cm3), m_star and n_star (the shaly-sand exponents of Waxman & Smits: F* = PHIT^-m*, no tortuosity factor; Juhasz: n* about m* or m* + 0.1 in sands, 2.0 in carbonates - but from core when available); B from temperature and Rw (Juhasz 1981 eq. 5), warned outside 22-200 degC. Run `petro_porosity` first, so that PHIT exists |
| Clay-bound water modelled explicitly | dual_water | + rwb (bound-water resistivity), PHIT and PHIE |
| Vuggy or fractured carbonate | archie with m from SCAL | m from core, not 2 |

**Archie in shaly rock overstates Sw**, because it reads shale conduction as water. Then pay is missed. Where VSH is above about 0.15, run a shaly-sand model too and report both. Their difference is the shale effect. The tool warns only when the whole-well median VSH is above 0.15. So check each zone yourself.

**Clay distribution** (before a shaly-sand model): register `phi_sand_clean` and `phi_shale` (total porosities of the clean sand and the shale, with sources). `petro_saturation` then returns `thomas_stieber`: the fraction of samples nearest the laminar, dispersed and structural lines, per zone. Mostly laminar points to laminated shale. Near the clean sand and near the pure shale the lines are closer than the tolerance: those samples are unclassified (`unclassified_near_clean_sand`, `unclassified_near_pure_shale`), not laminar. The tool has no laminated-sand Sw model: report that as a limitation. The diagnostic does not change Sw.

## 4. a, m, n and Rsh

- **a, m and n come from SCAL** if supplied: fit them with `petro_scal` (formation factor and resistivity index; skill `petro-core-scal`), and register its `register_as`. Otherwise a = 1, m = 2, n = 2 are allowed only as `assumption`, **with ranges** (for example m 1.8-2.2, n 1.8-2.2). The ranges let `petro_net_pay` show their effect. m and n usually dominate the uncertainty in Sw.
- **Rsh**: read it from a thick, in-gauge shale near the zone (`log_crossplot`), and cite the interval.

## 5. Run and read: `petro_saturation`

- **Output**: writes `SW`, `BVW`, `RW` (the temperature-corrected Rw) and `TEMP` into a new well file. `sw_basis` names the porosity Sw is a fraction of. Waxman-Smits: SW is SwT and BVW = SWT x PHIT. Every other model: BVW = SW x PHIE. Never compare SwT with an effective Sw directly.
- **Missing inputs**: where Rt, Rw, porosity, VSH (where used) or temperature (Waxman-Smits) is missing, Sw is missing too, not 1. `sw_missing_samples_with_rt` counts samples with Rt but no Sw.
- **Per-zone statistics**: add `tops` to get SW's 10th, 50th and 90th percentiles per zone.
- **Figure**: the tool draws the interpreted composite log (with VSH, PHIE and BVW tracks and the tops) into `result.figures`.

## 6. Fluid claims

Saturation from logs is a `measurement` (`derived`). The fluid (gas, oil, water) is an `interpretation`:
- **Supported**: when pressures, tests, samples or core agree.
- **Partially supported**: when logs are consistent (neutron-density crossover, resistivity, low Sw) but uncalibrated.
- **Name the alternatives**: low-resistivity pay (laminated or conductive minerals) against wet shaly sand; fresh water against hydrocarbons; tight rock against hydrocarbons.
- **Sw does not name the gas.** The saturation models hold for any non-conductive pore fluid. For a hydrogen target, mark every hydrogen interpretation as emerging practice.

## 7. Report

- `measurements`: give each with the provenance id. For example: "Sw (Simandoux; Rw 0.0755 ohm.m at 25 degC from the DST-1 water sample, corrected to 48-49 degC; a = 1, m = 2, n = 2 assumed) median 0.25 in SAND_B above 1142 m MD".
- `assumptions`: every parameter with its source.
- `limitations`: e.g. "m and n not measured".

<!-- end of skill: petro-saturation -->
