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

`petro_prepare_wells` handles several wells in one call: it loads and locates them, and publishes `correlation_logs` and `well_tops`. It locates each well by a survey, a track, or `vertical: true` where the task says so. Use it first for any multi-well task, then QC each well as below. Its reply has a provenance id. Its `warnings` carry every load, unit and trajectory warning, prefixed with the file path: carry them into `limitations`.

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
  | CALI | CALI, CAL, HCAL (C1 only with a length unit: in mud-gas files C1 is methane) |
  | BS | BS, BIT (or bit size from the header) |
  | SP | SP |
  | K, TH, U | POTA/K/HFK (%), THOR/TH/HTHO (ppm), URAN/U/HURA (ppm): spectral gamma ray |
  | CGR | CGR, HCGR, GRKT (uranium-free GR; never used as GR) |

- **Units**: normalised to API, mV, in, g/cm3, v/v, b/e, us/ft and ohm.m. For example: neutron in percent to v/v, density in kg/m3 to g/cm3, slowness in us/m to us/ft.
- **Repeated mnemonics**: if a file lists a curve twice (for example two `DEPTH` curves), the loader recognises the base name. It drops a repeat of the depth index, and says so in `warnings`.
- **Neutron scale**: limestone-scale neutron (`NPHI`, `TNPH`, `NPHILS`) is preferred. The loader records the NPHI matrix scale from the mnemonic (NPHISS sandstone, NPHILS limestone) or the curve description ("sandstone matrix", "limestone units"). A sandstone or dolomite scale is warned: set `nphi_ma` for that scale (0 for quartz on a sandstone scale). `matrix_id` and `multimineral` refuse it. A scale that is not recorded is unknown: confirm it from the header or the report. Read crossover displays with care, because they are drawn on the limestone scale.
- **Unit conflicts are refused, never guessed.** If a header unit disagrees with the data, nothing loads, and the message names the curve. Examples: density labelled g/cm3 but reading 2300; NPHI labelled % or PU with a median below 1 PU (the values look like v/v); DT read as us/ft with a median above 200 us/ft (slower than water: the values look like us/m). Pass `unit_overrides={"RHOB": "kg/m3"}` only with evidence (the report, the task, a vendor convention stated in the file). Record the override in `assumptions`.
  - A unit in `unit_overrides` also confirms a unit that the data seem to contradict. This applies to NPHI in % or PU with a median below 1 PU, and to DT in us/ft with a median above 200. The well then loads with a warning: confirm the unit. Values outside the plausible range are still refused.
- **A curve with no unit** is read in its canonical unit only if its values are plausible. A unitless neutron with values above 1 is read as percent. A spectral curve (K, TH, U, CGR) whose unit or values fit no unit keeps its own name, with family `other`, and a warning. The well still loads. It becomes `<MNEMONIC>_FILE` only when another mnemonic (for example POTA) was read as that curve. A bare K or U with no unit can be another quantity. It is read as potassium or uranium only when the file has a thorium curve, or when `unit_overrides` states its unit. Each such reading is reported in `warnings`: carry those into `limitations`.
- **CSV nulls**: -999.25, -999, -9999 and -9999.25 are read as no data (warned).
- **Header facts kept**: well name, UWI, datum (from `LMF`) and elevations (`EKB`, `EDF`, `EGL`, converted to m), X/Y and CRS, mud type, Rm, Rmf and Rmc with their temperatures, BHT, bit size. The Rw and temperature tools read the header Rmf and BHT units. A blank or unknown temperature unit is refused there: then register `rmf_temp_c`, or `surface_temp_c` and `temperature_gradient_c_per_km`. A header bit size with no unit is read as inches only between 3 and 30 (warned); otherwise no BS curve is made.
- **Mud-gas curves** (for example C1, H2) are kept under their own names. No petrophysics tool interprets them. A hydrogen reading in mud gas is an observation, not proof of natural hydrogen. Drilling can make it: bit heating with PDC or diamond bits cracks the drilling fluid, and CO, ethene and propene mark that gas. Steel in the well can also react with water (Zgonnik 2020). Gas chromatographs that use hydrogen as carrier gas cannot detect it. State the detector, the bit type and whether helium was measured. Report the reading as an observation with these alternatives.

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
| deviation_survey | md_m (or md_ft), inc_deg (or inclination_deg, incl_deg, inc), azi_deg (or azimuth_grid_deg, azimuth_deg, azimuth_true_deg, azi; read as given, warned) | |
| checkshots | md_m / tvdss_m (or _ft), twt_ms (or owt_ms, doubled) | well |
| core_data | md_m (or md_ft), porosity (v/v) or porosity_pct | permeability_md, grain_density_g_cc, sw, sample (rows with a blank depth or porosity are skipped, with a note) |
| pressure_data | md_m / tvdss_m (or _ft), pressure_psi (or pressure_bar, pressure_mpa) | mobility_md_cp, quality, well, test (LOT, XLOT, FIT, DFIT, PIT, SRT, minifrac); text columns type, kind, test_type, note, notes, comment, comments, remarks |
| water_analysis | rw_ohmm or salinity_ppm_nacl | temp_c (or temp_f), sample, well |
| temperature_data | md_m / tvdss_m (or _ft), temp_c (or temp_f) | kind, hours_since_circulation, well |
| well_track | md_m (or md_ft), tvdss_m or tvd_m (or _ft) | x, y, well |
| scal_formation_factor | porosity (or porosity_pct), ff | sample, well |
| scal_resistivity_index | sw, ri | sample, well |
| capillary_pressure | sample, pc_psi (or pc_bar, pc_kpa), one of sw, s_hg, snw (non-wetting fraction) or s_hg_pct, snw_pct (percent of pore volume) | porosity (or porosity_pct) and permeability_md (Leverett J needs both), system, well, md_m (or md_ft) |
| dip_picks | md_m (or md_ft), dip_deg, dip_azimuth_deg (or azimuth_deg) | type, quality, well, structural_dip_removed |
| well_tests | md_top_m and md_base_m (or _ft; or one depth md_m / tvdss_m), fluid or recovery | well, test, formation, interpretation, pressure_psi (or _bar, _mpa); text kept as text |

- **Layout**: the tool detects comma, tab, semicolon or whitespace delimiters. Lines that start with `#` are comments.
- **Vendor column names**: map them with `columns`. The unit in the canonical name drives the conversion, for example `{"MD (ft)": "md_ft", "TVDSS (ft)": "tvdss_ft"}`. Report the mapping in `assumptions`.
  - **Times must be in milliseconds**: `twt_ms` (two-way) or `owt_ms` (one-way, doubled on registration). A mapping cannot convert seconds: ask for milliseconds.
- **Refusals**: the tool refuses malformed tables (a non-increasing survey, core porosity above 0.6, which is probably percent). A refusal for a missing column proposes a `columns` mapping: check it, then retry with it.
- **Test summaries** (DST, RFT, MDT, production tests): register them as `well_tests`. The result gives the rows in `rows_preview`. Read each recovery: formation water with no free hydrocarbon marks a proven water leg (`petro-saturation`, Pickett). A `pressure_data` refusal for such a table names `well_tests`. RFT and MDT pressure points stay `pressure_data`: `well_tests` is for recovery and test summaries.
- **A table that fits no kind**: read it with `petro_preview_table` (columns and up to 50 rows, as written). It registers nothing.
- **Multi-well tables**: tools use only the rows whose `well` matches the loaded well's name. If names differ ("W-1" versus "W1"), reload the well with `well_name` set to match.

## 4. Trajectory and TVDSS: `petro_trajectory`

- **With a track** (`track`, a registered `well_track` table: MD with TVDSS or TVD, optional X/Y, as most interpretation software exports):
  - TVDSS and X/Y are interpolated along the track.
  - If the track gives TVDSS and its first leg is vertical, the tool **derives** the datum elevation from the data (MD - TVDSS at the top point). The result gives it. Otherwise supply it.
  - If the header's datum elevation disagrees with the track's, the tool warns: report the disagreement.
- **With a survey**: pass the registered `deviation_survey` table. The tool uses minimum curvature, and follows each leg's arc between stations. If a survey starts below 0 m MD, the tool adds a vertical tie-in station at 0 m MD (warned). If the first station is inclined, the tool builds along one arc from vertical (warned): give the tie-in station if it is known. If a log runs below the last station, the tool extends it along the last leg (warned; report it).
- **Without a survey**: use `vertical: true` only when the task or data say the well is vertical. Otherwise list the survey in `missing_data`.
- **Publishing without a trajectory**: `petro_publish_correlation_logs` treats the well as vertical (datum offset only) and warns. Carry the warning into `limitations`.
- **TVDSS** = TVD - datum elevation: metres below mean sea level, positive down. If there is no datum elevation, the tool refuses: supply it or report it missing. It is never assumed.
- **The result is a new well file**, with a trajectory. Use its path for all later tools.
- **Products published before the trajectory** (by `petro_publish_correlation_logs` or `petro_publish_table`) are published again with it. The result gives the new references in `products` and the old ones in `superseded`. Never use or pass on a superseded reference. Other products with an old TVDSS are in `stale_products`: run their tools again.

## 5. Log QC: `petro_qc_logs`

| Output | Meaning | What you do |
|---|---|---|
| `borehole.washout` | CALI - BS > 1 in | Density, neutron, PEF and shallow resistivity are unreliable there: exclude them from statistics and say so |
| `borehole.probable_bit_size_change` | BS from one header value (`SINGLE_HEADER_BIT_SIZE`), and CALI above it by more than the washout limit over 100 m or more | Probably another bit size, not a washout: these runs are not in `washout`. Ask for the bit sizes in `missing_data` |
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
- **Outside the trajectory**: the trajectory is stored on the log grid. So `tvdss_extrapolated` (`TVDSS_EXTRAPOLATED`) means outside the logged interval: TVDSS there extends the end leg. A top given in TVDSS outside the logged interval is refused: give it in MD. A zone whose base is above its top is flagged `ZONE_BASE_ABOVE_TOP`. Report each flag.
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
- **Parameter names**: the list below adds to the names above. The property, rock and physics skills tell when each is necessary.
  - Logs and shale: `nphi_ma`, `nd_gas_tolerance`, `sonic_c`, `pef_sh`, `sp_shale_mv`, `phi_sand_clean`, `phi_shale`.
  - Rw and saturation: `rmf`, `rmf_temp_c`, `ssp_mv`, `qv`, `m_star`, `n_star`, `rwb`, `swirr`, `salinity_ppm`.
  - Core and cutoffs: `overburden_porosity_factor`, `overburden_perm_factor`, `k_cutoff_md`.
  - Capillary pressure: `sigma_cos_lab` (or `sigma_cos_air_mercury`, `sigma_cos_gas_brine`, `sigma_cos_oil_brine`, `sigma_cos_gas_oil`), `sigma_cos_res`, `buoyant_fluid_density`, `brine_density`, `fwl_tvdss_m`.
  - Pressure, temperature and thermal: `hydrostatic_reference_elevation_m`, `ground_elevation_m`, `tc_shale_w_mk`, `circulation_hours`.
  - Time-depth: `srd_elevation_m`, `source_offset_m`.
  - Fluids: `oil_api`, `gor_l_per_l`, `gas_gravity`, `reservoir_pressure_mpa`, `reservoir_temp_c`.
- **What is refused**: a value with no source, or a value, `low` or `high` outside physical ranges. The result lists every assumption in `assumed`, and asks for `low`/`high` where they are missing.
- **What is warned, not refused**: a `temperature_gradient_c_per_km` above 150 (a geothermal or convective setting only), and a `brine_density` below 0.9 g/cm3 without `reservoir_temp_c` of 200 degC or more. Confirm the value and the unit. The `hydrocarbon_density` floor is 0.001 g/cm3, so a hydrogen density is accepted.
- **Report the set** in `assumptions`, and cite its provenance id. Rw is never a textbook default. Without a sample, an SP-derived value, a proven water zone, a regional value or a value the task gives, put Rw in `missing_data`.

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
