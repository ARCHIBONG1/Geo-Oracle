---
name: petro-data-qc
description: Getting well data in and checking it - importing uploads, loading LAS/CSV logs (mnemonics, units, unit conflicts, header facts, datums), registering tables (tops, deviation survey, checkshots, core, pressures, water analyses, temperatures), trajectories and TVDSS, log QC, reading interval descriptions, zones from tops, registering parameters with sources, and figures. Load before loading or quality-controlling any well data.
---

# Well data and quality control

## 1. Getting files in

| Where the file is | Do |
|---|---|
| Named by a path in the shared inputs folder (`operations/inputs`) | Nothing: use the path |
| Uploaded in your chat (you see it under `/opt/tf/uploads/`) | `petro_import_upload(sandbox_path="/opt/tf/uploads/W1.las")` |
| Uploaded in Geo Oracle's chat (the task says so) | `petro_import_upload(sandbox_path=..., source_agent="geo-oracle")` |

The import returns a path such as `uploads/W1.las`; use it from then on. If the import is refused because the file is not in any recent sandbox, ask for the file to be uploaded again or copied into `operations/inputs`, via `missing_data`.

## 2. Loading logs: `petro_load_well`

It loads LAS 2.0/3.0 or CSV logs. DLIS is not supported; ask for LAS.

**What it does:**
- **Depth**: converted to metres MD (feet are detected from the header). The index must be MD; a file indexed in TVD or TVDSS is refused.
- **Mnemonics**: mapped to canonical curves, first match in priority order. Other curves are kept under their own names.

  | Canonical | Common source mnemonics |
  |---|---|
  | GR | GR, GRC, SGR |
  | RHOB | RHOB, RHOZ, ZDEN, DEN |
  | NPHI | NPHI, TNPH, NPOR, CNC (limestone units) |
  | DT | DT, DTC, DTCO, AC |
  | DTS | DTS, DTSM |
  | RT (deep) | RT, RD, ILD, LLD, AT90, RLA5 |
  | RM (medium) | ILM, LLM, AT30 |
  | RS (shallow) | SFL, LLS, AT10 |
  | RXO | RXO, MSFL, MCFL |
  | DRHO | DRHO, HDRA, ZCOR |
  | PEF | PEF, PEFZ |
  | CALI | CALI, CAL, HCAL |
  | BS | BS, BIT (or bit size from the header) |
  | SP | SP |

- **Units**: normalised to API, mV, in, g/cm3, v/v, b/e, us/ft and ohm.m (e.g. neutron in percent to v/v, density in kg/m3 to g/cm3, slowness in us/m to us/ft).
- **Unit conflicts are refused, never guessed.** If a header unit disagrees with the data, e.g. density labelled g/cm3 but reading 2300, nothing loads and the message names the curve. Pass `unit_overrides={"RHOB": "kg/m3"}` only with evidence (the report, the task, a vendor convention stated in the file). Record the override in `assumptions`.
- **A curve with no unit** is read in its canonical unit only if its values are plausible. A unitless neutron with values above 1 is read as percent. Each such reading is reported in `warnings`: carry those into `limitations`.
- **Header facts kept**: well name, UWI, datum (from `LMF`) and elevations (`EKB`, `EDF`, `EGL`, converted to m), X/Y and CRS, mud type, Rm, Rmf and Rmc with their temperatures, BHT, bit size.

**What you must supply when the file does not say** (from the task excerpt or report, never assumed):
- `depth_unit`;
- `datum` and `datum_elevation_m` (metres above mean sea level);
- `well_name` (so tables match the well);
- `x`, `y`, `crs`.

**After loading**, read `petro_describe_well`:
- **`missing_for`**: what blocks lithology, porosity, saturation or bad-hole control. Each gap goes in `missing_data` or `limitations`.
- **`datum.tvdss_available`**: false until a trajectory and a datum elevation exist.
- **`aliases_applied`**: report surprising mappings (e.g. a medium induction taken as deep because no deep curve exists).

## 3. Tables: `petro_register_table`

Give a CSV file path, or `csv_text` for small tables from the task. Column names carry units; other units are converted.

| kind | Required columns | Optional |
|---|---|---|
| well_tops | well, top, md_m (or md_ft, tvdss_m, tvdss_ft) | source |
| deviation_survey | md_m (or md_ft), inc_deg, azi_deg | |
| checkshots | md_m / tvdss_m (or _ft), twt_ms (or owt_ms, doubled) | well |
| core_data | md_m (or md_ft), porosity (v/v) or porosity_pct | permeability_md, grain_density_g_cc, sw, sample |
| pressure_data | md_m / tvdss_m (or _ft), pressure_psi (or pressure_bar, pressure_mpa) | mobility_md_cp, quality, well |
| water_analysis | rw_ohmm or salinity_ppm_nacl | temp_c (or temp_f), sample, well |
| temperature_data | md_m / tvdss_m (or _ft), temp_c (or temp_f) | kind, hours_since_circulation, well |
| scal_formation_factor | porosity (or porosity_pct), ff | sample, well |
| scal_resistivity_index | sw, ri | sample, well |
| capillary_pressure | sample, porosity (or porosity_pct), permeability_md, pc_psi (or pc_bar, pc_kpa), sw | system, well |

- **Refusals**: malformed tables are refused (a non-increasing survey, core porosity above 0.6, which is probably percent).
- **Multi-well tables**: tools use only the rows whose `well` matches the loaded well's name. If names differ ("W-1" versus "W1"), reload the well with `well_name` set to match.

## 4. Trajectory and TVDSS: `petro_trajectory`

- **With a survey**: pass the registered `deviation_survey` table. The tool uses minimum curvature, following each leg's arc between stations. A survey that starts below the datum gets a vertical segment above its first station (warned). A log below the last station is extended along the last leg (warned; report it).
- **Without a survey**: use `vertical: true` only when the task or data say the well is vertical. Otherwise list the survey in `missing_data`.
- **TVDSS** = TVD - datum elevation: metres below mean sea level, positive down. With no datum elevation the tool refuses: supply it or report it missing. It is never assumed.
- **The result is a new well file**, with a trajectory. Use its path for all later tools.

## 5. Log QC: `petro_qc_logs`

| Output | Meaning | What you do |
|---|---|---|
| `borehole.washout` | CALI - BS > 1 in | Density, neutron, PEF and shallow resistivity are unreliable there: exclude them from statistics and say so |
| `borehole.density_correction` | DRHO above 0.15 g/cm3 | RHOB unreliable |
| `under_gauge_or_mudcake` | Caliper below bit size | Mudcake: a sign of permeability |
| `spikes_md` | Isolated one-sample spikes | Ignore in statistics; mention only if at a key depth |
| `constant_readings` | A curve flat over 3 m or more | A tool fault or a filled gap: treat as missing |
| `outside_physical_range` | Values that cannot be real | Unit or calibration problem: flag |
| `gaps` | No data | Record intervals without data |
| `invasion` | Deep versus shallow resistivity separation | Evidence of permeable beds; its sign depends on Rmf versus Rw and on hydrocarbons |
| `environment` | Mud type, Rm, Rmf, BHT given or not; environmental corrections | Corrections are "unknown" unless the task or a report says: always state it |

## 6. Interval description: `petro_describe_interval` (your view of the logs)

Call it per zone (`zone` + `tops`) or per MD range. It returns statistics (10th, 50th and 90th percentiles per curve) and located features, in MD and TVDSS:
- **`neutron_density_crossover`**: on the limestone-compatible display, with each interval's mean strength. `strong_crossover_over_0.06_vv` lists the strong ones. A clean water-bearing sandstone shows about 0.03-0.05 v/v on this scale, so strength matters.
- **`neutron_above_density`**: wide separation (shale, bound water).
- **`resistivity_separation`**: deep/shallow ratio beyond 1.5 in either direction.
- **`high_resistivity`**: more than 5 times the interval's 25th percentile.
- **`gamma_ray`**: cleanest and shaliest intervals, *relative to this interval*: these are not clean and shale lines.

Features are observations. Their meaning is an interpretation with alternatives (see `petro-core`, section 5).

## 7. Zones: `petro_zones`

- **Tops**: zones run from each supplied top to the next (tops in MD, or in TVDSS with a trajectory). With a trajectory, zones get TVDSS and true vertical thickness.
- **Tops with no log step** (gamma ray, else density, within ±2 m) are flagged `TOP_WITHOUT_LOG_EXPRESSION`. Report the flag as a contradiction or limitation, citing the top's source. Never move a top: tops belong to stratigraphy.

## 8. Parameters: `petro_parameters`

Register every interpretation parameter before it is used, one set per zone or case:

```json
{"name": "SAND_B base case", "parameters": {
  "rw": {"value": 0.045, "source": "water_sample", "reference": "W2 DST-1 sample, 85 degC"},
  "rw_temp_c": {"value": 85, "source": "water_sample", "reference": "W2 DST-1"},
  "m": {"value": 2.0, "source": "assumption", "low": 1.8, "high": 2.2},
  "rho_ma": {"value": 2.65, "source": "core", "reference": "grain density, 46 plugs"}}}
```

- **Sources**: core, scal, water_sample, log_crossplot, log_sp, log_percentile, pressure_data, test, analogue, regional, task, assumption.
- **Parameter names** also include `nphi_ma`, `rmf`, `rmf_temp_c`, `ssp_mv`, `qv`, `rwb`, `pef_sh`, `overburden_porosity_factor`, `overburden_perm_factor`, `swirr`, `sigma_cos_lab`, `sigma_cos_res`, `brine_density` and `fwl_tvdss_m` (see the property and rock skills for when each is needed).
- **What is refused**: a value with no source, or outside physical ranges. Every assumption is returned in `assumed`, with a request for `low`/`high` if missing.
- **Report the set** in `assumptions`, citing its provenance id. Rw is never a textbook default: without a sample, SP-derived value or proven water zone, it goes to `missing_data`.

## 9. Figures: `petro_render`

- **Kinds**:
  - `composite`: gamma ray and caliper against bit size, resistivity (log scale), density-neutron with crossover shaded, sonic/PEF, tops, MD with TVDSS;
  - `nd_crossplot`: neutron against density with sandstone, limestone and dolomite lines, coloured by gamma ray.
- **You cannot see the image.** Read numbers from the describe tools.
- **When the user wants to see logs**, put `result.markdown` at the top of your final message (it shows inline), with a one-line caption. Always also list it in `conclusions` as `Figure: <caption> — <url>`.

## 10. Exports: `petro_export`

- **Formats**: a loaded well as LAS or CSV (MD, plus TVDSS when a trajectory exists, and the chosen curves).
- **Delivery**: give the user `download_url` in `conclusions` as `Download: <file> — <url>`.
