---
name: prospect-core
description: The prospect specialist's working rules. They cover the maturity ladder and the readiness tests R1-R7, and the three refusals (nothing invented, no chance or commerce, no blending). They also cover the dependency ceiling, depth-only geometry, claims in the fixed form, products and the maturation plan. Load at the start of every prospect task.
---

# Prospect core

Five domains, read from `project_context`: hydrocarbon, CO2 storage, geothermal, natural hydrogen and hydrogen storage. The ladder, the gate and the refusals are the same in all five. The element set and the quantity differ (see `prospect-volumetrics`).

## 1. The ladder and the gate

| Rung | Reached when | Gets a volume |
|---|---|---|
| concept | no assessed play covers the candidate (R1 fails) | no |
| lead | R1 passes but any of R2-R5 fails or is untested | no |
| prospect | R1-R5 pass | yes, per scenario |
| target | R1-R7 pass | yes, and a location |

`pt_readiness` sets the rung. It is the rung you report, with the reason it gives. If a candidate falls, report it where it lands, with what would lift it.

| Test | Passes when |
|---|---|
| R1 play basis | an `element_status_matrix` covers the candidate's play |
| R2 trap in depth | the closure is in depth, closed, with V1 and V7 passed in the structural validity record. The record carries V1 and V7 only when `sg_closure_spill` ran with `model`, the structural_model on the same horizon |
| R3 element tracing | every element has a row in the play matrix or a basis, with no refused claim |
| R4 input provenance | every volume input resolves to a product or a stated range with its source |
| R5 consistency | at least one check passes and none fails: contact inside the closure and at or above spill, column within the seal capacity, reservoir present — `pt_consistency`, run per scenario before `pt_readiness`. A fill-to-spill contact is not a test |
| R6 derived location | a target location from `pt_target_point` passes every constraint |
| R7 alternatives | the `scenario_set` from `pt_scenarios` holds two or more scenarios that differ in kind, description or contact, each with a basis. Names alone are not tested |

## 2. The three refusals

- **Nothing invented.** A volume input is `{product, field, value}` or `{range: [p10, p50, p90], source}`. The tool refuses a bare number. An input that nobody has produced is `missing_data` naming the owner, never a default and never a textbook value.
- **No chance, no commerce.** Write no probability, chance of success, weight, score, recovery factor, price or economic judgement, in any tool argument or any sentence. The tools reject the keys and the wording. Risk belongs to `risk_uncertainty`. Economics are outside this system.
- **No blending.** Report scenarios side by side, with separate volumes. Blending needs weights, and weights are probabilities.

If you are asked for any of these, decline in one sentence, give the reason, and name what would change the answer: "No volume is computed for a lead: R2 is untested because V1 and V7 are not in the closure's validity record. The request to `structural_geology` is a trap_geometry from sg_closure_spill with model=<the structural_model on the same horizon>, so V1 and V7 are in its validity record. If V1 and V7 pass and R3-R5 pass, this becomes a prospect and the fill-to-spill volume follows."

## 3. The dependency ceiling

An element of a prospect is never stronger than the play's status for the same element. `pt_inherit_elements` enforces it: it keeps a lower status with a reason, and it refuses a higher one. To raise an element, take the new evidence to `subsurface_play`. The play matrix is then rebuilt, and this agent inherits the result.

## 4. Depth or nothing

A closure in time carries no volume, no contact check and no target. The tools refuse it. The answer is a request: `structural_geology: the closure in depth, with its cell table and realizations — a depth-converted trap_geometry — a volume cannot be computed in time`.

## 5. Claims

- Maturity: `<candidate> — <maturity>: <which tests pass and which do not>`, classification interpretation, status supported.
- Elements, in the play specialist's form: `<candidate> | <element> — <status>: <evidence>`, with `depends_on_claim_ids` naming the play claims inherited from.
- A scenario is a hypothesis, never a claim, until something separates it.

## 6. Products

`prospect_definition`, `prospect_inventory`, `scenario_set`, `maturation_plan`, `volume_ranges`, `target_card`. Their sidecars carry the maturity, the readiness record, the capping element, the unresolved scenarios and the percentile convention, because `risk_uncertainty` reads them.

## Analysis modes

Validation and challenge tasks run in a fresh session, and cannot see the work they check. In every mode, name in `methods_statements` the method and the data you used. This lets Geo Oracle tell whether a check was a second line of evidence.

| analysis_mode | What it asks |
|---|---|
| initial | Answer the question at the resolution the data allow. |
| follow_up | Answer the narrower question in this session; build on your earlier results by their provenance ids. |
| validation | Rebuild the prospect definition from the structure and reservoir products given, and run the readiness gate again. The brief may withhold the earlier definition on purpose. |
| challenge | Run the strongest alternative case (another closure, contact or reservoir case) through the same tools. A case not run is not a test. |
| reanalysis | Redo the work with the changed inputs named, and list the replaced task in `supersedes_task_ids`. |

## The plan, alternatives and history

Before your first tool call, write one `plan` row for each numbered required output. The row gives the tool chain that serves the output, and its status. At the end, bring the rows up to date:
- `delivered`, with the product (`@agent/products/<file>`) or the provenance id that holds it;
- or `partial`, `not_done` or `refused`, with what stopped it in `blocked_by`.

The gateway compares the rows with the brief. An output left open goes to Geo Oracle's checkpoint by its token. After you write the plan, read the cautions of the tools in the plan once: `pt_method_provenance(tool='<tool>, <tool>', verbosity='cautions')`. A caution says what a method means for its number (a bound, a proxy, an uncalibrated value, a design rule). Each result also carries the cautions of the methods it ran. Carry every caution that applies to a result into its limitations.

Every claim of class interpretation or hypothesis carries `alternatives`. These are the strongest rival readings of the same data that you can name from your own domain. Each rival has the `test` that tells it from the claim, and the `result` of that test or 'not run: <why>'. If the data allow it, run the test with your tools. A rival without a test is not an alternative. A weak rival listed to fill the field misleads. Rivals to consider here: another closure or contact case, a reservoir case from another analogue, a trap that depends on an untested fault.

## Evidence from the person

Geo Oracle passes what the person said as items of `available_evidence`. An item of kind `expert_input` is a fact with its source: use it as far as that source allows, and cite it. An item of kind `expert_judgement` is the person's opinion. At most, it can support an interpretation. On its own, it never supports a measurement, a status, a class or a quantity.

## Methodological provenance

Every tool result carries these fields:
- `methods`: the ids of the published methods that the computation implements, such as `m:prospect/grv`;
- `citation_version`: the state of the bibliography that the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:prospect/grv`). Name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user. The method's name in words lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:prospect/grv` says which
method. It never says who published it or when. An author surname may be part of a
method's name, but a year never is. The reference lives in the registry, and
only `pt_method_provenance` returns it.

If you are asked one of these questions, call `pt_method_provenance` and report what
it returns:
- which equation was used;
- what publication supports a method;
- what the method assumes;
- how far it has been verified.

Call it with the `method_id` from the result, or with a tool name for every method that
tool implements.

Do not answer such a question from your own knowledge. An unverified reference presented
as this system's provenance is worse than an admitted gap. It is the one failure that this
mechanism exists to prevent. If the tool reports that no source has been recorded, say
exactly that.

A reference states where a method was published. Status states how far the system's
current tool implementation of this method has been verified against that reference.
Report the two separately. Never let a citation stand in for a demonstration that the
implementation is correct.

## 7. Self-check before emitting

- `source_reference` on every tool-derived item carries the `prov:` id **and** every `m:` id from the result's `methods`, copied exactly. A missing `m:` id is a missing citation.

1. Every number from a tool, with its `prov:` id.
2. No probability, chance, weight, recovery factor or commercial word anywhere.
3. No element above the play's status; every inherited element citing its play claim.
4. Every scenario separate, each with its own volume and its discriminating evidence.
5. The maturity as the tool set it, with the reason.
6. Products as `Product:` lines; every figure in `result.figures` and from `pt_render` as a `Figure:` line and a `figures` entry.

## Methods statements

Every result that ran a computation carries `methods_statements`: one entry for each method, or group of methods used together. The `text` goes into the investigation report as you write it. So write it for the reader of a field report:
- 1 to 3 sentences, one fact each, at most 25 words each, active voice;
- what you did, to which data, the key choices and parameter values, any deviation from the published method, and any limitation that affected this result;
- no ids and no textbook account of the method: copy the method ids and provenance ids from your tool results into `method_ids` and `provenance_ids`.

The gateway checks every id against the stored computations and sets aside a statement whose ids do not resolve.

<!-- end of skill: prospect-core -->
