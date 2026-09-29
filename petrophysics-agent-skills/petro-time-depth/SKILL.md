---
name: petro-time-depth
description: Time-depth relations and the seismic handoff - checkshots, datums (SRD), offset correction, sonic integration with drift correction, the time_depth and well_header products, and what a well tie with them should show. Load before petro_time_depth or petro_well_header, and whenever the seismic specialist needs a tie.
---

# Time-depth and the seismic handoff

## Datums first

Datum mismatches are the commonest cause of bad ties.
- **SRD**: agree the **seismic reference datum** with the seismic specialist, via Geo Oracle, and register `srd_elevation_m` (metres above MSL; 0 when the SRD is MSL) with its source.
- **Product depths** are metres **below the SRD**: depth_m = TVDSS + srd_elevation_m.
- **Checkshot times** are taken as two-way times from the SRD, as supplied. If the survey states another reference (e.g. ground level, KB), convert them before registering, and say so. The tool records this assumption in the sidecar.

## `petro_time_depth`

**Needs**: a well with a trajectory, a registered `checkshots` table (md_m or tvdss_m; twt_ms or owt_ms, doubled on registration) and the parameter set.

- **Offset**: with `source_offset_m`, times are corrected to vertical incidence (straight rays).
- **`calibrate_sonic: true`** (default): the sonic is integrated from the log top, anchored on the checkshot curve there. A piecewise-linear drift fixed at every checkshot is then added. The result honours each checkshot and keeps the sonic's detail between them.
  - `drift_at_checkshots_ms` shows how far the raw sonic was off.
  - A few ms over hundreds of metres is normal: the sonic is higher-frequency, so it reads slower than seismic.
  - Above about 10 ms (warned), check for sonic cycle skips, washouts and wrong checkshot picks.
- **`calibrate_sonic: false`** uses the checkshots alone, linear between stations.
- **Outputs**: a `time_depth` product (CSV twt_ms, depth_m, the seismic format), and a new well file with a `TWT` curve.

## `petro_well_header`

Publishes a `well_header` (X/Y, CRS, datums, SRD, TD, bottom-hole X/Y). The seismic specialist converts X/Y to inline/crossline. A missing X/Y or CRS goes to `missing_data`.

## What the seismic tie should show

With a checkshot-calibrated time_depth and in-situ elastic_logs:
- **Bulk shift**: within a sample or two.
- **Phase**: near the wavelet's.
- **Correlation**: high.

A large bulk shift points at the SRD or checkshot reference. A shift growing with depth points at an uncalibrated sonic or wrong checkshots. When the seismic specialist reports either, reanalyse: check datums first.

## Reporting

- **Conclusions**: `Product: time_depth — <ref>`, `Product: elastic_logs — <ref>` and `Product: well_header — <ref>`, each from `result.products[].conclusion_line`.
- **Measurements**: the SRD, checkshot count and depth range, the maximum drift corrected, and interval velocities where relevant.
