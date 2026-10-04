---
name: prospect-definition
description: Turning closures into candidates, the readiness tests in detail, what each failure means, and the maturation plan. Load before pt_readiness.
---

# Prospect definition

## From a closure to a candidate

One closure is one candidate. A closure cut by a fault the structural specialist found sealing is two candidates, not one; a closure with stacked reservoirs is one candidate per reservoir, each with its own elements and volume. Say which you have chosen and why.

## Reading the readiness result

| Test | Fails when | Not tested when | What it means |
|---|---|---|---|
| R1 | — | — | no play matrix covers it: a concept, whatever the closure looks like |
| R2 | the closure is in time, or not closed | the validity record does not show V1 and V7 passed | a lead; the request goes to structural_geology |
| R3 | an element claim was refused by the ceiling | no elements inherited | lower the element or take the evidence to the play specialist |
| R4 | an input has no product or source | the volume step has not run | leave unset before volumes; set it only once every input resolves |
| R5 | a contact is outside the closure, a column exceeds the seal, or no reservoir is present | nothing was checked | the geological inconsistency is the finding, not an inconvenience |
| R6 | no location satisfies the constraints | no location was derived | a prospect, not a target |
| R7 | fewer than two scenarios | one scenario | a single scenario means the alternatives were not tested |

## The maturation plan

`pt_maturation_plan` turns every failing or untested test into an item with the owning specialist and a request string in the shared format. Those items are the `missing_data` entries and the `recommended_followups`. Generic advice ("acquire more data") is never acceptable: the item names the data, the form and the test it would pass.

## Reporting

The maturity claim carries the tests. The inventory product carries every candidate with its rung and capping element, which is what Geo Oracle plans the next wave from.
