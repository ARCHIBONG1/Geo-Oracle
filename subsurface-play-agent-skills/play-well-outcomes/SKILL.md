---
name: play-well-outcomes
description: Classifying every well in a play area as a success, a failure on a named element or undetermined, using the operator's stated reason and the mapped closures, and what each outcome demonstrates. Load before play_classify_outcomes.
---

# Wells as tests

## Classification (`play_classify_outcomes`)

| Result | Class | Demonstrates | Failed element |
|---|---|---|---|
| discovery | success | every critical element of the play (hydrocarbon) | — |
| shows | partial | source, maturation, migration | from the stated reason if any |
| dry with a stated reason | failure | — | the reason mapped: trap (closure, structure), reservoir (tight, absent, facies), seal (leak, caprock), preservation (breach, biodegradation, flushing), migration (no shows, charge), timing |
| dry outside the mapped closure, no reason | failure | — | trap (from the closure outline) |
| dry without a usable reason, inside the closure | undetermined | — | — |
| not reached, mechanical | undetermined | — | the well did not test the play |

Give `closures` from `trap_geometry` in the wells' CRS; give `explanations` with cited evidence for wells the rules leave undetermined. PT5 fails while any well is undetermined.

## Reading the result

- The operator's reason is kept beside your classification; where the two disagree (a well called a charge failure that lies outside the closure), say so in `contradictions`.
- A failure on an element is evidence against that element in that well's area, not everywhere: it lowers the element locally and feeds the fairway (phase 2).
- Several failures on the same element across the area contradict the play; one failure with a reason does not.

## Reporting

The classification as `[interpretation, derived]` with the tool's provenance id; the outcome table as `[data, project_data]`; the product `well_outcome_analysis`.
