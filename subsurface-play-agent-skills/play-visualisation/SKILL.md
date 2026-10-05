---
name: play-visualisation
description: Status matrices, events charts and dependency graphs - the standards the tool enforces (discrete classes, unknown hatched, no colour ramps, the weakest element outlined) and what a figure may support. Load before play_render.
---

# Play visualisation


## What a figure is and is not

A figure illustrates a claim a tool computed. It never carries a claim, and no number, depth, area or boundary is read off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested, which is the one way a picture can damage an argument.

You do not see the figures you render: the tool returns a URL, not an image. They are for the reader and for the audit, so the caption must say what the figure shows, from which product and version, well enough to be read without you.

## `play_render`

- `status_matrix`: plays as rows, elements as columns, the status word in every cell, unknown hatched, the weakest critical element outlined, supporting elements marked.
- `events_chart`: the dated events on a Ma axis with their sources, the critical moment as a line.
- `dependency_graph`: one play's critical elements coloured by status, arrows from prerequisite to dependent, the weakest outlined.

## Standards

Discrete classes only; no continuous ramp, because a ramp reads as a probability. Unknown is drawn as its own pattern: no evidence is not bad evidence. Every figure's caption names the products and the tests.

## Protocol

1. Describe, then interpret: which element caps which play, before what to do about it.
2. Figures give hypotheses, tools give statuses: a pattern on the matrix enters the findings only through the tool's result.

## Reporting

`Figure: <caption> — <url>` in `conclusions`.
