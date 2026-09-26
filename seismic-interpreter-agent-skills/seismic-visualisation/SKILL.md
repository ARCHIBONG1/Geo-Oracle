---
name: seismic-visualisation
description: How the agent "sees" seismic - reading seismic_describe_section output as its only view of the data - reading map descriptions from the map tools, how to produce figures for people with seismic_render_section, seismic_render_slice, seismic_render_map and seismic_render_3d (CIGVIS 3D scenes with fault, geobody, channel and horizon overlays), and how to show them inline in the chat. Load before describing, rendering or citing any section, and whenever the user asks to see or view anything.
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

## Your view of maps

Map tools (`seismic_track_horizon`, `seismic_horizon_map`, `seismic_time_to_depth`, `seismic_avo`, `seismic_4d_difference`) return a `map` object: coverage, minimum and maximum with inline/crossline (and X/Y), p5/p50/p95, a grid of tile means with their inline and crossline ranges (null = no data), and the mean gradient per inline and crossline step. For structure maps (z down) the minimum is the shallowest point. Describe maps from these numbers only, with their locations.

## Figures

`seismic_render_section` (inline or crossline), `seismic_render_slice` (z-slice) and `seismic_render_map` (any map file: structure with contours, isochron, extraction, closure, facies or AVO classes, NRMS) write PNGs to fixed standards:
- signed amplitude uses the RdBu_r colormap, clipped symmetrically at a percentile of |amplitude|;
- unsigned attributes use cividis, clipped at percentiles;
- axes are labelled inline/crossline and TWT or depth;
- the clip, colormap and polarity are printed on the figure.

- Render the sections your key evidence rests on, so a person can check them. One figure per key claim is enough.
- Cite the figure in the related evidence item's `source_reference`, for example `prov:… seismic_render_section IL 120, figure figures/….png`.

## Showing figures to the user

Every render tool returns `markdown`, a ready image line such as `![caption](http://127.0.0.1:8792/files/seismic-figures/….png)`. TrueForge's chat displays it inline. When the user asks to see, view or show something, render it and open your final message with the `markdown` line(s) and a one-line caption each, then the findings JSON in a ```json block (see the output contract). Never send the user to a link to view a static figure, and never put figures in a `sandbox_artifacts` block.

## 3D views with CIGVIS (`seismic_render_3d`)

The 3D tool drives [CIGVIS](https://github.com/JintaoLee-Roger/cigvis), a published, tested library for seismic visualization. It draws the seismic as three orthogonal slices and adds interpretation overlays and horizons. Use it whenever the object is 3D (a fault system, geobody, channel, horizon, or the volume itself) and the user asks to see it. Use 2D sections and maps for measuring and citing.

**Pre-check (enforced by the tool).** Every overlay must have the same shape as the seismic on the same IL,XL,T grid. Ingested volumes are always in IL,XL,T, which is also CIGVIS's own order (x = inline, y = crossline, z = time downwards), so no transposing is needed.
- If a pre-check fails, the message gives both grids. Either compute the overlay on the whole volume, or window the seismic to the overlay's ranges with `seismic_extract_window`.
- Plan ahead: to overlay faults in 3D, run `seismic_fault_likelihood` on the same volume or window you will display.

**Overlays** (`overlays: [{path, style, threshold, color, cmap, alpha, bodies}]`):

| Overlay | Typical settings |
|---|---|
| Binary faults (`seismic_binarize_volume`, fault surfaces) | `style: "both"`: faults coloured on the slices and as 3D sheets; threshold default 0.5 |
| Geobody, channel or anomaly body (binary) | `style: "body"`, `bodies: "intersecting"` to show only bodies cut by the displayed slices (CIGVIS's "full" geobody perspective), or `"all"` |
| Probability or likelihood volume, not binarized | `style: "mask"` with an explicit `threshold` (for example the fault-extraction threshold) |
| Labelled faults (`output: "labels"`) | `threshold: 0.5` shows every labelled fault |

**Horizons**: pass horizon maps in `horizons`; they are drawn as surfaces coloured by time or depth.

**View**:
- Slice positions (`inline`, `crossline`, `z`) are in survey units, and default to the middle. To reveal bodies inside the volume, put the slices on the faces behind them (for example `inline` = last, `crossline` = first, `z` = bottom) and choose `azimuth` so those faces are at the back.
- `hide_slices` removes slices that hide the feature.
- `vertical_exaggeration` defaults to a roughly cubic box.
- State the camera and slices in the caption.

**How to present a 3D view**: show the static image inline (its `markdown` line), then give `interactive_url` with the tool's `interactive_note`. For example: "TrueForge's chat can show images but not interactive 3D, so the static view is shown here; to rotate, zoom and pan the same scene, open the interactive link in a browser tab: <interactive_url>". The interactive page is the same CIGVIS scene, decimated when large.

**CIGVIS 0.3 API notes**, if you read CIGVIS code or examples:
- `create_overlay` no longer works on the VisPy backend; `create_slices` followed by `add_mask` replaces it.
- `create_bodys` is `create_bodies`.
- `plot3D` takes `view=Plot3DView(...)` and `save=Plot3DSave(...)`.
- The Plotly backend lacks fault skins and well logs.
The tool handles all of this; you only choose the parameters.
- Time slices cut across dipping strata. Label any time-slice pattern as mixed-level unless the strata are flat.
- Never describe what a figure "shows". Describe the measurements behind it.
