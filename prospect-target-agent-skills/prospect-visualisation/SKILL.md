---
name: prospect-visualisation
description: Closure maps and maturity tables - what the tool enforces and what a figure may support. Load before pt_render.
---

# Prospect visualisation


## What a figure is and is not

A figure illustrates a claim a tool computed. It never carries a claim, and no number, depth, area or boundary is read off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested, which is the one way a picture can damage an argument.

You do not see the figures you render: the tool returns a URL, not an image. They are for the reader and for the audit, so the caption must say what the figure shows, from which product and version, well enough to be read without you.

## `pt_render`

- `closure_map`: the closure drawn from its own depth cells, with the crest, the spill point and the scenario's contact marked and the area above the contact distinguished, so a gross rock volume can be read against the map.
- `maturity_table`: candidates by readiness test, the test word printed in every cell, the maturity named with its reason in the caption.

## Standards

Discrete classes with their words; never a colour alone. A closure is drawn from the cells that define it, not from a smoothed outline, so its real extent is visible. The caption carries the products, the scenario and the validity record.

## Protocol

1. Describe, then interpret: which cells lie above the contact, before what that means for the volume.
2. Figures give hypotheses, tools give numbers: never read an area, a depth or a coordinate off a map.

## Reporting

`Figure: <caption> — <url>` in `conclusions`.
