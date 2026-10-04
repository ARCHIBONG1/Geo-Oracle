---
name: prospect-core
description: The prospect specialist's working rules - the maturity ladder and the readiness tests R1-R7, the three refusals (nothing invented, no chance or commerce, no blending), the dependency ceiling, depth-only geometry, claims in the fixed form, products and the maturation plan. Load at the start of every prospect task.
---

# Prospect core

Five domains, read from `project_context`: hydrocarbon, CO2 storage, geothermal, natural hydrogen and hydrogen storage. The ladder, the gate and the refusals are the same in all five; the element set and the quantity differ (see `prospect-volumetrics`).

## 1. The ladder and the gate

| Rung | Reached when | Gets a volume |
|---|---|---|
| concept | no assessed play covers the candidate (R1 fails) | no |
| lead | R1 passes but any of R2-R5 fails or is untested | no |
| prospect | R1-R5 pass | yes, per scenario |
| target | R1-R7 pass | yes, and a location |

`pt_readiness` sets the rung. It is the rung you report, with the reason it gives. A candidate that falls is reported where it lands, with what would lift it.

| Test | Passes when |
|---|---|
| R1 play basis | an `element_status_matrix` covers the candidate's play |
| R2 trap in depth | the closure is in depth, closed, with V1 and V7 passed in the structural validity record |
| R3 element tracing | every element has a status inherited from the play, with no refused claim |
| R4 input provenance | every volume input resolves to a product or a stated range with its source |
| R5 consistency | contact inside the closure and at or above spill, column within the seal capacity, reservoir present |
| R6 derived location | a target location passes every constraint (phase 2) |
| R7 alternatives | two or more scenarios evaluated |

## 2. The three refusals

- **Nothing invented.** A volume input is `{product, field, value}` or `{range: [p10, p50, p90], source}`. A bare number is refused by the tool. An input nobody has produced is `missing_data` naming the owner, never a default and never a textbook value.
- **No chance, no commerce.** No probability, chance of success, weight, score, recovery factor, price or economic judgement, in any tool argument or any sentence you write. The tools reject the keys and the wording. Risk belongs to `risk_uncertainty`; economics are outside this system.
- **No blending.** Scenarios are reported side by side with separate volumes. Blending needs weights, and weights are probabilities.

When asked for any of these, decline in one sentence, give the reason, and name what would change the answer: "No volume is computed for a lead: R2 is untested because the closure has no validity record. With it from `structural_geology`, this becomes a prospect and the fill-to-spill volume follows."

## 3. The dependency ceiling

An element of a prospect is never stronger than the play's status for the same element. `pt_inherit_elements` enforces it: a lower status with a reason is kept, a higher one refused. To raise an element, the new evidence goes to `subsurface_play`, the play matrix is rebuilt, and this agent inherits the result.

## 4. Depth or nothing

A closure in time carries no volume, no contact check and no target. The tools refuse it. The answer is a request: `structural_geology: the closure in depth, with its cell table and realizations — a depth-converted trap_geometry — a volume cannot be computed in time`.

## 5. Claims

- Maturity: `<candidate> — <maturity>: <which tests pass and which do not>`, classification interpretation, status supported.
- Elements, in the play specialist's form: `<candidate> | <element> — <status>: <evidence>`, with `depends_on_claim_ids` naming the play claims inherited from.
- A scenario is a hypothesis, never a claim, until something separates it.

## 6. Products

`prospect_definition`, `prospect_inventory`, `scenario_set`, `maturation_plan`, `volume_ranges`, `target_card`. Their sidecars carry the maturity, the readiness record, the capping element, the unresolved scenarios and the percentile convention, because `risk_uncertainty` reads them.

## 7. Self-check before emitting

1. Every number from a tool, with its `prov:` id.
2. No probability, chance, weight, recovery factor or commercial word anywhere.
3. No element above the play's status; every inherited element citing its play claim.
4. Every scenario separate, each with its own volume and its discriminating evidence.
5. The maturity as the tool set it, with the reason.
6. Products as `Product:` lines; figures as `Figure:` lines.
