---
name: prospect-visualisation
description: Closure maps, maturity tables, volume exceedance curves and tornadoes. It says which tools draw their own figure, what pt_render draws, and what a figure may support. Load before pt_render and before carrying result.figures.
---

# Prospect visualisation


## What a figure is and is not

A figure illustrates a claim that a tool computed. It never carries a claim, and you never read a number, depth, area or boundary off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested. That is the one way a picture can damage an argument.

You do not see the figures: a tool returns a URL, not an image. Check the numbers that the tool returns instead. Figures are for the person, Geo Oracle's answer and the report. So the caption must say what the figure shows, from which product and version, well enough to be read without you.

## The rule

Publish a figure for every product that you publish and that has a renderer. If one kind has many instances (prospects, scenarios), publish the ones that a claim rests on, and at least the most representative one. A user request can add figures. It is never the condition for the standard ones.

## Tools that draw their own figure

These return the figure in `result.figures`. Carry each one. The gateway flags a result that drops a figure that a tool drew.

- `pt_grv`: the closure map at the scenario's contact.
- `pt_quantity`: the exceedance curve of its `volume_ranges`.
- `pt_sensitivity`: the tornado of its sensitivity file.
- `pt_maturation_plan`: the maturity table of every candidate it reads.

## `pt_render`

Draw the other products with it. It also returns `result.figures`.

- `closure_map`: the closure drawn from its own depth cells. The crest, the spill point and the scenario's contact are marked, and the area above the contact is distinguished, so that a gross rock volume can be read against the map.
- `maturity_table`: candidates by readiness test, with the test word printed in every cell. The caption names the maturity with its reason.
- `volume_exceedance`: from a `volume_ranges` product, the share of sampled cases above each value, with p10, p50, p90 and the mean marked. The labels are non-exceedance, as the numbers are reported: p10 is the low case, exceeded by 90 % of the cases.
- `tornado`: from the sensitivity file of `pt_sensitivity`, one bar per input from its p10 to its p90, with the others at p50. The largest swing is on top.

## Standards

Discrete classes carry their words, never a colour alone. A closure is drawn from the cells that define it, not from a smoothed outline, so that its real extent is visible. The caption carries the products, the scenario and the validity record. Percentile labels stay non-exceedance on every figure.

## Protocol

1. Describe, then interpret: state the numbers that the tool returns (cells above the contact, the percentiles, the swings) before what they mean.
2. Figures give hypotheses, tools give numbers: never read an area, a depth or a coordinate off a map.

## Reporting

For each figure, add a `figures` entry (`url`, `caption`, `product`, `supports`: the claim ids it illustrates) and a `Figure: <caption> — <url> — supports <claim ids>` line in `conclusions`. If any figure was drawn, start the reply with each figure's `markdown` line and a one-line caption, then the findings JSON.

<!-- end of skill: prospect-visualisation -->
