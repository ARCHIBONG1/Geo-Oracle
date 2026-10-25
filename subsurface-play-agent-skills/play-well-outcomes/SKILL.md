---
name: play-well-outcomes
description: How to classify every well in a play area as a success, partial, a failure on a named element or undetermined. The classification uses the operator's stated reason, the trap's closed cells and your explanations. Also what each outcome demonstrates. Load before play_classify_outcomes.
---

# Wells as tests

## Classification (`play_classify_outcomes`)

| Result | Class | Demonstrates | Failed element |
|---|---|---|---|
| discovery | success | every critical element of the play (hydrocarbon) | — |
| shows | partial | source, maturation, migration | from the stated reason, when it names one element |
| dry with a stated reason | failure | — | the reason mapped: trap (closure, structure), reservoir (tight, absent, facies), seal (leak, caprock), preservation (breach, biodegradation, flushing), migration (no shows, charge), timing |
| dry with a reason naming several elements | undetermined | — | listed as `candidate_elements`; settle it with `explanations` `{element, reason}`, citing the evidence |
| dry outside every closure, no reason | failure | — | trap (from the trap's closed cells or a supplied outline) |
| dry without a usable reason, inside the closure | undetermined | — | — |
| not reached, mechanical | undetermined | — | the well did not test the play: give `explanations` `{element: "not a test", reason}`, and it leaves the success and failure counts |

Give the structural `trap_geometry` products as `traps`. The tool reads their closed cells: a well is inside when its nearest grid cell is closed. Give `closures` only for outlines supplied as polygons in the wells' CRS. For each well that the rules leave undetermined, give `explanations` as `{well: {element, reason}}`. `element` is one of the play's elements, or "not a test". A plain-string reason stays unsettled. PT5 fails while any well is unexplained or unsettled.

If the wells have x and y, the tool draws the segment map in `result.figures`. It shows each well by outcome class over the outlines you gave (it does not draw trap cells). Publish it.

## Reading the result

- The operator's reason stays beside your classification. Where the two disagree (a well called a charge failure that lies outside the closure), say so in `contradictions`.
- A failure on an element is evidence against that element in that well's area, not everywhere. It lowers the element locally, and it feeds the fairway (PT7, which no tool tests yet).
- Several failures on the same element across the area contradict the play. One failure with a reason does not.

## Reporting

Give the classification as `[interpretation, derived]` with the tool's provenance id, the outcome table as `[data, project_data]`, and the product `well_outcome_analysis`.

<!-- end of skill: play-well-outcomes -->
