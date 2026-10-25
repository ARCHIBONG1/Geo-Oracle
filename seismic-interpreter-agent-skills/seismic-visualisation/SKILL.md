---
name: seismic-visualisation
description: How the agent "sees" seismic. It covers reading seismic_describe_section output as its only view of the data, and reading map descriptions from the map tools. It also covers how to publish figures with seismic_render_section (with horizon, fault and well overlays), seismic_render_slice, seismic_render_map and seismic_render_3d (CIGVIS 3D scenes with fault, geobody, channel and horizon overlays). Last, it covers how to show them inline in the chat. Load before describing or rendering a section, and before publishing figures.
---

# Seeing seismic without images

You cannot view images. Tool results reach you as text. Figures are for the person, Geo Oracle's answer and the report. Your view of the data is `seismic_describe_section`. Never claim to have looked at a display.


## What a figure is and is not

A figure illustrates a claim that a tool computed. It never carries a claim, and you never read a number, depth, area or boundary off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested. That is the one way a picture can damage an argument.

You do not see the figures you render: the tool returns a URL, not an image. They are for the person, Geo Oracle's answer and the report. So the caption must say what the figure shows, from which product and version, well enough to be read without you.

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
- Describe the attribute volume on the same sections as the amplitude volume, so that you can compare them tile by tile.
- Quote values with their location. Example: "IL 120: continuity 0.91–0.96 in all tiles except the XL 248–260 column (0.58–0.66) from 300 to 900 ms (E4, prov:…)."

Limits:
- Tiles average over their area, so features smaller than a tile show only as reduced values.
- Noise lowers continuity, as geology does. Read it alongside the `seismic_qc` SNR.
- The anomaly list shows only the strongest five. Absence from the list does not mean absence of anomalies.

## Your view of maps

Map tools (`seismic_track_horizon`, `seismic_horizon_map`, `seismic_time_to_depth`, `seismic_avo`, `seismic_4d_difference`) return a `map` object with these parts:
- coverage;
- minimum and maximum with inline/crossline (and X/Y);
- p5/p50/p95;
- a grid of tile means with their inline and crossline ranges (null = no data);
- the mean gradient per inline and crossline step.

For structure maps (z down), the minimum is the shallowest point. Describe maps from these numbers only, with their locations.

## Figures

`seismic_render_section` (inline or crossline), `seismic_render_slice` (z-slice) and `seismic_render_map` (any map file: structure with contours, isochron, extraction, closure, facies or AVO classes, NRMS) write PNGs to fixed standards:
- signed amplitude uses the Petrel colormap (Petrel's seismic colour table: blue and cyan troughs, grey at zero, brown, red and yellow peaks), clipped symmetrically at a percentile of |amplitude|. `seismic_render_3d` uses the same by default;
- unsigned attributes use cividis, clipped at percentiles;
- interpretation figures over the seismic (`seismic_interpret_faults`' faults figure, `seismic_render_extraction` with the seismic as `background`) use the same Petrel table, with faults black and bodies yellow, each numbered. Over a recipe score or an attribute, the background stays grey, with one colour per body;
- axes are labelled inline/crossline and TWT or depth;
- the clip, colormap and polarity are printed on the figure.

**Section overlays** (`seismic_render_section`). Draw the products of a claim on its section:
- `horizons`: horizon maps on the volume's grid and z unit. `horizon_names` labels them in the same order.
- `faults`: a fault set (`outputs.fault_set` of `seismic_interpret_faults` or `seismic_extract_faults`). Each fault is a black stick where it cuts the section.
- `wells`: each well's `well_header` product and its `tops_time` product or `well_tops` table. If a well is within `well_distance` lines, the tool draws it vertical at its X/Y, with its tops as ticks. It also draws a deviated well vertical, and `result.overlays` says so.
- `result.overlays` gives each drawn item's measured position (line range, z range, distance). In `not_drawn`, it lists what does not cut the section, with the reason. Quote positions from there.

**Closures**: `seismic_render_map` with `closures` (the closure map of `seismic_closure`) outlines each closure on its horizon map, and marks its crest.

- Publish a figure for every product that you publish and that has a renderer (system prompt, Figures step). Draw the sections that your key evidence rests on, with the horizons, faults and wells of the claim on them.
- List each figure in `figures`, with the claim ids it illustrates in `supports`. Add its `Figure: <caption> — <url> — supports <claim ids>` line to `conclusions`. Never cite a figure in an evidence item's `source_reference`: the evidence cites the tool result's `prov:` id, and a figure carries no claim.
- These tools draw their own figure into `result.figures`: `seismic_track_horizon`, `seismic_horizon_map` (every map it writes), `seismic_closure` (closures on the horizon), `seismic_time_to_depth` (maps), `seismic_interpret_faults`, `seismic_segment_volume`, `seismic_classify_facies`, `seismic_avo` (class map), `seismic_dhi_screen` (amplitude map), `seismic_4d_difference` (NRMS map), `seismic_extract_wavelet`, `seismic_synthetic` and `seismic_well_tie`.
  - The last three also return `wavelet_figure_markdown`, `synthetic_figure_markdown` and `well_tie_figure_markdown`: the same figures.
  - Publish all three for every well you tie (the seismic-qi skill says what the reader should check).

## Showing figures to the user

Every figure in `result.figures` carries `markdown`, a ready image line such as `![caption](http://127.0.0.1:8792/files/seismic-figures/….png)`. TrueForge's chat displays it inline. Open your final message with the `markdown` line of every figure you publish, and a one-line caption for each. Then give the findings JSON in a ```json block (see the output contract). A user request adds figures to these. It is never the condition for them. Never send the user to a link to view a static figure, and never put figures in a `sandbox_artifacts` block.

## 3D views with CIGVIS (`seismic_render_3d`)

The 3D tool drives [CIGVIS](https://github.com/JintaoLee-Roger/cigvis), a published, tested library for seismic visualization. It draws the seismic as three orthogonal slices, and adds interpretation overlays and horizons. A 3D render takes seconds to minutes, so draw it on request only. Draw one when the task or the user asks for a 3D view of a 3D object (a fault system, geobody, channel, horizon, or the volume itself). The standard figures are 2D sections and maps, which are also what you measure and cite.

**Pre-check (enforced by the tool).** Every overlay must have the same shape as the seismic, on the same IL,XL,T grid. Ingested volumes are always in IL,XL,T. This is also CIGVIS's own order (x = inline, y = crossline, z = time downwards), so no transposing is needed.
- If a pre-check fails, the message gives both grids. Either compute the overlay on the whole volume, or window the seismic to the overlay's ranges with `seismic_extract_window`.
- Plan ahead: to overlay faults in 3D, run `seismic_interpret_faults` (or `seismic_fault_likelihood`) on the same volume or window that you will display.

**Overlays** (`overlays: [{path, style, threshold, color, cmap, alpha, bodies}]`):

| Overlay | Typical settings |
|---|---|
| Binary faults (`seismic_interpret_faults` with `output: "binary"`, or `seismic_binarize_volume` with a fault set) | `style: "both"`: faults coloured on the slices and as 3D sheets; threshold default 0.5 |
| Geobody, channel or anomaly body (binary) | `style: "body"`, `bodies: "intersecting"` to show only bodies cut by the displayed slices (CIGVIS's "full" geobody perspective), or `"all"` |
| Probability or likelihood volume, not binarized | `style: "mask"` with an explicit `threshold` (for example the fault-extraction threshold) |
| Labelled faults (`output: "labels"`) | `threshold: 0.5` shows every labelled fault |

The tool draws faults black and bodies yellow, so both stand out on the Petrel-coloured slices. It reads which an overlay is from its lineage. Set `color` (bodies) or `cmap` (masks) only when the person asks for other colours.

**Horizons**: pass horizon maps in `horizons`. The tool draws them as surfaces coloured by time or depth.

**View**:
- Slice positions (`inline`, `crossline`, `z`) are in survey units, and default to the middle. To reveal bodies inside the volume, put the slices on the faces behind them (for example `inline` = last, `crossline` = first, `z` = bottom). Choose `azimuth` so that those faces are at the back.
- `hide_slices` removes slices that hide the feature.
- `vertical_exaggeration` defaults to a roughly cubic box.
- Leave `zoom` unset. The tool frames the scene automatically on any display: it zooms out if the scene would be clipped, and in if it is small, then it crops. Set `zoom` only to look closer at a detail (for example 2 for twice as close).
- State the camera and slices in the caption.

**How to present a 3D view**: show the static image inline (its `markdown` line). Then give `interactive_url` with the tool's `interactive_note`. For example: "TrueForge's chat can show images but not interactive 3D, so the static view is shown here; to rotate, zoom and pan the same scene, open the interactive link in a browser tab: <interactive_url>". The interactive page is the same CIGVIS scene, decimated when large.

- Time slices cut across dipping strata. Label any time-slice pattern as mixed-level unless the strata are flat.
- Never describe what a figure "shows". Describe the measurements behind it.

<!-- end of skill: seismic-visualisation -->
