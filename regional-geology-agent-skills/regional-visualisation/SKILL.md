---
name: regional-visualisation
description: Regional maps, rose diagrams and profiles - the rendering standards the tool enforces, the describe-then-interpret protocol, and what maps may and may not support. Load before reg_render.
---

# Regional visualisation


## What a figure is and is not

A figure illustrates a claim a tool computed. It never carries a claim, and no number, depth, area or boundary is read off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested, which is the one way a picture can damage an argument.

You do not see the figures you render: the tool returns a URL, not an image. They are for the reader and for the audit, so the caption must say what the figure shows, from which product and version, well enough to be read without you.

## `reg_render`

| Kind | Inputs | Use |
|---|---|---|
| map | `grid` (a surface), `points` (tables from `reg_points_near`), `faults`, the AOI | Where the data are relative to the AOI; structural grain; fields |
| rose | `azimuths` | Fault trends; SHmax indicators |
| profile | `profile_table` from `reg_grid_sample(mode=profile)` | Bathymetry, sediment thickness or anomalies along a line |

## Standards (enforced by the tool)

- The AOI outline (red) and the regional extent (dashed) are always drawn.
- The projection is printed: a local azimuthal equal-area, so areas and distances read correctly.
- Measurement points are drawn over any surface: a smooth grid looks like data everywhere; the points show where it is constrained.
- Fields use a perceptually uniform colour map; anomalies (gravity, magnetics; `anomaly=true`) a diverging, zero-centred one.
- Scale bar and graticule are drawn; the caption names datasets, versions and the extent.

## Protocol

1. **Describe, then interpret**: "heat-flow points cluster west of the AOI; none within 40 km east" before any meaning.
2. **Maps give hypotheses, tools give numbers**: a trend seen on a map enters the findings only with `reg_trend_statistics` or `reg_shmax` behind it.
3. **Support before surface**: check point density near the AOI (`reg_points_near`) before reading a gridded field there; sparse support goes in `limitations`.
4. **Distances and angles come from tools**, not from reading the map.

## Reporting

`Figure: <caption> — <url>` in `conclusions`, with the caption the tool returned (datasets, versions, extent).
