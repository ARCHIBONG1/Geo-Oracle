---
name: petro-data-qc
description: Getting well data in and checking it. It covers importing uploads, loading LAS/CSV logs (mnemonics, units, unit conflicts, header facts, datums) and registering tables (tops, deviation survey, checkshots, core, pressures, water analyses, temperatures). It also covers trajectories and TVDSS, log QC, reading interval descriptions, zones from tops, registering parameters with sources, and figures. Load before loading or quality-controlling any well data.
---

# Well data and quality control

## 1. Getting files in

| Where the file is | Do |
|---|---|
| Named by a path in the shared inputs folder (`operations/inputs`) | Nothing: use the path |
| Uploaded in your chat (you see it under `/opt/tf/uploads/`) | `petro_import_upload(sandbox_path="/opt/tf/uploads/W1.las")` |
| Uploaded in Geo Oracle's chat (the task says so) | `petro_import_upload(sandbox_path=..., source_agent="geo-oracle")` |

The import returns a path such as `uploads/W1.las`. Use it from then on. The import refuses a file that is not in any recent sandbox. Then ask, via `missing_data`, for the file to be uploaded again or copied into `operations/inputs`.

## 2. Loading logs: `petro_load_well`

`petro_prepare_wells` handles several wells in one call: it loads and locates them, and publishes `correlation_logs` and `well_tops`. It locates each well by a survey, a track, or `vertical: true` where the task says so. Use it first for any multi-well task, then QC each well as below.

The loader reads LAS 2.0/3.0 or CSV logs. It does not support DLIS: ask for LAS.

**What it does:**
- **Depth**: converted to metres MD (the loader detects feet from the header). The index must be MD. The loader refuses a file indexed in TVD or TVDSS.
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

- **Units**: normalised to API, mV, in, g/cm3, v/v, b/e, us/ft and ohm.m. For example: neutron in percent to v/v, density in kg/m3 to g/cm3, slowness in us/m to us/ft.
- **Repeated mnemonics**: if a file lists a curve twice (for example two `DEPTH` curves), the loader recognises the base name. It drops a repeat of the depth index, and says so in `warnings`.
- **Neutron scale**: limestone-scale neutron (`NPHI`, `TNPH`, `NPHILS`) is preferred. If only a sandstone-scale neutron (`NPHISS`) exists, it becomes NPHI with a warning. Then set `nphi_ma` for that scale (0 for quartz). Read crossover displays with care, because they are drawn on the limestone scale.
- **Unit conflicts are refused, never guessed.** If a header unit disagrees with the data (for example, density labelled g/cm3 but reading 2300), nothing loads, and the message names the curve. Pass `unit_overrides={"RHOB": "kg/m3"}` only with evidence (the report, the task, a vendor convention stated in the file). Record the override in `assumptions`.
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

Give a CSV file path, or `csv_text` for small tables from the task. Column names carry units. The tool converts other units.

| kind | Required columns | Optional |
|---|---|---|
| well_tops | well, top, md_m (or md_ft, tvdss_m, tvdss_ft) | source |
| deviation_survey | md_m (or md_ft), inc_deg, azi_deg | |
| checkshots | md_m / tvdss_m (or _ft), twt_ms (or owt_ms, doubled) | well |
| core_data | md_m (or md_ft), porosity (v/v) or porosity_pct | permeability_md, grain_density_g_cc, sw, sample |
| pressure_data | md_m / tvdss_m (or _ft), pressure_psi (or pressure_bar, pressure_mpa) | mobility_md_cp, quality, well |
| water_analysis | rw_ohmm or salinity_ppm_nacl | temp_c (or temp_f), sample, well |
| temperature_data | md_m / tvdss_m (or _ft), temp_c (or temp_f) | kind, hours_since_circulation, well |
| well_track | md_m (or md_ft), tvdss_m or tvd_m (or _ft) | x, y, well |
| scal_formation_factor | porosity (or porosity_pct), ff | sample, well |
| scal_resistivity_index | sw, ri | sample, well |
| capillary_pressure | sample, porosity (or porosity_pct), permeability_md, pc_psi (or pc_bar, pc_kpa), sw | system, well |

- **Layout**: the tool detects comma, tab, semicolon or whitespace delimiters. Lines that start with `#` are comments.
- **Vendor column names**: map them with `columns`. The unit in the canonical name drives the conversion, for example `{"MD (ft)": "md_ft", "TVDSS (ft)": "tvdss_ft"}`. Report the mapping in `assumptions`.
  - **Times must be in milliseconds**: `twt_ms` (two-way) or `owt_ms` (one-way, doubled on registration). A mapping cannot convert seconds: ask for milliseconds.
- **Refusals**: the tool refuses malformed tables (a non-increasing survey, core porosity above 0.6, which is probably percent).
- **Multi-well tables**: tools use only the rows whose `well` matches the loaded well's name. If names differ ("W-1" versus "W1"), reload the well with `well_name` set to match.

## 4. Trajectory and TVDSS: `petro_trajectory`

- **With a track** (`track`, a registered `well_track` table: MD with TVDSS or TVD, optional X/Y, as most interpretation software exports):
  - TVDSS and X/Y are interpolated along the track.
  - If the track gives TVDSS and its first leg is vertical, the tool **derives** the datum elevation from the data (MD - TVDSS at the top point). The result gives it. Otherwise supply it.
  - If the header's datum elevation disagrees with the track's, the tool warns: report the disagreement.
- **With a survey**: pass the registered `deviation_survey` table. The tool uses minimum curvature, and follows each leg's arc between stations. If a survey starts below the datum, the tool adds a vertical segment above its first station (warned). If a log runs below the last station, the tool extends it along the last leg (warned; report it).
- **Without a survey**: use `vertical: true` only when the task or data say the well is vertical. Otherwise list the survey in `missing_data`.
- **TVDSS** = TVD - datum elevation: metres below mean sea level, positive down. If there is no datum elevation, the tool refuses: supply it or report it missing. It is never assumed.
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
- **`gamma_ray`**: cleanest and shaliest intervals, *relative to this interval*. These are not clean and shale lines.

Features are observations. Their meaning is an interpretation with alternatives (see `petro-core`, section 5).

## 7. Zones: `petro_zones`

- **Tops**: zones run from each supplied top to the next (tops in MD, or in TVDSS with a trajectory). With a trajectory, zones get TVDSS and true vertical thickness.
- **Tops with no log step** (gamma ray, else density, within ±2 m): the tool flags them `TOP_WITHOUT_LOG_EXPRESSION`. Report the flag as a contradiction or limitation, and cite the top's source. Never move a top: tops belong to stratigraphy.

## 8. Parameters: `petro_parameters`

Register every interpretation parameter before you use it, one set per zone or case:

```json
{"name": "SAND_B base case", "parameters": {
  "rw": {"value": 0.045, "source": "water_sample", "reference": "W2 DST-1 sample, 85 degC"},
  "rw_temp_c": {"value": 85, "source": "water_sample", "reference": "W2 DST-1"},
  "m": {"value": 2.0, "source": "assumption", "low": 1.8, "high": 2.2},
  "rho_ma": {"value": 2.65, "source": "core", "reference": "grain density, 46 plugs"}}}
```

- **Sources**: core, scal, water_sample, log_crossplot, log_sp, log_percentile, pressure_data, test, analogue, regional, task, assumption.
- **Parameter names** also include `nphi_ma`, `rmf`, `rmf_temp_c`, `ssp_mv`, `qv`, `m_star`, `n_star`, `rwb`, `nd_gas_tolerance`, `sonic_c`, `pef_sh`, `overburden_porosity_factor`, `overburden_perm_factor`, `swirr`, `sigma_cos_lab`, `sigma_cos_res`, `brine_density`, `fwl_tvdss_m`, `srd_elevation_m`, `source_offset_m`, `salinity_ppm`, `oil_api`, `gor_l_per_l`, `gas_gravity`, `reservoir_pressure_mpa`, `reservoir_temp_c` and `circulation_hours` (see the property, rock and physics skills for when each is needed).
- **What is refused**: a value with no source, or a value outside physical ranges. The result lists every assumption in `assumed`, and asks for `low`/`high` where they are missing.
- **Report the set** in `assumptions`, and cite its provenance id. Rw is never a textbook default. Without a sample, an SP-derived value or a proven water zone, put Rw in `missing_data`.

## 9. Figures: `petro_render`

- **Kinds**:
  - `composite`: gamma ray and caliper against bit size, resistivity (log scale), density-neutron with crossover shaded, sonic/PEF, tops, MD with TVDSS;
  - `nd_crossplot`: neutron against density with sandstone, limestone and dolomite lines, coloured by gamma ray;
  - `pickett` and `poro_perm`: see `petro-saturation` and `petro-permeability`;
  - `pressure_depth`: pressures, fitted gradients, contacts and hydrostatic from a `pressure_profile` product (see `petro-pressure-temperature`).
- **Tools that draw their own figure** return it in `result.figures`: `petro_porosity`, `petro_saturation`, `petro_net_pay`, `petro_rw` (pickett), `petro_permeability` (core_transform) and `petro_pressure`.
- **You cannot see the image.** Read numbers from the describe tools.
- **Publish a composite log of each well a claim rests on**, and at least of the most representative well. A user request can add figures (other wells, intervals, crossplots). It is never the condition for the standard ones.
- **For each figure**, put its `markdown` line and a one-line caption at the top of your final message. The chat shows it inline. Also add it to `figures` and list it in `conclusions` as `Figure: <caption> — <url> — supports <claim ids>`.

## 10. Exports: `petro_export`

- **Formats**: a loaded well as LAS or CSV (MD, plus TVDSS when a trajectory exists, and the chosen curves).
- **Delivery**: give the user `download_url` in `conclusions` as `Download: <file> — <url>`.

<!-- end of skill: petro-data-qc -->
