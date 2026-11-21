---
name: strat-visualisation
description: Stratigraphic figures. It covers which tools draw their own, the stg_render kinds (panel, age_depth, wheeler, isopach) and the rendering standards that the tools enforce. It also covers the two-datum check, and what a figure may and may not support. Load before stg_render and before reporting any figure.
---

# Stratigraphic visualisation

## What a figure is and is not

A figure illustrates a claim that a tool computed. It never carries a claim, and you never read a number, depth, area or boundary off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested. That is the one way a picture can damage an argument.

You do not see the figures: a tool returns a URL, not an image. Figures are for the person, Geo Oracle's answer and the investigation report. The caption must say what the figure shows, from which product and version. It must say it well enough that a reader can read it without you.

## Publish a figure for every product that has a renderer

- Tools that draw their own figure return it in `result.figures` (`url`, `caption`, `product`, `markdown`). Carry each one into your findings.
  - `stg_build_panel` draws the panel (hung on its datum when every well has the datum pick).
  - `stg_age_model` draws the well's age-depth plot.
  - `stg_consistency` with `age_models` draws the panel's Wheeler diagram.
  - `stg_isopach` draws the isopach map.
- Draw the other products with `stg_render`. If a product's figure must show a change (a tested panel, another datum, several wells' age models on one plot), draw it again with `stg_render`.
- If one kind has many instances, publish the ones that a claim rests on, and at least the most representative one.
- A user request can add figures. It is never the condition for the standard ones.

## `stg_render` kinds

- `panel`: the panel file, the wells' `correlation_logs` in panel order, the curve, the datum. The tool draws the unflattened panel (TVDSS) and the panel hung on the datum side by side.
- `age_depth`: one or more `age_model` products on one plot, with the datums' error bars and the p10-p90 envelopes.
- `wheeler`: the panel file, the wells' `correlation_logs` and their `age_model` products. The tool draws each interval between panel surfaces at its modelled ages per well.
- `isopach`: an `isopach_trends` product. The tool posts the thickness at each well, with the residual from the planar trend in brackets. It draws the planar trend as dashed contours. It grids nothing between the wells beyond that plane.

## Standards (enforced)

- Wells spaced by their distance apart, not evenly; one curve track per well with identical scale.
- Depth axis in TVDSS; the hung panel's axis is depth relative to the datum.
- Reported tops dashed, interpreted picks solid, with a key; tie ranks printed beside the correlation lines.
- The caption names the panel, its status and its ledger.
- The isopach map states its thickness basis: isopach (true stratigraphic) or isochore (true vertical).

## Protocol

1. **Describe, then interpret**: "the sand's top is 15 m below FS3 in W1 and 12 m below FS2 in W4" before any meaning. The numbers come from the tools.
2. **Check two datums.** Render the key correlation hung on two different time lines (two `stg_render` panel calls with different `datum`). Test the conclusion on the numbers that the tools return for each hanging. Run `stg_isopach` from each datum to the surface in question, and compare the thicknesses per well. Flag a conclusion that holds on only one datum.
3. **Panels give hypotheses, tools give correlations.** A match on a panel enters the findings only with a DTW result and a ledger behind it.
4. **Depths, thin beds and small offsets come from tools**, never from a figure.

## Reporting

For each figure, add a `figures` entry (`url`, `caption`, `product`, `supports`: the claim ids it illustrates). Also add a `Figure: <caption> — <url> — supports <claim ids>` line in `conclusions`, with the datum and the panel's status named. Begin the reply with each figure's `markdown` line and a one-line caption, then give the findings JSON.

<!-- end of skill: strat-visualisation -->
