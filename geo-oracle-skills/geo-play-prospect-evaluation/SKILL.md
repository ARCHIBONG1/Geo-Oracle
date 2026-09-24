---
name: geo-play-prospect-evaluation
description: Gates, element checklists, support labels and maturity rules for Geo Oracle's play, prospect/target and risk stages. It covers when the integrated model is ready for subsurface_play, prospect_target and risk_uncertainty, what each call must be given, how to judge element support for hydrocarbon, CO2-storage, geothermal and mineral systems, and how to route critical uncertainties back to specialists. Use before calling any of those three tools, when the user asks about prospectivity, plays, leads, prospects, drilling targets, chance of success or risk, and when reviewing their results.
---

# Plays, prospects and risk

These are the stages where overstatement does the most damage. A prospect inherits every weakness of the model beneath it, so each stage has a gate.

## 1. The resource type determines the elements

Use the project's own element definitions if it has them. Otherwise, start from these:

| Resource | Elements to evaluate |
|---|---|
| Conventional hydrocarbons | **Source:** presence, richness, kerogen type, maturity. **Reservoir:** presence, quality, distribution. **Seal:** top and lateral seal, capacity, integrity, fault seal. **Trap:** geometry, closure, spill point. **Charge and migration:** pathways, volume, focusing. **Timing:** trap formation relative to generation and migration. **Preservation:** leakage, biodegradation, uplift and tilting. |
| CO₂ storage | **Storage formation:** capacity and injectivity. **Primary and secondary seals:** containment, continuity, capillary entry pressure. **Trap or containment geometry.** **Faults and legacy wells** as leakage paths. **Pressure regime:** fracture pressure and pressure build-up. **Monitoring feasibility.** |
| Geothermal | **Heat:** gradient, heat flow, heat source. **Reservoir permeability:** matrix, fracture or fault-controlled. **Fluid:** presence, recharge, chemistry and scaling. **Cap rock or insulating cover.** **Structural controls.** |
| Mineral systems | **Source:** fluids, metals, energy. **Pathways:** crustal architecture, conduits. **Trap:** physical or chemical depositional site. **Preservation:** post-mineralisation modification, cover. **Geodynamic setting.** |
| Other (e.g. hydrogen, helium, lithium brines) | Agree the element set with the user before play analysis. |

## 2. Support labels

Label every element of every candidate play with one of these, and cite the ids behind it.

| Label | Meaning |
|---|---|
| demonstrated | Shown by project evidence in or near the play area, e.g. a well penetrating a reservoir with measured properties |
| supported by literature | Published evidence for this area or interval; not project data |
| supported by analogy | Inferred from a comparable setting elsewhere; not local evidence |
| inferred | Reasoned from other local evidence, with the chain stated |
| hypothesised | Proposed to make the play work; untested |
| unknown | No evidence either way |

A play can remain a viable hypothesis while several of its elements are unknown. What it cannot be is described as supported on the strength of analogy or a basin's reputation.

## 3. Gate before `subsurface_play`

**Full evaluation.** Run it when all of the following hold:
- the framework the play depends on exists: stratigraphy and structure, or their equivalents for the resource;
- reservoir and seal evidence has been reviewed;
- material contradictions are recorded in the ledger.

**Screening.** Earlier in the investigation, you may run it explicitly as a screening, to identify which elements lack evidence. Set `required_outputs` accordingly and label the result "screening".

**Brief contents.** Pass the following:
- the resource type and its element list;
- the per-element findings, as `upstream_findings` with global ids and classes;
- the competing interpretations;
- the contradictions from the ledger;
- `known_constraints` such as "Regional analogues are not local evidence" and "No numerical probabilities unless the project methodology below is supplied".

**Result review.** Check that:
- each element's label matches the evidence ids cited;
- timing has been evaluated as a relationship between two dated events, not assumed;
- the specialist has not generated prospects; that is not its job;
- the specialist has identified the evidence that would discriminate between competing plays.

## 4. Gate before `prospect_target`

All of the following must hold:
- the play concept has been evaluated, and its element labels are recorded;
- each element the target relies on has traceable support, even if that support is "hypothesised";
- the trap geometry comes from structural or seismic results, with the depth-conversion status stated;
- no element would have to be filled by an assumption that is absent from the integrated model.

If the gate fails, do not call the tool just because prospects were requested. Tell the user which elements are missing and what would supply them.

### Maturity

Use the project's terms if it defines them. Otherwise use this ladder, which follows common usage such as the play, lead and prospect classes of SPE-PRMS:

| Level | Minimum basis |
|---|---|
| Concept or play | A plausible idea consistent with the play; elements largely hypothesised or analogue-based |
| Lead | A specific feature identified in the data, but too poorly defined to assess as a drilling target |
| Prospect | A mapped trap with its elements evaluated, and sufficient data to assess risk and volumes |
| Drillable target | A prospect matured to location, depth and objectives, with data confidence adequate for the decision |

Never upgrade maturity because a feature has a name, a map or a volume.

## 5. Risk and uncertainty

**Brief contents.** Pass `risk_uncertainty` the following:
- the plays and targets, with their element labels;
- their dependencies;
- the contradictions and uncertainties from the ledger.

**Numerical chance of success.** Request a numerical chance of success only if the project supplies a risking methodology, such as company risk tables, element-probability rules or calibration. Pass it explicitly in `known_constraints` or `available_evidence`. Otherwise, ask for qualitative risk: critical elements, failure modes, and whether each is reducible.

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

Record each routed uncertainty in the ledger (`U#`) with its task.

## 6. Circularity checks

Before accepting play, prospect or risk conclusions, confirm that none of the following has happened:
- a prospect or play conclusion was used to justify the assumptions it depends on;
- an analogue-supported element was later treated as demonstrated;
- risk was lowered by dropping an alternative interpretation without evidence;
- several elements were counted as supported by what is really one line of evidence, such as a single seismic interpretation;
- a prospect was matured while its underlying hypothesis was unresolved or contradicted.
