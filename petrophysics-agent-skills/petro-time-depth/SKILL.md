---
name: petro-time-depth
description: Time-depth relations and the seismic handoff - checkshots, datums (SRD), offset correction, sonic integration with drift correction, the time_depth and well_header products, and what a well tie with them should show. Load before petro_time_depth or petro_well_header, and whenever the seismic specialist needs a tie.
---

# Time-depth and the seismic handoff

## Datums first

Datum mismatches are the commonest cause of bad ties.
- **SRD**: agree the **seismic reference datum** with the seismic specialist, via Geo Oracle, and register `srd_elevation_m` (metres above MSL; 0 when the SRD is MSL) with its source.
- **Product depths** are metres **below the SRD**: depth_m = TVDSS + srd_elevation_m.
- **Time referenced to another datum.** A relation referenced to the SRD gives 0 ms at depth 0.
  - `petro_time_depth` warns when the checkshot curve does not (`twt_at_srd_ms_before_shift`). This is common for models timed from the rig floor, or with an interpolated shallow section.
  - Do not guess the correction. Ask, via Geo Oracle, for the seismic seabed check (below), then apply the difference as the parameter `td_time_shift_ms` (source `task`, reference the seismic evidence id). The tool reports the shift and drops points above the SRD.
- **Checkshot times** are taken as two-way times from the SRD, as supplied. If the survey states another reference (e.g. ground level, KB), convert them before registering, and say so. The tool records this assumption in the sidecar.

## `petro_time_depth`

**Needs**: a well with a trajectory, a registered `checkshots` table (md_m or tvdss_m; twt_ms or owt_ms, doubled on registration) and the parameter set.

- **Offset**: with `source_offset_m`, times are corrected to vertical incidence (straight rays).
- **`calibrate_sonic: true`** (default): the sonic is integrated from the log top, anchored on the checkshot curve there. A piecewise-linear drift fixed at every checkshot is then added. The result honours each checkshot and keeps the sonic's detail between them.
  - `drift_at_checkshots_ms` shows how far the raw sonic was off.
  - A few ms over hundreds of metres is normal: the sonic is higher-frequency, so it reads slower than seismic.
  - Above about 10 ms (warned), check for sonic cycle skips, washouts and wrong checkshot picks.
- **`calibrate_sonic: false`** uses the checkshots alone, linear between stations. This is right for a **dense time-depth model** (e.g. from a velocity survey or VSP, hundreds of points), which already holds the detail. Results then list about 20 coarse interval velocities and a summary.
- **Points outside the logged interval** (e.g. a time-depth model from the surface) are placed by extending the trajectory's end segment, which is exact for a straight hole, and the tool says how many.
- **Outputs**: a `time_depth` product (CSV twt_ms, depth_m, the seismic format), and a new well file with a `TWT` curve.

## A Vp and density product for a first tie

`petro_elastic_logs(vs_source="none")` publishes depth, Vp and density only. It needs just DT, RHOB, a trajectory and `srd_elevation_m`: no shale volume, porosity or fluid parameters. The logs are used as measured, so check `petro_qc_logs` first: washouts and spikes make false reflections. Use the full elastic product (with Vs, substitution and Backus) when AVO or fluid cases are needed.

## `petro_well_header`

Publishes a `well_header` (X/Y, CRS, datums, SRD, TD, bottom-hole X/Y). The seismic specialist converts X/Y to inline/crossline. A missing X/Y or CRS goes to `missing_data`.

## Suspect stations

`suspect_stations` lists checkshot intervals whose velocity disagrees with the sonic over the same interval by more than 20-25%. A single station far off (e.g. 2,700 m/s where the sonic reads 4,500 m/s in carbonates) is usually a wrong pick or a typo in the source table.
- Report it as an observation with both velocities.
- Rerun with `exclude_stations` for that station, say so in `assumptions`, and compare the drift before and after.
- Never fix a time by hand.

## Marker times

Pass the registered tops as `tops`: the result's `tops_twt` and the `tops_time` product give each marker's two-way time from this relation. The seismic specialist compares them with the events on the trace (`seismic_trace_events`) and with the synthetic tie.

## What the seismic tie should show

With a checkshot-calibrated time_depth and in-situ elastic_logs:
- **Bulk shift**: within a sample or two.
- **Phase**: near the wavelet's.
- **Correlation**: high.

A large bulk shift points at the SRD or checkshot reference. A shift growing with depth points at an uncalibrated sonic or wrong checkshots. When the seismic specialist reports either, reanalyse: check datums first.

## Reporting

- **Conclusions**: `Product: time_depth — <ref>`, `Product: elastic_logs — <ref>` and `Product: well_header — <ref>`, each from `result.products[].conclusion_line`.
- **Measurements**: the SRD, checkshot count and depth range, the maximum drift corrected, and interval velocities where relevant.
