---
name: prospect-definition
description: How closures become candidates, the readiness tests in detail, what each failure means, and the maturation plan. Load before pt_readiness.
---

# Prospect definition

## From a closure to a candidate

One closure is one candidate. A closure cut by a fault that the structural specialist found sealing is two candidates, not one. A closure with stacked reservoirs is one candidate per reservoir, each with its own elements and volume. Say which you have chosen and why.

## Reading the readiness result

| Test | Fails when | Not tested when | What it means |
|---|---|---|---|
| R1 | — | — | no play matrix covers it: a concept, whatever the closure looks like |
| R2 | the closure is in time, or not closed | the validity record does not show V1 and V7 passed | a lead; ask structural_geology for a trap_geometry from sg_closure_spill with model=<the structural_model on the same horizon>, so V1 and V7 are in its validity record |
| R3 | an element claim was refused by the ceiling | no elements inherited, or an element has no row in the play matrix and no basis (listed as `elements_untraced`) | lower the element or take the evidence to the play specialist; for an untraced element, ask the play specialist to assess it |
| R4 | — (`pt_resolve_inputs` refuses an input with no product or source, so no inputs file records a failure) | no inputs file is given | leave unset before volumes; give the inputs file once every input resolves; a refused input is `missing_data` with its owner |
| R5 | a contact is outside the closure, a column exceeds the seal, or no reservoir is present | no check passed; a fill-to-spill contact is not a test, because the spill depth stands in for the contact | the geological inconsistency is the finding, not an inconvenience |
| R6 | no location satisfies the constraints | no location was derived | a prospect, not a target |
| R7 | no scenario; a scenario without a basis; scenarios that do not differ in kind, description or contact | one scenario, or names without the `scenario_set` | give `scenario_set` from `pt_scenarios`: names alone do not show that the alternatives differ |

## The maturation plan

`pt_maturation_plan` turns every failing or untested test into an item, with the owning specialist and a request string in the shared format. Those items are the `missing_data` entries and the `recommended_followups`. It also draws the maturity table of every candidate it reads, in `result.figures`. Generic advice ("acquire more data") is never acceptable: the item names the data, the form and the test it would pass.

## Reporting

The maturity claim carries the tests. The inventory product carries every candidate with its rung and capping element. Geo Oracle plans the next wave from it.

<!-- end of skill: prospect-definition -->
