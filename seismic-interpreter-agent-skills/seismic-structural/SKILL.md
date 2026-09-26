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
| Fault attribute | `seismic_fault_likelihood` (job) | discontinuity volume, measured only on reflectors |
| Fault traces | `seismic_extract_faults` | id, extent, strike (grid and azimuth), z range, polyline; fault-set file |
| Throw | `seismic_fault_throw` | throw along the fault, downthrown side, cycle-skip check, horizon offsets |
| Closure | `seismic_closure` | crest, spill point and type, column height, area |
| Depth | `seismic_time_to_depth` | depth map or depth volume with the stated velocity model |
| Sections, attributes | `seismic_describe_section`, edge and dip attributes | breaks, continuity, dip, as before |
| Figures | `seismic_render_map`, `seismic_render_section` | for people |

## Horizon procedure

1. Choose the event at a well, crest or clear location with `seismic_list_events`. Give the event kind (peak or trough) and z from its result; never pick z from memory or a description.
2. Before tracking across a faulted area, extract the faults. Tracking does not jump faults: put at least one seed in each fault block.
3. Run `seismic_track_horizon`. Check `coverage`, `correlation_p10` and `rejected_candidates`:
   - coverage below 1 is expected at faults, polarity changes and fading reflectors; say where the gaps are (`seismic_horizon_map` structure tile means show undefined tiles);
   - `low_correlation` stops mean the event changes character; do not lower `min_correlation` just to fill the map without saying so.
4. A polarity change along a horizon (for example a soft gas sand becoming a weak hard brine sand) stops a peak or trough tracker. That is information: report it, and grid supplied picks or track a nearby continuous event instead.

## Fault procedure

1. `seismic_fault_likelihood` on the interval with reflectors (crop with z_range; quiet intervals carry no fault information).
2. `seismic_extract_faults`. Positions are uncertain by about one trace. "No faults detected at this threshold" is not "no faults": report the threshold.
3. Confirm each fault that matters on at least one section with `seismic_describe_section` (sharpest_lateral_breaks at the same position).
4. `seismic_fault_throw` with horizons tracked on both sides where possible. Correlation throw and horizon throw should agree; if `possible_cycle_skip_fraction` is high, the correlation throw may be off by one period and only the horizon throw counts.
5. Alternatives to consider every time: channel edge, chaotic facies boundary, acquisition footprint (aligned with inline or crossline, regular spacing), migration artefact near steep dip, velocity pull-up or push-down.

## Closure and depth

1. `seismic_closure` on a structure map gives four-way closures only. For each: crest, spill z and location, spill type, column height, area.
   - `saddle`: the closure spills into a neighbouring culmination; the spill point location is part of the answer.
   - `data_edge`: the closure reaches the edge of the mapped data; the true closure may be larger or fault-bounded. Report the numbers as minimum values.
   - Fault-dependent closure is not computed: state it as a limitation when a fault bounds the structure.
2. A closure in time always carries the velocity warning. Convert with `seismic_time_to_depth` using a supplied time-depth table or a stated velocity model, then run `seismic_closure` on the depth map. Report both, and whether the closure survives conversion.
3. The velocity model is an input: name its source (task, well W1 checkshot, wells_petrophysics finding). A constant or linear velocity is an **assumption** unless supplied.
4. Closure is not a hydrocarbon column. A spill point bounds the trap; the fluid contact needs other evidence (see `seismic-qi`).

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
