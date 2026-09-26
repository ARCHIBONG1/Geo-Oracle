---
name: seismic-visualisation
description: How the agent "sees" seismic - reading seismic_describe_section output as its only view of the data - reading map descriptions from the map tools, and how to produce standardised audit figures for people with seismic_render_section, seismic_render_slice and seismic_render_map. Load before describing, rendering or citing any section.
---

# Seeing seismic without images

You cannot view images. Tool results reach you as text, so rendered figures are for people to audit your work. Your view of the data is `seismic_describe_section`. Never claim to have looked at a display.

## seismic_describe_section

Inputs:
- `path` (amplitude or attribute volume);
- one of `inline` or `crossline`;
- `z_range`;
- `n_lateral` and `n_vertical` tiles, default 8 × 8, up to 12;
- `max_lag`, default 6 samples.

The output has these parts:
- `tiles.columns` and `tiles.rows`: the inline/crossline and z extent of each tile.
- `tiles.relative_rms`: tile RMS divided by the section RMS. Above 1 is brighter than average.
- `tiles.continuity`: mean best correlation of adjacent traces, from 0 to 1. It measures how laterally continuous the reflectors are.
- `tiles.apparent_dip`: z units per trace. Positive means deeper towards higher inline or crossline numbers.
- `strongest_envelope_anomalies`: up to five located envelope peaks above the 99.5th percentile.
- `sharpest_lateral_breaks`: adjacent trace pairs much less similar than their row, with the z ranges where each occurs.
- `low_continuity_tiles`: tiles well below the section's typical continuity.

Good practice:
- Describe several sections, not one. Include sections perpendicular to the feature, and control sections away from it for comparison.
- Use finer tiles (12) over a narrow `z_range` to resolve a target. Coarse tiles over the full trace give context.
- Increase `max_lag` where dips are steep. If `apparent_dip` values sit at ±max_lag × dt, the true dip is steeper than measured.
- Describe the attribute volume on the same sections as the amplitude volume, so you can compare them tile by tile.
- Quote values with their location. Example: "IL 120: continuity 0.91–0.96 in all tiles except the XL 248–260 column (0.58–0.66) from 300 to 900 ms (E4, prov:…)."

Limits:
- Tiles average over their area, so features smaller than a tile show only as reduced values.
- Continuity is lowered by noise as well as by geology. Read it alongside the `seismic_qc` SNR.
- The anomaly list shows only the strongest five. Absence from the list does not mean absence of anomalies.

## Audit figures

## Your view of maps

Map tools (`seismic_track_horizon`, `seismic_horizon_map`, `seismic_time_to_depth`, `seismic_avo`, `seismic_4d_difference`) return a `map` object: coverage, minimum and maximum with inline/crossline (and X/Y), p5/p50/p95, a grid of tile means with their inline and crossline ranges (null = no data), and the mean gradient per inline and crossline step. For structure maps (z down) the minimum is the shallowest point. Describe maps from these numbers only, with their locations.

## Figures

`seismic_render_section` (inline or crossline), `seismic_render_slice` (z-slice) and `seismic_render_map` (any map file: structure with contours, isochron, extraction, closure, facies or AVO classes, NRMS) write PNGs to fixed standards:
- signed amplitude uses the RdBu_r colormap, clipped symmetrically at a percentile of |amplitude|;
- unsigned attributes use cividis, clipped at percentiles;
- axes are labelled inline/crossline and TWT or depth;
- the clip, colormap and polarity are printed on the figure.

- Render the sections your key evidence rests on, so a person can check them. One figure per key claim is enough.
- Cite the figure in the related evidence item's `source_reference`, for example `prov:… seismic_render_section IL 120, figure figures/….png`. The result also gives a `url` a person can open.
- Time slices cut across dipping strata. Label any time-slice pattern as mixed-level unless the strata are flat.
- Never describe what a figure "shows". Describe the measurements behind it.
