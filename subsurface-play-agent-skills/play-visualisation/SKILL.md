---
name: play-visualisation
description: Status matrices, events charts, dependency graphs and segment maps. It covers the figures the tools draw, the standards they enforce (discrete classes, unknown hatched, no colour ramps, the weakest element outlined) and what a figure may support. Load before play_render and before you publish the figures the tools drew.
---

# Play visualisation


## What a figure is and is not

A figure illustrates a claim that a tool computed. It never carries a claim, and you never read a number, depth, area or boundary off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested. That is the one way a picture can damage an argument.

You cannot see the figures: a tool returns a URL, not an image. Check the numbers that the tool returns, never the picture. Figures are for the person, Geo Oracle's answer and the report. The caption says what the figure shows, from which product and version, well enough to be read without you.

## Figures the tools draw

- `play_status_matrix`: the status matrix and one dependency graph per play.
- `play_events_chart`: the events chart.
- `play_classify_outcomes`: the segment map, when the wells have x and y.

Each comes in `result.figures` with its `url`, `caption`, `product` and `markdown`. Publish every one.

## `play_render`

Draw a product that no tool drew, or an extra view the person asks for.


- `status_matrix`: plays as rows, elements as columns, the status word in every cell, unknown hatched, the weakest critical element outlined, supporting elements marked.
- `events_chart`: the dated events on a Ma axis with their sources, the critical moment as a line.
- `dependency_graph`: one play's critical elements coloured by status, arrows from prerequisite to dependent, the weakest outlined.
- `segment_map`: the wells from the outcomes file, posted by outcome class over the mapped closures that the classification used: supplied outlines as polygons, trap products as their closed cells (four-way and fault-bounded drawn apart). The class word is at every well. The play agent stores no play or segment outline, so the map shows none. Wells without x and y are listed, not drawn.

## Standards

The figures use discrete classes only. They use no continuous ramp, because a ramp reads as a probability. Unknown is drawn as its own pattern: no evidence is not bad evidence. Every figure's caption names the products and the tests.

## Protocol

1. Describe, then interpret: which element caps which play, before what to do about it.
2. Figures give hypotheses, and tools give statuses. A pattern on the matrix enters the findings only through the tool's result.

## Reporting

Publish a figure for every product that you publish and that has a renderer. For each, add a `figures` entry with `supports` (the claim ids it illustrates). Also add a `Figure: <caption> — <url> — supports <claim ids>` line in `conclusions`. Put each figure's `markdown` line at the top of your reply.

<!-- end of skill: play-visualisation -->
