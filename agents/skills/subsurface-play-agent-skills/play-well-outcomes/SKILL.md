---
name: play-well-outcomes
description: How to settle every well in a play area as a success, partial, a failure on a named element or not a test. The tool proposes candidate elements from the operator's stated reason and the trap's closed cells; your explanations settle each dry well. Also what each outcome demonstrates. Load before play_classify_outcomes.
---

# Wells as tests

## Classification (`play_classify_outcomes`)

The tool proposes; you settle. It never classifies a dry well as a failure: it reads candidate elements from the stated reason, and you decide.

| Result | Class | Demonstrates | Failed element |
|---|---|---|---|
| discovery | success | what the well shows at that well (`demonstrates`). A discovery well inside the closure may be linked as a study-area observation of reservoir, seal and trap. Such links make those elements demonstrated in the matrix, which is correct for the area the well tests: state in the link `note` that the observation is at one well. The charge chain (source, maturation, migration, timing) is lifted separately: a discovery lifts the charge chain to inferred for its play through `discoveries` `{play: [wells]}` | — |
| shows | partial | source, maturation, migration | none from the tool; you may give one with `explanations` `{well: {element, reason}}` |
| dry (D&A, water wet, tight and the like) | undetermined until you settle it | — | yours: `explanations` `{well: {element, reason}}` citing evidence, or `{well: {element: "not a test", reason}}` |
| not reached, mechanical | undetermined | — | the well did not test the play: give `{well: {element: "not a test", reason}}`, and it leaves the success and failure counts |

The tool reads the result without case. Whole phrases such as "oil discovery" or "gas producer" are successes. D&A, dry and abandoned, water wet and wet count as dry. A plugging status alone is not an outcome (discovery wells are plugged too): P&A or abandoned leaves the well undetermined. Any word the tool does not know is listed in `unrecognised_results`. Such a well is missing data, not a failed test: PT5 reads not tested for it. Put the well's outcome in the table and register it again, or, when the cell states it in other words, give `explanations` `{well: {result, quote}}`: `result` is one of the tool's words and `quote` is the words of the cell that state it.

What the tool proposes for a dry or shows well:
- `candidate_elements`: the elements the stated reason reads as failed, by keyword. A keyword counts only in a negative clause ("no", "not", "absent", "poor", "thin", "leaking" or a failure word such as "tight"). Negated or all-clear wording ("not breached", "ruled out", "seal not an issue") counts nothing. For a dry well outside every closure whose reason names no failed element, the candidate is trap (trapping_mechanism for CO2 storage).
- `mentioned_elements`: every element the reason mentions, failed or not.
- `not_tested_wording`: the reason says the element was not tested at the well ("not penetrated", "did not reach the target", "no logs").
- `hedged`: the reason is hedged ("may", "likely", "suspected").

`proposed` lists each proposal with its basis (keyword or outline), and one warning names every well still to settle with what is proposed. The reading is by keyword, so it can be wrong: an all-clear or a hedged reason can still give a candidate. Check each against the reason and the evidence.

Settle every dry well that tests the play with `explanations` `{well: {element, reason}}`: `element` is one of the play's elements, or "not a test", and `reason` cites the evidence. The well name is the key, for example `{"W-2": {"element": "seal", "reason": "E4: water-wet sand below a silty top seal"}, "W-5": {"element": "not a test", "reason": "E7: mechanical failure above the target"}}`. A plain-string reason keeps a dry well unsettled. PT5 fails while any dry, mechanical or not-reached well is not settled. A well whose result the table does not state as an outcome cannot be settled: give its outcome in the table and register the table again. If the evidence does not state the outcome, leave the well unsettled and name it in `missing_data`. Never write an outcome that you do not have. Give "not a test" yourself; the tool never sets it. Give `traps` (the structural `trap_geometry` products: their closed cells are read; a well is inside when its nearest grid cell is closed) or `closures` (outlines supplied as polygons in the wells' CRS).

If the wells have x and y, the tool draws the segment map in `result.figures`. It shows each well by outcome class over the closures the classification used: the outlines you gave, and the closed cells of the trap products. Publish it.

## Reading the result

- The operator's reason stays beside your classification. Where the two disagree (a well called a charge failure that lies outside the closure), say so in `contradictions`.
- A failure on an element is evidence against that element in that well's area, not everywhere. It lowers the element locally, and it feeds the fairway (PT7, which no tool tests yet).
- Several failures on the same element across the area contradict the play. One failure with a reason does not.

## Reporting

Give the classification as `[interpretation, derived]` with the tool's provenance id, the outcome table as `[data, project_data]`, and the product `well_outcome_analysis`.

<!-- end of skill: play-well-outcomes -->
