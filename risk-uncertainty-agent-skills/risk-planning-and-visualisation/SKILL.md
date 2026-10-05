---
name: risk-planning-and-visualisation
description: The reduction plan - actions with owners, modes and neutral framing, split into actionable now, needs acquisition and irreducible, ordered by conclusions settled - the follow-up after a reduction wave with every attempt reported, and the figures that never become a risk matrix. Load before ru_reduction_plan, ru_attempt_outcomes or ru_render.
---

# Planning, outcomes, figures

## The plan (`ru_reduction_plan`)

For every material entry: the action, the owning specialist, the analysis mode and a task framing.

- Mode is `challenge` (test a conclusion against its alternatives) or `validation` (audit a product or record); never `initial`, which re-asks the question.
- Framing is a test: "test whether the closure survives V1 and V7 against the alternative pick uncertainty". The tool refuses "confirm", "prove", "verify that", "show that", "establish that".
- `data_in_hand` says whether the owner can act now (a rerun, a different model on existing data) or the item needs acquisition (a record, a sample, a survey). State it; the tool infers it from the wording only as a fallback.
- An irreducible entry (oil or gas before drilling) is marked so with a reason and carried as a scenario.

The plan is ordered by actionable first, then the number of material conclusions each action settles, then shared before specific. The rule is printed in the sidecar and it is the nearest this agent comes to a ranking.

Geo Oracle runs the `actionable_now` items as one reduction wave in the modes named, and gives the `needs_acquisition` items to the user.

## The follow-up (`ru_attempt_outcomes`)

After the wave: the same flip tests rerun on the new products (an unchanged input answers from the record), a new `ru_materiality_report`, then `ru_attempt_outcomes` against the prior one with every attempt named: entry, task id, owner, mode, what was tried, why. Outcomes: settled, narrowed, unchanged, newly material. Every attempt is reported, succeeded or not; an unchanged outcome with a named attempt is a finding the user needs.

## Figures (`ru_render`)

- `register`: the entries as a table, words in every cell, undetermined hatched.
- `dependency_graph`: the claim graph with load-bearing, shared and stale nodes marked.
- `flip_diagram`: each test's conclusion under each plausible value side by side, flips in red, counts with their totals and the threshold.

No likelihood-impact grid and no heat map exist in this agent, because a coloured grid is a score whether or not one was computed.

## Reporting

The plan's items as `recommended_followups` in order, each with its mode and whether it is actionable now; the acquisition items also as `missing_data` in the shared format; attempts as `claims` in the statement form. `Figure: <caption> — <url>` in `conclusions`.
