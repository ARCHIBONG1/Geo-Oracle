---
name: geo-play-prospect-evaluation
description: Gates, element checklists, the element status scale and the maturity ladder for Geo Oracle's play, prospect/target and risk stages. It covers when the integrated model is ready for subsurface_play, prospect_target and risk_uncertainty; what each call must be given; the five domains those specialists assess (hydrocarbon, CO2 storage, geothermal, natural hydrogen, hydrogen storage); how element statuses are set and checked; the risk brief and the reduction loop; and how to route critical uncertainties back to specialists. Load before calling subsurface_play, prospect_target or risk_uncertainty, when the user asks about prospectivity, plays, leads, prospects, drilling targets, chance of success (which this system does not produce) or risk; and when you review their results.
---

# Plays, prospects and risk

These are the stages where overstatement does the most damage. A prospect inherits every weakness of the model beneath it, so each stage has a gate.

## 1. The domain fixes the elements

`subsurface_play` and `prospect_target` assess five domains, each with a fixed element template. `play_element_template` returns the template, with the evidence that would lift each element. Use these names in briefs. The specialists link evidence to these names and to no others.

| Domain | Critical elements | Supporting |
|---|---|---|
| Hydrocarbon | source, maturation, migration, reservoir, seal, trap, timing, preservation | overburden |
| CO2 storage | storage_unit, injectivity, trapping_mechanism, caprock_containment, fault_containment, well_containment, pressure_limit | monitorability |
| Geothermal | heat, permeability, fluid_recharge, structural_setting | cap_seal, stress_for_stimulation |
| Natural hydrogen | source, generation_activity, migration, reservoir, seal, trap, preservation | associated_gases |
| Hydrogen storage | storage_unit, injectivity_deliverability, caprock_containment, fault_containment, well_containment, geochemical_microbial_stability, cycling_integrity | cushion_gas, monitorability |

**Any other resource** (mineral systems, helium, lithium brines) has no template, and `subsurface_play` refuses the domain. Tell the person so before you plan. Regional, literature and the data specialists can still describe the setting. But neither you nor any specialist can produce a play matrix, maturity or materiality for it.

## 2. The element status scale

`subsurface_play` sets every element's status from its evidence links. You check the status; never assign it. The scale is ordinal, with no number attached:

| Status | Needs |
|---|---|
| demonstrated | study-area observations or measurements |
| inferred | study-area evidence through a stated reasoning step, or a direct line of partial status |
| analogue-supported | regional or analogue evidence only: it never demonstrates |
| hypothesised | proposed, not yet supported |
| unknown | no evidence either way |
| disfavoured | evidence against that does not establish failure: a study-area interpretation or inference against, or regional or analogue evidence against, with no better evidence for the element |
| contradicted | study-area observations or measurements against: the element fails |

demonstrated > inferred > analogue-supported > hypothesised > unknown > disfavoured, and contradicted fails the element. The play takes its weakest critical element's status. Published evidence counts by its scope and kind, like any other evidence. A published measurement from the study area is study-area evidence. A regional synthesis or another basin is regional or analogue. Better evidence for an element outranks weaker evidence against it. The outranked evidence is listed as `conflicting_evidence`.

A play can remain a viable hypothesis while several of its elements are unknown. But it cannot be described as supported on the strength of analogy or a basin's reputation.

## 3. Gate before `subsurface_play`

**Full evaluation.** Run it when all of the following hold:
- the framework the play depends on exists: stratigraphy and structure, or their equivalents for the domain;
- reservoir and seal evidence has been reviewed;
- material contradictions are recorded in the ledger.

**Screening.** Earlier in the investigation, you may run it explicitly as a screening, to identify which elements lack evidence. Set `required_outputs` accordingly, and label the result "screening".

**After a screening.** A screening is a statement about the evidence at the time it ran. When a result that the screening lacked arrives later (the trap, the reservoir, the seal or the charge evidence), run the evaluation again before you close the investigation. Send it as `reanalysis`, with `previous_task_id` set to the screening task and the new result in `upstream_task_ids`. A risk result that used the screening needs the same rerun. If you do not run it again, record in the ledger why the screening result still stands.

**Brief contents.** Pass the following:
- the domain (one of the five) and the play types in scope;
- the per-element findings, as `upstream_findings` with global ids, classes and their `[regional]` or `[analogue]` scope tags. Include the findings against an element;
- the competing interpretations;
- the contradictions from the ledger;
- `known_constraints` such as "Regional analogues are not local evidence". Do not ask for probabilities: the play, prospect and risk specialists refuse them.

**Result review.** Check that:
- each element's status matches the evidence ids cited and the scope rule;
- evidence against is reported: every disfavoured element, and any `conflicting_evidence`, as a contradiction in the findings;
- timing has been evaluated as a relationship between two dated events, not assumed;
- the specialist has not generated prospects: that is not its job;
- the specialist has identified the evidence that would discriminate between competing plays.

## 4. Gate before `prospect_target`

All of the following must hold:
- the play concept has been evaluated, and its `element_status_matrix` is recorded;
- each element the target relies on has traceable support, even if that support is "hypothesised";
- the trap geometry comes from structural or seismic results in depth, with the depth-conversion status stated;
- no element would have to be filled by an assumption that is absent from the integrated model.

If the gate fails, do not call the tool just because prospects were requested. Tell the user which elements are missing and what would supply them.

### Maturity

`prospect_target` places each candidate on one ladder, set by its readiness tests R1-R7 (`pt_readiness`). Report that rung:

| Rung | Basis |
|---|---|
| concept | no assessed play to inherit from (R1 fails) |
| lead | an assessed play, but the trap in depth, element tracing, input provenance or consistency (R2-R5) not all passing |
| prospect | R1-R5 pass; a derived location (R6) or the alternatives (R7) still missing |
| target | R1-R7 pass |

A lead gets no volume. Volumes are in place only. Never upgrade maturity because a feature has a name, a map or a volume. Never overrule the rung the tool set.

## 5. Risk and uncertainty

Risk runs after the play and prospect waves. This is the one list of what its brief carries.

**Brief contents.** Pass `risk_uncertainty` the following:
- the ledger's absolute sandbox path, as `available_evidence` of kind `investigation_ledger`: the path you gave `set_hotl_mode` as `ledger_path`. Do not build it again from `pwd`. The gateway adds the path on record when the brief has none, and corrects a path that differs. The risk specialist reads the ledger directly, so update it before the call;
- every finding relied on, as `upstream_findings` with full ids, classification, status, `[regional]`/`[analogue]` tags and `depends_on`;
- the plays and targets, with their element statuses and the products behind them;
- every product, by reference;
- the contradictions and uncertainties from the ledger;
- the objectives in scope, by name, and the decision context.

It returns the register, the materiality report and a `reduction_plan`. The plan splits its items into `actionable_now` and `needs_acquisition`, each with an owner, an `analysis_mode` and a `task_framing`. The plan also lists:
- `irreducible` entries, with the reason;
- `settled_failures`: conclusions decided against in every case, which no reduction can change. They are not sent for reduction. The only checks on them are their threshold's source and the basis of their inputs.

**The reduction loop.** The investigation does not end at its first answer:
1. **Reduction wave.** Run every `actionable_now` item as one wave. Send each item as a task to the named owner, in the named mode (`challenge` or `validation`, never `initial`). Use the item's `task_framing` as the geological question, unchanged. It is framed as a test. The one thing that would make the loop harmful is to reframe it as a confirmation. Record each task in the ledger's *Reduction attempts* table: entry, task id, owner, mode, what it tries, why. If a human-on-the-loop setting is on, present the plan first, and run only what the human approves. If it is off, run the wave automatically. The `needs_acquisition` items are not tasks. They go to the user in the final answer.
2. **Follow-up.** Task `risk_uncertainty` again in `follow_up` mode, with the new products, the prior materiality file and every attempt, copied from the ledger's *Reduction attempts* table (the risk tools do not read that table). It reruns the same flip tests and returns `reduction_outcomes`. For each attempt, these give what was tried, why, and what happened to the uncertainty: settled, settled as a failure (every case now fails), narrowed, unchanged, newly material, overturned (a settled failure no longer fails), newly a settled failure, changed, or entry withdrawn (the entry is gone from the current register).
3. **One wave by default.** Run a second reduction wave only when the user asks. Never reissue an attempt already recorded in the ledger. Report an unchanged outcome; do not retry it.
4. **Report every attempt.** The final answer says what was tried, why, and what happened, whichever way it went. A reduction that did not settle its uncertainty is one of the more useful things the investigation can report. The acquisition items are its recommendations.

**No numerical chance of success.** The play, prospect and risk specialists refuse probabilities, chance factors, scores and risk matrices, whatever methodology is supplied. So never request one. Ask instead for materiality:
- which uncertainties flip a categorical conclusion (`material`);
- which leave it failing in every case (`settled_failure`);
- which only move a range (`contributing`);
- which change nothing when tested (`immaterial`);
- which could not be tested, and why (`undetermined`);
- what would reduce each.

If the person asks for a chance of success, say in one sentence that this system does not produce one. Give them the element statuses, the material entries and the settled failures. Their own risking method can take these up outside the system.

**Base case of a flip test.** A flip test of `risk_uncertainty` computes the play status or the maturity again before it changes an input. That base case must agree with the result of the specialist that owns it. If the risk result reports a different base case, or refuses a test because the candidate cannot be rebuilt, do not use the materiality class of that test. Ask the owner (`subsurface_play` or `prospect_target`) to publish the product again, then ask `risk_uncertainty` to run the test again.

**A changed play element.** When a reduction task changes the status of a play element, the prospect result that inherited the element is out of date. Plan `prospect_target` again before the risk follow-up.

**Routing.** Send each critical, reducible uncertainty back to the specialist who can reduce it:

| Uncertainty | Route to |
|---|---|
| Seismic imaging, amplitude meaning, depth conversion | `seismic_interpretation` |
| Fault geometry, fault seal, closure | `structural_geology` |
| Correlation, unit identity, ages | `stratigraphy` |
| Reservoir presence and distribution, facies | `sedimentology` |
| Rock and fluid properties, log QC | `wells_petrophysics` |
| Regional controls, analogue applicability | `regional_geology` or `literature_review` |
| Play-element support | `subsurface_play` |

Record each routed uncertainty in the ledger (`U#`), with its task. Do not route a settled failure for reduction: check its threshold's source and the basis of its inputs instead.

## 6. Circularity checks

Before accepting play, prospect or risk conclusions, confirm that none of the following has happened:
- a prospect or play conclusion was used to justify the assumptions it depends on;
- an analogue-supported element was later treated as demonstrated;
- evidence against an element (a disfavoured element, or `conflicting_evidence` outranked by better evidence) was left out of the report;
- risk was lowered by dropping an alternative interpretation without evidence;
- several elements were counted as supported by what is really one line of evidence, such as a single seismic interpretation;
- a prospect was matured while its underlying hypothesis was unresolved or contradicted.

<!-- end of skill: geo-play-prospect-evaluation -->
