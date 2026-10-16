---
name: seismic-structural
description: Structural interpretation from seismic - horizon tracking, faults and throw, structure and isochron maps, closure and spill point, time-to-depth conversion - with the procedures, traps and uncertainty rules for each tool. Load for questions about faults, traps, deformation, structure maps or depth.
---

# Structural interpretation

## Tools

| Task | Tool | Key outputs |
|---|---|---|
| Choose a horizon seed | `seismic_list_events` | peaks/troughs with sub-sample z, amplitude, strength relative to the trace |
| Track a horizon | `seismic_track_horizon` (job) | horizon map, coverage, correlation quality, why tracking stopped |
| Grid supplied picks | `seismic_horizon_map` operation `from_picks` | horizon map from a horizon_picks table |
| Describe a structure map | `seismic_horizon_map` operation `structure` | shallowest and deepest points with locations, tile means, gradient |
| Isochron | `seismic_horizon_map` operation `isochron` | thickness map (ms in time) |
| Faults and the fault volume (the standard chain in one call) | `seismic_interpret_faults` (job) | labelled (0 = no fault, 1 = F1, 2 = F2, ...) or binary fault volume, fault set, likelihood volume, `qc`, per fault: extent, strike, apparent dip, `support`; figure |
| The same chain step by step | `seismic_fault_likelihood` (job) -> `seismic_extract_faults` -> `seismic_binarize_volume` with the fault set | likelihood volume; fault set; 0/1 or labelled volume, threshold and its source |
| Throw | `seismic_fault_throw` | throw along the fault, downthrown side, cycle-skip check, horizon offsets |
| Faults when likelihood is poor; channels, karst, geobodies | `seismic_segment_volume` (see `seismic-attribute-interpretation`) | labelled or binary volume, body catalogue, extraction figure |
| 3D view of faults and horizons | `seismic_render_3d` | inline image plus interactive link (see `seismic-visualisation`) |
| Closure | `seismic_closure` | crest, spill point and type, column height, area |
| Depth | `seismic_time_to_depth` | depth map or depth volume with the stated velocity model |
| Sections, attributes | `seismic_describe_section`, edge and dip attributes | breaks, continuity, dip, as before |
| Figures | `seismic_render_map`, `seismic_render_section` | for people |

## Horizon procedure

1. Choose the event at a well, crest or clear location with `seismic_list_events`. Give the event kind (peak or trough) and z from its result; never pick z from memory or a description.
2. Before tracking across a faulted area, extract the faults (`seismic_interpret_faults`). Tracking does not jump faults: put at least one seed in each fault block.
3. Run `seismic_track_horizon`. Check `coverage`, `correlation_p10` and `rejected_candidates`:
   - coverage below 1 is expected at faults, polarity changes and fading reflectors; say where the gaps are (`seismic_horizon_map` structure tile means show undefined tiles);
   - `low_correlation` stops mean the event changes character; do not lower `min_correlation` just to fill the map without saying so.
4. A polarity change along a horizon (for example a soft gas sand becoming a weak hard brine sand) stops a peak or trough tracker. That is i## Fault procedure

1. `seismic_interpret_faults` on the seismic amplitude volume, cropped with `z_range` to the interval with reflectors (quiet intervals carry no fault information). It runs the whole chain in a fixed order: fault likelihood, its QC, 3D extraction of steep planes, and each fault drawn as a surface. Use it whenever faults or a fault volume are asked for. `output: "labels"` (default) gives 0 = no fault, 1 = F1, 2 = F2, ...; `output: "binary"` gives 1 = fault. It also writes the fault set (for `seismic_fault_throw`, `seismic_publish_structure` and `seismic_render_3d`) and the likelihood volume.
   - Read `result.qc` first. If it gives `suggested_max_dip_samples`, reflectors dip beyond the correlation's lag: rerun with that `max_dip_samples`. If its notes say few measured voxels reach the threshold, the input may not be reflection amplitudes, or the interval holds no faults at this scale: check the input (`seismic_describe_volume` shows its lineage) before changing any threshold.
   - If likelihood still looks poor (low `measured_fraction`, low `separability.otsu_eta`, or no sensible faults where sections show sharp breaks), also run the `fault_attribute` recipe (`seismic-attribute-interpretation`) and compare the two. An approach the user asked for explicitly is run as asked.
   - The likelihood shows every lateral discontinuity (faults, channel edges, chaotic facies); the faults are only the steep planes it supports over a large depth range. Never call a discontinuity a fault just because it is bright in the likelihood.
   - Each fault carries `support`: `measured`, or `mostly_interpolated` when under a fifth of its surface rests on measured likelihood. Report a mostly interpolated fault as weakly supported.
   - **Persistence.** The tool also looks for the faults at the eight settings around the one asked for (threshold and `min_z_extent` each x0.8 and x1.25) and draws only those found at two thirds of them (`min_persistence`). Each fault carries `found_at` (e.g. `9/9`); `result.persistence` gives the count at every setting and lists the less persistent faults. Report the drawn faults with that table. A count that changes across settings is reported as unstable, with the table: **never change settings, or keep a subset of faults, to reach a number anyone expects**, the person included. Their expected count is a hypothesis you test and report on.
   - Report each fault's `apparent_dip` (traces per sample, in index units: true dip needs velocity and trace spacing) and z range. Positions are uncertain by about one trace.
   - "No faults found at this threshold" is not "no faults": report the threshold (default 0.3) and the qc notes. If sections show sharp breaks, retry once with `threshold: 0.2` or a smaller `min_z_extent` and report both runs.
2. **Step by step**, only for a step the single tool does not expose (`method: "map"` for vertical faults in legacy workflows, keeping chosen faults with `fault_ids` and a stated `selection_reason`, `fault_geometry: "voxels"`): `seismic_fault_likelihood` on the seismic, then `seismic_extract_faults` on the likelihood, then `seismic_binarize_volume` on that likelihood with the fault set. The tools refuse any other order: likelihood on an attribute such as coherence, extraction from the seismic, thresholding likelihood without a fault set (that gives blobs, not faults), or a fault set drawn on a volume it was not extracted from. `override_reason` exists only for an input the user explicitly asked for; put the reason in `limitations`.
3. Confirm each fault that matters on at least one section with `seismic_describe_section` (sharpest_lateral_breaks at the same position).
4. `seismic_fault_throw` with horizons tracked on both sides where possible. Correlation throw and horizon throw should agree; if `possible_cycle_skip_fraction` is high, the correlation throw may be off by one period and only the horizon throw counts.
5. **Delivery**: show `result.faults_figure_markdown`; give the volume to the user with `seismic_export_volume` (in their axis order if they supplied an array); view it with `seismic_render_slice` or `seismic_render_3d`.
6. Alternatives to consider every time: channel edge, chaotic facies boundary, acquisition footprint (aligned with inline or crossline, regular spacing), migration artefact near steep dip, velocity pull-up or push-down.

sline, regular spacing), migration artefact near steep dip, velocity pull-up or push-down.

## Closure and depth

1. `seismic_closure` on a structure map gives four-way closures only. For each: crest, spill z and location, spill type, column height, area.
   - `saddle`: the closure spills into a neighbouring culmination; the spill point location is part of the answer.
   - `data_edge`: the closure reaches the edge of the mapped data; the true closure may be larger or fault-bounded. Report the numbers as minimum values.
   - Fault-dependent closure is not computed: state it as a limitation when a fault bounds the structure.
2. A closure in time always carries the velocity warning. Convert with `seismic_time_to_depth` using a supplied time-depth table or a stated velocity model, then run `seismic_closure` on the depth map. Report both, and whether the closure survives conversion.
3. The velocity model is an input: name its source (task, well W1 checkshot, wells_petrophysics finding). A constant or linear velocity is an **assumption** unless supplied.
4. Closure is not a hydrocarbon column. A spill point bounds the trap; the fluid contact needs other evidence (see `seismic-qi`).

## Publishing for structural_geology

When another specialist needs your faults and horizons — structural_geology above all — publish them with `seismic_publish_structure`: fault traces in X/Y with z ranges and, given a time-depth table or a depth volume, strike and dip; horizon grids with inline, crossline, x, y, z. The sidecar states the domain and the pick uncertainty, and structural_geology's tools refuse a time-domain product, so convert first where depth is needed. List each reference in `conclusions` as `Product: fault_set — @seismic-interpreter-agent/products/<file>` (likewise `horizon`). A task that feeds a structural model, a closure with faults, fault stability or fault seal is incomplete until these are published.

## Classification

- Tracked horizon values, map extremes, throw, closure numbers and depths are **measurements** (they depend on stated parameters and inputs).
- "Fault F1 is a normal fault downthrown to the east" is an **interpretation**.
- "The structure is a four-way closed trap of 0.28 km²" is an **interpretation** resting on the closure measurement plus the depth conversion.
- Trap validity (seal, charge, timing) is outside seismic geometry: an **inference** at most, needing other specialists.

## Uncertainty to report

- Horizon: tracking quality, coverage gaps and their causes, event choice (peak/trough) and phase.
- Faults: position ±1 trace and the lateral resolution; throw in time is apparent; throw below tuning thickness is unresolved.
- Depth: the velocity model and its source; for a closure, how sensitive spill and column are to it.
- Time data: geometry, dips and closures are apparent until converted.
- Structural claims usually need an independent line (well dips, regional structure from the structural_geology specialist) to be `supported`.
