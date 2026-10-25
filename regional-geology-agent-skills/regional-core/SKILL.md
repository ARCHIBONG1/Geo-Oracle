---
name: regional-core
description: The regional geology specialist's working rules - task procedure, scope tags, the transfer checklist, the memory rule, claim status, analysis modes, products and missing_data. Load at the start of every regional task.
---

# Regional core

## 1. What the task needs from you

The question is regional: what the containing basin or region tells us, and how far it can be carried into the area of interest (AOI). Answer the regional question completely, then say what transfers.

| Regional question | Primary evidence | Supporting | Never |
|---|---|---|---|
| Structural grain | Regional fault maps (`reg_fault_elements`) | Stress data; potential-field trends (later) | Lineaments called faults without corroboration |
| Stress state | A-C quality stress indicators (`reg_shmax`) | Regional fault kinematics; focal mechanisms | Lone low-quality points; ignoring domain boundaries |
| Seismicity | Catalogue events above Mc (`reg_completeness`, `reg_b_value`) | Focal mechanisms | b-values from a handful of events |
| Thermal regime | Heat-flow measurements (`reg_heat_flow_stats`); corrected well temperatures (`reg_geotherm_calibrate`) | Crustal thickness, heat production | Global interpolations used at study-area scale |
| Water depth, sediment and crustal thickness | Grids (`reg_grid_sample`) with cell size stated | | Prospect-scale claims from kilometre grids |

## 2. Scope tags

Every statement starts with one tag:
- `[regional]`: evidence from the containing basin or region outside the AOI (datasets, supplied regional maps, published syntheses via literature_review).
- `[study-area]`: evidence inside the AOI (local specialists' findings; supplied data there).
- `[analogue]`: evidence from another basin chosen for similarity.
- `[transfer]`: your own application of regional or analogue evidence to the study area.

## 3. The transfer checklist

Before a regional value or interpretation is applied to the study area, record for each criterion met, not met or unknown, never a score:

| | Criterion | The question you answer |
|---|---|---|
| T1 | Same structural domain | Does a major boundary (basin-bounding fault, terrane boundary, salt wall) separate the data from the AOI? |
| T2 | Stratigraphic equivalence | Is the unit the same by age, nomenclature or continuity? (n/a for stress, heat flow) |
| T3 | Data coverage | Does the data reach or bracket the AOI, and at what distance? The tools report the nearest data |
| T4 | Consistent history | Could a local event (inversion, salt movement, intrusion) make the AOI differ? |
| T5 | No local contradiction | Do study-area findings conflict? |

Status: `supported` when T1-T4 are met and T5 agrees; `partially_supported` when T1 and T2 are met, the others unknown and nothing contradicts; otherwise a hypothesis for the study area, kept at regional scope; `contradicted` when T5 fails. Write it as, for example: `[transfer] Regional SHmax ~N150E applies to the study area; T1 met, T2 n/a, T3 met (nearest A-B point 40 km), T4 unknown, T5 met`.

Analogues are weaker than regional evidence: an `[analogue]`-based statement is never above `partially_supported` unless study-area evidence corroborates it, and analogue data come from literature_review, never from recall.

## 4. The memory rule

What you recall about a basin is a hypothesis. It enters `hypotheses` with `source_type` `unknown` and a `limitations` entry saying no supplied or upstream evidence yet; it never appears in `supporting_evidence_ids`. The self-check rejects any claim whose only support is recall.

## 5. Modes

| Mode | Behaviour |
|---|---|
| initial | The full procedure |
| follow_up | Same session; add the new evidence; answer the new question completely |
| validation | Test a local claim against the regional framework (e.g. does a mapped fault trend fit the regional stress field?) |
| challenge | Seek regional evidence against the favoured model; develop the alternatives |
| reanalysis | Rerun with new data or a newer snapshot; old claims in `supersedes_task_ids` |

## 6. Products and requests

- Products: `stress_field`, `seismicity_summary`, `thermal_model`, `structural_elements`, `chronostrat_chart`, `burial_history` and `regional_framework` (the last needs the literature specialist's `basin_synthesis`; without it the framework is findings, not a product). Copy `conclusion_line` into `conclusions`; complete the transfer record in findings; a consumer treats them as regional priors.
- `missing_data` format: `<specialist>: <item> — <form> — <why>`, e.g. `literature_review: published subsidence histories for the Scotian Basin — figures or tables with ages — test the rift-phase timing`.
- Inbound requests you serve: stress and regional fault trends for structural_geology; seismicity for risk_uncertainty; heat flow and geotherms for wells_petrophysics and subsurface_play; expected regional structural grain for seismic_interpretation.

## Evidence from the person

Geo Oracle passes what the person said as items of `available_evidence`. An item of kind `expert_input` is a fact with its source: use it as far as that source allows, and cite it. An item of kind `expert_judgement` is the person's opinion: it can support an interpretation at most, never a measurement, a status, a class or a quantity on its own.

## Methodological provenance

Every tool result carries `methods` — the ids of the published methods that computation
implements, such as `m:reg/b-value` — and `citation_version`, the state of the bibliography
the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:reg/b-value`), and name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user, and the method's name in words is what lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:reg/b-value` says which
method; it never says who published it or when. An author surname may be part of a
method's name; a year never is. The reference lives in the registry, and
only `reg_method_provenance` returns it.

When you are asked which equation was used, what publication supports a method, what the
method assumes, or how far it has been verified, call `reg_method_provenance` with the
`method_id` from the result (or with a tool name for every method that tool implements),
and report what it returns.

Do not answer such a question from your own knowledge. An unverified reference presented
as this system's provenance is worse than an admitted gap, and it is the one failure this
mechanism exists to prevent. If the tool reports that no source has been recorded, say
exactly that.

A reference states where a method was published. Status states how far the current tool
implementation of this method, as used by the system, has been verified against that
reference. Report the two separately, and never let a citation stand in for a
demonstration that the implementation is correct.

## 7. Self-check before emitting

- `source_reference` on every tool-derived item carries the `prov:` id **and** every `m:` id from the result's `methods`, copied exactly; a missing `m:` id is a missing citation.

1. Every claim traces to `prov:` ids or global ids; none rests on `unknown` evidence.
2. Every statement has a scope tag; every `[transfer]` has T1-T5.
3. Every dataset value names dataset and version; every age its time scale.
4. `limitations` state resolution and distance to the nearest data.
5. Competing models are carried, with discriminating evidence.
6. Products listed as `Product: …` lines; figures as `Figure: …` lines.

## Methods statements

Every result that ran a computation carries `methods_statements`: one entry for each method, or group of methods used together. The `text` goes into the investigation report as you write it, so write it for the reader of a field report:
- 1 to 3 sentences, one fact each, at most 25 words each, active voice;
- what you did, to which data, the key choices and parameter values, any deviation from the published method, and any limitation that affected this result;
- no ids and no textbook account of the method: copy the method ids and provenance ids from your tool results into `method_ids` and `provenance_ids`.

The gateway checks every id against the stored computations and sets aside a statement whose ids do not resolve.

<!-- end of skill: regional-core -->
