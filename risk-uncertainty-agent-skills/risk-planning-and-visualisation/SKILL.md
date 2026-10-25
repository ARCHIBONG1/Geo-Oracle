---
name: risk-planning-and-visualisation
description: The reduction plan — actions with owners, modes and neutral framing, split into actionable now, needs acquisition and irreducible, and ordered by actionable first and then conclusions settled. Also the follow-up after a reduction wave, with every attempt reported, and the figures that never become a risk matrix, including the ones the tools draw. Load before ru_reduction_plan, ru_attempt_outcomes or ru_render, and before you publish the figures the tools drew.
---

# Planning, outcomes, figures


## What a figure is and is not

A figure illustrates a claim that a tool computed. It never carries a claim, and you never read a number, depth, area or boundary off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested. That is the one way a picture can damage an argument.

You cannot see the figures: a tool returns a URL, not an image. Check the numbers that the tool returns, never the picture. Figures are for the person, Geo Oracle's answer and the report. The caption says what the figure shows, from which product and version, well enough to be read without you.

## The plan (`ru_reduction_plan`)

For every material entry: the action, the owning specialist, the analysis mode and a task framing.

- The mode is one of these:
  - `challenge` (test a conclusion against its alternatives);
  - `validation` (audit a product or record);
  - `follow_up` (the owner continues in its own session and builds on its earlier results).

  It is never `initial`, which re-asks the question. If you give no mode, the tool uses `challenge`.
- Framing is a test: "test whether the closure survives V1 and V7 against the alternative pick uncertainty". The tool refuses "confirm", "prove", "verify that", "show that", "establish that".
- `data_in_hand` says whether the owner can act now (a rerun, a different model on existing data) or the item needs acquisition (a record, a sample, a survey). State it. The tool infers it from the wording only as a fallback.
- Mark an irreducible entry (oil or gas before drilling) as irreducible, with a reason. It is carried as a scenario.

The tool orders the plan: actionable first, then the number of material conclusions each action settles, then shared before specific. The sidecar prints the rule. It is the nearest this agent comes to a ranking.

Geo Oracle runs the `actionable_now` items as one reduction wave in the modes named, and gives the `needs_acquisition` items to the user.

## The follow-up (`ru_attempt_outcomes`)

After the wave, in this order:
- rerun the same flip tests on the new products (an unchanged input answers from the record);
- run a new `ru_materiality_report`;
- then run `ru_attempt_outcomes` against the prior one, with every attempt named: entry, task id, owner, mode, what was tried, why.

Outcomes: settled, settled as a failure, narrowed, unchanged, newly material, overturned (a settled failure no longer fails), newly a settled failure, changed, entry withdrawn (the entry is gone from the current register). Report every attempt, succeeded or not. An unchanged outcome with a named attempt is a finding that the user needs.

## Figures the tools draw

- `ru_merge`: the register.
- `ru_dependencies`: the claim graph.
- `ru_materiality_report`: the register with its materiality classes, and one flip diagram of the tests it was given.

Each comes in `result.figures` with its `url`, `caption`, `product` and `markdown`. Publish every one.

## Figures (`ru_render`)

Draw a product that no tool drew, such as one flip test on its own, or an extra view the person asks for.

- `register`: the entries as a table, words in every cell, undetermined hatched.
- `dependency_graph`: the claim graph with load-bearing, shared and stale nodes marked.
- `flip_diagram`: each test's conclusion under each plausible value side by side, flips in red, counts with their totals and the threshold.

This agent has no likelihood-impact grid and no heat map, because a coloured grid is a score whether or not one was computed.

## Reporting

Give the plan's items as `recommended_followups` in order, each with its mode and whether it is actionable now. Also give the acquisition items as `missing_data` in the shared format. Give attempts as `claims` in the statement form. Publish a figure for every product that you publish and that has a renderer. For each, add a `figures` entry with `supports` (the claim ids it illustrates). Also add a `Figure: <caption> — <url> — supports <claim ids>` line in `conclusions`.

<!-- end of skill: risk-planning-and-visualisation -->
