---
name: seismic-structural
description: Structural interpretation from seismic - horizon tracking, faults and throw, structure and isochron maps, closure and spill point, time-to-depth conversion. It gives the procedures, traps and uncertainty rules for each tool. Load for questions about faults, traps, deformation, structure maps or depth.
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
| Distance from a well to each feature | `seismic_feature_distance` | per fault or body: lateral metres, 3D metres with a time-depth table, closest voxel, whether the well passes through it and over which z range |
| Faults when likelihood is poor; channels, karst, geobodies | `seismic_segment_volume` (see `seismic-attribute-interpretation`) | labelled or binary volume, body catalogue, extraction figure |
| 3D view of faults and horizons | `seismic_render_3d` | inline image plus interactive link (see `seismic-visualisation`) |
| Closure | `seismic_closure` | crest, spill point and type, column height, area |
| Depth | `seismic_time_to_depth` | depth map or depth volume with the stated velocity model |
| Sections, attributes | `seismic_describe_section`, edge and dip attributes | breaks, continuity, dip, as before |
| Figures | `seismic_track_horizon`, `seismic_horizon_map`, `seismic_closure` and `seismic_time_to_depth` draw their maps themselves; `seismic_render_section` with `horizons`, `faults` and `wells`; `seismic_render_map` with `closures` | `result.figures`; section overlay positions in `result.overlays` |

## Horizon procedure

1. Choose the event at a well, crest or clear location with `seismic_list_events`. Take the event kind (peak or trough) and z from its result. Never pick z from memory or a description.
   **At a tied well, the synthetic chooses the event.** Run `seismic_trace_events` at the well with these inputs:
   - the synthetic and the tie's shift and phase (`synthetic`, `tie_shift_ms`, `tie_phase_deg`);
   - the horizon's marker (`markers`: its top, or the synthetic's reflection at the target level). Without markers, `synthetic_reflections` matches each strong reflection.
   Seed on `markers[].seed`: the seismic event of the polarity that the synthetic shows at the marker. Do not seed on the nearest event, and never on `first_strong_event` (the seabed criterion). If you seed on another event, say why in assumptions. Suppose that the synthetic predicts an event of the other polarity a little deeper. Then a seed on the first strong event puts the horizon on the wrong loop, and every depth with it.
2. Before tracking across a faulted area, extract the faults (`seismic_interpret_faults`). Tracking does not jump faults, so put at least one seed in each fault block. **Each block's seed takes its time from that block**, never the well's time copied across:
   - run `seismic_fault_throw` for the fault between the well's block and the next;
   - add the throw to the event's time on the far side (sign from `downthrown_side`);
   - confirm with `seismic_list_events` at the far seed.
   The same event at the same time on both sides of a fault with 30 ms of throw is a different loop. Pass the fault set to the tracker as `faults`. The tracker then compares the horizon's step across every fault with the seismic's own offset, and flags a cycle skip (`result.qc.faults`).
3. Run `seismic_track_horizon`. Check `coverage`, `correlation_p10` and `rejected_candidates`:
   - `result.qc`: `measured.coverage`, `signal.correlation_p10`, and `stability` (the same tracking with the stopping threshold x0.9 and x1.1: the share of cells within one sample). Cells that move with the setting are uncertain picks. Report their share.
   - Coverage below 1 is expected at faults, polarity changes and fading reflectors. Say where the gaps are (`seismic_horizon_map` structure tile means show undefined tiles).
   - `low_correlation` stops mean that the event changes character. Do not lower `min_correlation` just to fill the map without saying so. Check a map that you broaden that way, and do not assume it. A lowered threshold lets the tracker cross a fault onto a different loop, and a depth map inherits the error.
   - `result.qc.faults` (with `faults` given): per fault, the horizon's step against the seismic's own offset. `cycle_skip_suspected` means that the block beyond is on the wrong loop. Retrack with that block's seed at the throw-corrected time, or report the block as untracked. Never deliver a horizon with a suspected skip as it stands, and never convert it to depth.
4. A polarity change along a horizon (for example a soft gas sand becoming a weak hard brine sand) stops a peak or trough tracker. That is information, not a failure. Seed the opposite event beyond the change, and track it. Report where the change sits, as a candidate fluid or lithology boundary (`seismic-qi`).

## Fault procedure

1. Run `seismic_interpret_faults` on the seismic amplitude volume. Crop it with `z_range` to the interval with reflectors: quiet intervals carry no fault information. The tool runs the whole chain in a fixed order: fault likelihood, its QC, 3D extraction of steep planes, and each fault drawn as a surface. Use it whenever faults or a fault volume are asked for. `output: "labels"` (default) gives 0 = no fault, 1 = F1, 2 = F2, ...; `output: "binary"` gives 1 = fault. It also writes the fault set (for `seismic_fault_throw`, `seismic_publish_structure` and `seismic_render_3d`) and the likelihood volume.
   - Read `result.qc` first.
     - If it finds reflectors that dip beyond `max_dip_samples`, the tool has already recomputed the likelihood at the suggested value, and extracted the faults from it. `qc.rerun` holds both passes (`first_pass`, `to_max_dip_samples`), and `outputs.likelihood_first_pass` holds the first volume. Report both.
     - With `follow_qc: false`, the tool keeps the setting asked for, and warns. Then rerun yourself, or say why not.
     - If its notes say that few measured voxels reach the threshold, the input may not be reflection amplitudes, or the interval holds no faults at this scale. Check the input (`seismic_describe_volume` shows its lineage) before you change any threshold.
   - If likelihood still looks poor (low `measured_fraction`, low `separability.otsu_eta`, or no sensible faults where sections show sharp breaks), also run the `fault_attribute` recipe (`seismic-attribute-interpretation`). Compare the two. If the user explicitly asked for an approach, run it as asked.
   - The likelihood shows every lateral discontinuity (faults, channel edges, chaotic facies). The faults are only the steep planes that it supports over a large depth range. Never call a discontinuity a fault just because it is bright in the likelihood.
   - Each fault carries `support`: `measured`, or `mostly_interpolated` when under a fifth of its surface rests on measured likelihood. Report a mostly interpolated fault as weakly supported.
   - **Cross-check.** `qc.cross_check` compares the drawn faults with semblance computed on the same window. It gives each fault's share in the low-semblance tail, the share of that tail that the faults capture, and the tall low-coherence pieces that no fault touches.
     - A fault weakly expressed in semblance needs a section before you report it.
     - An unexplained tall piece is a candidate fault, a channel edge or chaotic facies: describe a section through it.
     - The seismic is the same, so agreement is consistency, not confirmation.
     - `seismic_structure_qc` runs the same comparison against any other discontinuity attribute.
   - **Persistence.** The tool also looks for the faults at the eight settings around the one asked for (threshold and `min_z_extent` each x0.8 and x1.25). It draws only the faults found at two thirds of them (`min_persistence`). Each fault carries `found_at` (for example `9/9`). `result.persistence` gives the count at every setting, and lists the less persistent faults. Report the drawn faults with that table. If a count changes across settings, report it as unstable, with the table: **never change settings, or keep a subset of faults, to reach a number anyone expects**, the person included. Their expected count is a hypothesis that you test and report on.
   - Report each fault's `apparent_dip` (traces per sample, in index units: true dip needs velocity and trace spacing) and z range. Positions are uncertain by about one trace.
   - "No faults found at this threshold" is not "no faults". Report the threshold (default 0.3) and the qc notes. If sections show sharp breaks, retry once with `threshold: 0.2` or a smaller `min_z_extent`, and report both runs.
2. **Step by step**: use this only for a step that the single tool does not expose (`method: "map"` for vertical faults in legacy workflows, keeping chosen faults with `fault_ids` and a stated `selection_reason`, `fault_geometry: "voxels"`). Run `seismic_fault_likelihood` on the seismic, then `seismic_extract_faults` on the likelihood, then `seismic_binarize_volume` on that likelihood with the fault set. The tools refuse any other order:
   - likelihood on an attribute such as coherence;
   - extraction from the seismic;
   - thresholding likelihood without a fault set (that gives blobs, not faults);
   - a fault set drawn on a volume that it was not extracted from.
   `override_reason` exists only for an input that the user explicitly asked for. Put the reason in `limitations`.
3. Before interpreting, describe at least one inline and one crossline with `seismic_describe_section`. `result.sections_to_inspect` names the pair through the largest fault. Confirm each fault that matters where `sharpest_lateral_breaks` sit at its position. One section is never enough: a fault seen on one inline only is a candidate.
4. Measure **well-to-feature distances**, and never read them from extents. Run `seismic_feature_distance` on the fault volume and on the body volume, with these inputs:
   - the well header (X/Y);
   - its track, when the well is deviated;
   - the time-depth table, so that the 3D distance is in metres.
   A bounding box that overlaps the well's position is not a distance and not an intersection.
5. Run `seismic_fault_throw` with horizons tracked on both sides where possible. Correlation throw and horizon throw should agree. If `possible_cycle_skip_fraction` is high, the correlation throw may be off by one period, and only the horizon throw counts.
6. **Delivery**: show `result.faults_figure_markdown`. Give the volume to the user with `seismic_export_volume` (in their axis order if they supplied an array). View it with `seismic_render_slice` or `seismic_render_3d`. A horizon or depth map goes to the user the same way: `seismic_export_volume` on the map file writes a csv (inline, crossline, x, y, value) or npy, never SEG-Y.
   - **Strike and dip.** The fault tools report `apparent_dip` in index units. The true orientation comes from `seismic_publish_structure` with a time-depth table (or a depth volume). Its reply's `fault_orientations` gives each fault's `strike_deg`, `dip_deg` and `dip_azimuth_deg`. Report those as the orientation, with the table as the source. "apparent grid-index dip" is the answer only when no table exists.
7. Alternatives to consider every time: channel edge, chaotic facies boundary, acquisition footprint (aligned with inline or crossline, regular spacing), migration artefact near steep dip, velocity pull-up or push-down.

## Closure and depth

1. `seismic_closure` on a structure map gives four-way closures only. For each, it gives the crest, spill z and location, spill type, column height and area.
   - `saddle`: the closure spills into a neighbouring culmination. The spill point location is part of the answer.
   - `data_edge`: the closure reaches the edge of the mapped data. The true closure may be larger or fault-bounded. Report the numbers as minimum values.
   - The tool does not compute fault-dependent closure. If a fault bounds the structure, state this as a limitation.
2. A closure in time always carries the velocity warning. Convert with `seismic_time_to_depth`, using a supplied time-depth table or a stated velocity model. Then run `seismic_closure` on the depth map. Report both, and whether the closure survives conversion.
3. The velocity model is an input: name its source (task, well W1 checkshot, wells_petrophysics finding). A constant or linear velocity is an **assumption** unless supplied.
   - **A depth map asked for is produced.** With one well's time-depth table (checkshots, `wells_petrophysics` `time_depth`), do these steps:
     - track the horizon (`seismic_track_horizon`, seeded at the well from `seismic_list_events`);
     - convert it with `seismic_time_to_depth` (`method: table`);
     - deliver the map with its uncertainty stated.
     The table is exact at the well, and the lateral velocity variation is unknown, so depths away from the well carry that error. Where the data allow, name a figure for it (for example from a second well or the checkshot drift). One well's function is the standard first depth map. Never withhold it as "not a validated lateral velocity model". Converting the whole volume is not a map.
4. Closure is not a hydrocarbon column. A spill point bounds the trap. The fluid contact needs other evidence (see `seismic-qi`).

## Publishing for structural_geology

When another specialist needs your faults and horizons (structural_geology above all), publish them with `seismic_publish_structure`:
- fault traces in X/Y with z ranges and, given a time-depth table or a depth volume, strike and dip;
- horizon grids with inline, crossline, x, y, z.

The sidecar states the domain and the pick uncertainty. structural_geology's tools refuse a time-domain product, so convert first where depth is needed. List each reference in `conclusions` as `Product: fault_set — @seismic-interpreter-agent/products/<file>` (likewise `horizon`). If a task feeds a structural model, a closure with faults, fault stability or fault seal, it is incomplete until these are published.

## Classification

- Tracked horizon values, map extremes, throw, closure numbers and depths are **measurements** (they depend on stated parameters and inputs).
- "Fault F1 is a normal fault downthrown to the east" is an **interpretation**.
- "The structure is a four-way closed trap of 0.28 km²" is an **interpretation** resting on the closure measurement plus the depth conversion.
- Trap validity (seal, charge, timing) is outside seismic geometry. It is an **inference** at most, and it needs other specialists.

## Uncertainty to report

- Horizon: tracking quality, coverage gaps and their causes, event choice (peak/trough) and phase.
- Faults: position ±1 trace and the lateral resolution. Throw in time is apparent. Throw below tuning thickness is unresolved.
- Depth: the velocity model and its source. For a closure, how sensitive spill and column are to it.
- Time data: geometry, dips and closures are apparent until converted.
- Structural claims usually need an independent line (well dips, regional structure from the structural_geology specialist) to be `supported`.


## Closure uncertainty

In `result.qc` of `seismic_closure`, the tool recomputes each closure on five realisations of the map, perturbed by its pick uncertainty (default: the map's measured pick noise). It gives `found_in` and the range of the closure's cells and column. If a closure is not found in every realisation, it is a pick-noise candidate until a denser pick or a well confirms it. Report the column range, not one number.

<!-- end of skill: seismic-structural -->
