---
name: regional-visualisation
description: Regional maps, rose diagrams, profiles, subsidence curves and geotherms - the figures the tools draw, the rendering standards, the describe-then-interpret protocol, and what figures may and may not support. Load before reg_render.
---

# Regional visualisation


## What a figure is and is not

A figure illustrates a claim that a tool computed. It never carries a claim. You never read a number, depth, area or boundary off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested. That is the one way a picture can damage an argument.

You do not see the figures you render: the tool returns a URL, not an image. Check the numbers the tool returns instead. Figures are for the person, Geo Oracle's answer and the report. So the caption must say what the figure shows, from which product and version, well enough to be read without you.

## `reg_render`

| Kind | Inputs | Use |
|---|---|---|
| map | `grid` (a surface), `points` (tables from `reg_points_near`), `faults`, the AOI | Where the data are relative to the AOI; structural grain; fields |
| rose | `azimuths` | Fault trends from a tool result. `reg_shmax` returns no azimuth per indicator, and the `stress_field` product has no figure |
| profile | `profile_table` from `reg_grid_sample(mode=profile)` | Bathymetry, sediment thickness or anomalies along a line |
| subsidence | `product`: a `burial_history` product | Total and tectonic subsidence against age; the McKenzie fit, beta and the rift interval when the product holds a fit |
| geotherm | `product`: a `thermal_model` product | Temperature and heat flow against depth; mean gradient, surface heat flow, layer boundaries |

## Figures the tools draw

These tools draw their product's figure themselves and return it in `result.figures`. Do not draw it again with `reg_render`.

- `reg_aoi`: a map of the AOI and its regional extent.
- `reg_grid_sample` in profile mode: the profile.
- `reg_fault_elements`: a rose of the fault trends.
- `reg_backstrip`: the subsidence curve.
- `reg_geotherm`: the geotherm.

If the tool warns that it could not draw a figure, the product stays valid. If a claim rests on that figure, draw it with `reg_render`.

## Standards (enforced by the tool)

- The AOI outline (red) and the regional extent (dashed) are always drawn.
- The projection is printed: a local azimuthal equal-area, so areas and distances read correctly.
- Measurement points are drawn over any surface. A smooth grid looks like data everywhere; the points show where it is constrained.
- Fields use a perceptually uniform colour map; anomalies (gravity, magnetics; `anomaly=true`) a diverging, zero-centred one.
- Scale bar (true at the projection centre) and graticule are drawn. The caption names datasets, versions and the extent.
- Rose diagrams are equal-area: a sector's radius goes as the square root of its count (Nemec 1988). So sector areas compare as counts do.
- Subsidence curves plot age increasing to the left and subsidence downward. Geotherms plot depth downward.

## Protocol

1. **Describe, then interpret**, from the numbers the tools return: "heat-flow points cluster west of the AOI; none within 40 km east" before any meaning.
2. **Figures give no numbers, tools do**: a trend enters the findings only with `reg_trend_statistics` or `reg_shmax` behind it.
3. **Support before surface**: check point density near the AOI (`reg_points_near`) before you read a gridded field there. Put sparse support in `limitations`.
4. **Distances and angles come from tools**, not from reading the map.

## Reporting

Publish a figure for every product that you publish and that has a renderer. For each figure, add a `figures` entry (`url`, `caption`, `product`, `supports`: the claim ids it illustrates). Also add a `Figure: <caption> — <url> — supports <claim ids>` line in `conclusions`. Use the caption the tool returned (datasets, versions, extent). Begin your reply with each figure's `markdown` line.

<!-- end of skill: regional-visualisation -->
