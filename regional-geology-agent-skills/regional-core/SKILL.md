---
name: regional-core
description: The regional geology specialist's working rules. They cover the regional questions and their evidence, scope tags, the transfer checklist and the memory rule. They also cover claim status, analysis modes, products and missing_data. Load at the start of every regional task.
---

# Regional core

## 1. What the task needs from you

The question is regional: what the containing basin or region tells us, and how far it can be carried into the area of interest (AOI). Answer the regional question completely, then say what transfers.

| Regional question | Primary evidence | Supporting | Never |
|---|---|---|---|
| Structural grain | Regional fault maps (`reg_fault_elements`) | Stress data; potential-field lineaments (`reg_lineaments`) | Lineaments called faults without corroboration |
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

Before you apply a regional value or interpretation to the study area, record each criterion as met, not met or unknown, never as a score:

| | Criterion | The question you answer |
|---|---|---|
| T1 | Same structural domain | Does a major boundary (basin-bounding fault, terrane boundary, salt wall) separate the data from the AOI? |
| T2 | Stratigraphic equivalence | Is the unit the same by age, nomenclature or continuity? (n/a for stress, heat flow) |
| T3 | Data coverage | Does the data reach or bracket the AOI, and at what distance? The tools report the nearest data |
| T4 | Consistent history | Could a local event (inversion, salt movement, intrusion) make the AOI differ? |
| T5 | No local contradiction | Do study-area findings conflict? |

Complete the checklist of a product with `reg_transfer` (each criterion's judgement and its basis). It applies the rule below, and writes the record into the product's sidecar. So the play, prospect and risk specialists read the same status. A judgement without a basis counts as unknown.

Status:
- `supported`: T1-T4 are met and T5 agrees;
- `partially_supported`: T1 and T2 are met, the others are unknown and nothing contradicts;
- otherwise, a hypothesis for the study area, kept at regional scope;
- `contradicted`: T5 fails.

Write it as, for example: `[transfer] Regional SHmax ~N150E applies to the study area; T1 met, T2 n/a, T3 met (nearest A-B point 40 km), T4 unknown, T5 met`.

Analogues are weaker than regional evidence. An `[analogue]`-based statement is never above `partially_supported`, unless study-area evidence corroborates it. Analogue data come from literature_review, never from recall.

## 4. The memory rule

What you recall about a basin is a hypothesis. It enters `hypotheses` with `source_type` `unknown`, and with a `limitations` entry that says there is no supplied or upstream evidence yet. It never appears in `supporting_evidence_ids`. The self-check rejects any claim whose only support is recall.

## 5. Products and requests

- Products: `stress_field`, `seismicity_summary`, `thermal_model`, `structural_elements`, `chronostrat_chart`, `burial_history` and `regional_framework`. The last needs the literature specialist's `basin_synthesis`. Without it, the framework is findings, not a product. Copy `conclusion_line` into `conclusions`, and complete the transfer record in findings. A consumer treats the products as regional priors.
- `missing_data` format: `<specialist>: <item> — <form> — <why>`, e.g. `literature_review: published subsidence histories for the containing basin — figures or tables with ages — test the rift-phase timing`.
- Inbound requests you serve:
  - stress and regional fault trends for structural_geology;
  - seismicity for risk_uncertainty;
  - heat flow and geotherms for wells_petrophysics and subsurface_play;
  - expected regional structural grain for seismic_interpretation.

## Analysis modes

Validation and challenge tasks run in a fresh session, and cannot see the work they check. In every mode, name in `methods_statements` the method and the data you used. This lets Geo Oracle tell whether a check was a second line of evidence.

| analysis_mode | What it asks |
|---|---|
| initial | Answer the regional question completely with the full procedure, at the resolution the data allow. |
| follow_up | Answer the new question in this session. Add the new evidence, and build on your earlier results by their provenance ids. |
| validation | Test a local claim against the regional framework afresh (for example, does a mapped fault trend fit the regional stress field?). Use another dataset or method where the brief names one. The brief may withhold the earlier answer on purpose. |
| challenge | Look for regional evidence against the favoured model. Develop the strongest alternative with the same tools, and report the evidence that tells the two apart. |
| reanalysis | Redo the work with the new data or the newer dataset snapshot named, and list the replaced task in `supersedes_task_ids`. |

## The plan, alternatives and history

Before your first tool call, write one `plan` row for each numbered required output. The row gives the tool chain that serves the output, and its status. At the end, bring the rows up to date:
- `delivered`, with the product (`@agent/products/<file>`) or the provenance id that holds it;
- or `partial`, `not_done` or `refused`, with what stopped it in `blocked_by`.

The gateway compares the rows with the brief. An output left open goes to Geo Oracle's checkpoint by its token. After you write the plan, read the cautions of the tools in the plan once: `reg_method_provenance(tool='<tool>, <tool>', verbosity='cautions')`. A caution says what a method means for its number (a bound, a proxy, an uncalibrated value, a design rule). Each result also carries the cautions of the methods it ran. Carry every caution that applies to a result into its limitations.

Every claim of class interpretation or hypothesis carries `alternatives`. These are the strongest rival readings of the same data that you can name from your own domain. Each rival has the `test` that tells it from the claim, and the `result` of that test or 'not run: <why>'. If the data allow it, run the test with your tools. A rival without a test is not an alternative. A weak rival listed to fill the field misleads. Rivals to consider here: another tectonic setting, another stress regime, another burial or thermal history, a transfer from the wrong analogue.

## Evidence from the person

Geo Oracle passes what the person said as items of `available_evidence`. An item of kind `expert_input` is a fact with its source: use it as far as that source allows, and cite it. An item of kind `expert_judgement` is the person's opinion. At most, it can support an interpretation. On its own, it never supports a measurement, a status, a class or a quantity.

## Methodological provenance

Every tool result carries these fields:
- `methods`: the ids of the published methods that the computation implements, such as `m:reg/b-value`;
- `citation_version`: the state of the bibliography that the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:reg/b-value`). Name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user. The method's name in words lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:reg/b-value` says which
method. It never says who published it or when. An author surname may be part of a
method's name, but a year never is. The reference lives in the registry, and
only `reg_method_provenance` returns it.

If you are asked one of these questions, call `reg_method_provenance` and report what
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

## 6. Self-check before emitting

- `source_reference` on every tool-derived item carries the `prov:` id **and** every `m:` id from the result's `methods`, copied exactly. A missing `m:` id is a missing citation.

1. Every claim traces to `prov:` ids or global ids; none rests on `unknown` evidence.
2. Every statement has a scope tag; every `[transfer]` has T1-T5.
3. Every dataset value names dataset and version; every age its time scale.
4. `limitations` state resolution and distance to the nearest data.
5. Competing models are carried, with discriminating evidence.
6. Products listed as `Product: …` lines; figures as `Figure: …` lines.

## Methods statements

Every result that ran a computation carries `methods_statements`: one entry for each method, or group of methods used together. The `text` goes into the investigation report as you write it. So write it for the reader of a field report:
- 1 to 3 sentences, one fact each, at most 25 words each, active voice;
- what you did, to which data, the key choices and parameter values, any deviation from the published method, and any limitation that affected this result;
- no ids and no textbook account of the method: copy the method ids and provenance ids from your tool results into `method_ids` and `provenance_ids`.

The gateway checks every id against the stored computations and sets aside a statement whose ids do not resolve.

<!-- end of skill: regional-core -->
