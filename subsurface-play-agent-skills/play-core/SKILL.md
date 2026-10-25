---
name: play-core
description: The play specialist's working rules. They cover the ordinal status scale, the scope rule, the weakest link, dependencies, wells as tests and the play tests PT1-PT7. They also cover the no-probability rule, element claims in the fixed form, products and discriminating evidence. Load at the start of every play task.
---

# Play core

## 1. The status scale

| Status | Needs | Claim status |
|---|---|---|
| demonstrated | study-area observations or measurements (direct) | supported |
| inferred | study-area evidence through a stated reasoning step; or a direct line of partial status | partially_supported |
| analogue-supported | regional or analogue evidence only, with its transfer status | partially_supported, `[analogue]` or `[regional]` |
| hypothesised | proposed, not yet supported | proposed |
| unknown | no evidence either way | unresolved |
| disfavoured | evidence against that does not establish failure: a study-area interpretation or inference against, or regional or analogue evidence against, with no better evidence for the element | unresolved, naming the evidence against |
| contradicted | study-area observations or measurements against | contradicted |

The scale is ordinal: demonstrated > inferred > analogue-supported > hypothesised > unknown > disfavoured. Contradicted fails the element. Never attach a number to any status. `play_status_matrix` refuses keys that read as a chance, score or percentage.

## 2. The rules the matrix applies

- **Scope.** A link whose scope is regional or analogue caps the element at analogue-supported. The matrix refuses a claim of more, with the reason. A link without a scope counts as regional, so give every link its scope.
- **Evidence against counts by its grade** (Rose 2001, Fig. 19: unfavourable data lower confidence however sparse they are; only good data justify a strong negative).
  - Study-area observations against contradict.
  - An interpretation or inference against, or regional or analogue evidence against, makes the element disfavoured, unless evidence for it of a higher grade outranks it. Study-area observations for it outrank both. A study-area interpretation for it outranks regional or analogue evidence against.
  - The matrix lists outranked evidence against as `conflicting_evidence`. It also lists there the evidence for a disfavoured element, and flags `tie` when that evidence is of the same grade as the evidence against. A tie stays disfavoured. An element that a discovery lifts keeps its evidence against there.
  - Report `conflicting_evidence` as a contradiction in the findings.
  - A hypothesis is not evidence against.
  - Link evidence against as `against: true`. Never leave it out because it is regional.
- **The elements come from the template.** For each domain, `play_element_template` gives the critical and supporting elements, their dependencies and what evidence lifts each. Link evidence to those elements and no others.
- **Weakest link.** The play's status is the minimum over its critical elements, and the capping element is named.
- **Dependencies** (from `play_element_template`):
  - Hydrocarbon: maturation needs source; migration needs maturation; timing needs trap and maturation; preservation needs timing.
  - CO2: injectivity and trapping need the storage unit; the pressure limit needs caprock and fault containment.
  - Geothermal: permeability needs the structural setting; recharge needs permeability.
  - Natural hydrogen: generation needs a source; migration needs generation; preservation needs the seal.
  - Hydrogen storage: injectivity and deliverability need the storage unit; geochemical and microbial stability needs the storage unit; cycling integrity needs caprock and well containment.

  A demonstrated product proves its prerequisites: a measured hydrogen flux lifts source to inferred. The matrix flags an element above its prerequisite. A discovery proves the charge chain.
- **A claimed status may lower an element, never raise it.** Lower it with the reason in the note (a product's ledger fails, a transfer is unknown, a caveat upstream).
- **Figures.** `play_status_matrix` draws the status matrix and each play's dependency graph in `result.figures`. Publish them.

## 3. Wells as tests (`play_classify_outcomes`)

Every well in the play area is one of these:
- a success (what it demonstrates);
- a failure on a named element. The tool maps the operator's stated reason to an element, and keeps the reason beside it. A dry well outside the mapped closure fails on trap;
- undetermined.

Undetermined wells fail PT5 until you explain them with cited evidence (`explanations`).

## 4. The play tests

| Test | Pass when | Tool |
|---|---|---|
| PT1 | every critical element has a status (unknown allowed, named) | `play_tests` |
| PT2 | no element demonstrated on regional or analogue evidence | the matrix's refusals |
| PT3 | no dependency flag: clear a flag by revising the matrix (lower the element or support its prerequisite); an explained flag keeps the play below supported | the matrix's flags |
| PT4 | trap and seal before charge; containment through time; heat and recharge sustained | `play_events_chart`, `play_timing_check` |
| PT5 | every well explained | `play_classify_outcomes` |
| PT6 | no post-charge uplift, breach or cool exposure (no tool computes it: give your reasoned judgement with cited evidence) | `play_tests` argument `preservation` |
| PT7 | fairway coherent (no tool tests it yet) | always not tested |

Status:
- supported: PT1-PT5 passed, and the weakest critical element inferred or better;
- proposed: a critical element unknown or disfavoured (the tests list disfavoured elements);
- contradicted: any failure or a contradicted element;
- partially_supported: otherwise;
- unresolved: competing concepts that pass the same tests.

## 5. Element claims

`<play> | <element> — <status>: <evidence in brief>`, with `supporting_evidence_ids` the upstream ids and `depends_on_claim_ids` the upstream claims. The play claim is `<play> — overall: <status>, capped by <element> (<status>)`, and it depends on every element claim. The dependency ceiling applies: an element claim is never stronger than the weakest upstream claim it rests on.

## 6. Discriminating evidence

For every element below demonstrated, `play_discriminating_evidence` ranks what would raise it and who supplies it. Its items go into `missing_data` in the shared format and into `recommended_followups`. Geo Oracle plans the next wave from them.

## 7. Products and requests

`element_status_matrix`, `play_concepts`, `petroleum_system_chart`, `well_outcome_analysis`, `containment_assessment` (CO2), `discriminating_evidence`. `missing_data` names: `regional_geology`, `seismic_interpretation`, `wells_petrophysics`, `structural_geology`, `stratigraphy`, `sedimentology`, `literature_review`.

## Analysis modes

Validation and challenge tasks run in a fresh session, and cannot see the work they check. In every mode, name in `methods_statements` the method and the data you used. This lets Geo Oracle tell whether a check was a second line of evidence.

| analysis_mode | What it asks |
|---|---|
| initial | Answer the question at the resolution the data allow. |
| follow_up | Answer the narrower question in this session; build on your earlier results by their provenance ids. |
| validation | Assess the play afresh from the evidence given and build the status matrix again. The brief may withhold the earlier matrix on purpose. |
| challenge | State the strongest rival play concept (another reservoir, seal or charge story), assess it with the status matrix and the play tests, and rank the discriminating evidence for both. |
| reanalysis | Redo the work with the changed inputs named, and list the replaced task in `supersedes_task_ids`. |

## The plan, alternatives and history

Before your first tool call, write one `plan` row for each numbered required output. The row gives the tool chain that serves the output, and its status. At the end, bring the rows up to date:
- `delivered`, with the product (`@agent/products/<file>`) or the provenance id that holds it;
- or `partial`, `not_done` or `refused`, with what stopped it in `blocked_by`.

The gateway compares the rows with the brief. An output left open goes to Geo Oracle's checkpoint by its token. After you write the plan, read the cautions of the tools in the plan once: `play_method_provenance(tool='<tool>, <tool>', verbosity='cautions')`. A caution says what a method means for its number (a bound, a proxy, an uncalibrated value, a design rule). Each result also carries the cautions of the methods it ran. Carry every caution that applies to a result into its limitations.

Every claim of class interpretation or hypothesis carries `alternatives`. These are the strongest rival readings of the same data that you can name from your own domain. Each rival has the `test` that tells it from the claim, and the `result` of that test or 'not run: <why>'. If the data allow it, run the test with your tools. A rival without a test is not an alternative. A weak rival listed to fill the field misleads. Rivals to consider here: another reservoir, seal or charge story for the same wells, a failed well read as a dry play.

## Evidence from the person

Geo Oracle passes what the person said as items of `available_evidence`. An item of kind `expert_input` is a fact with its source: use it as far as that source allows, and cite it. An item of kind `expert_judgement` is the person's opinion. At most, it can support an interpretation. On its own, it never supports a measurement, a status, a class or a quantity.

## Methodological provenance

Every tool result carries these fields:
- `methods`: the ids of the published methods that the computation implements, such as `m:play/status-matrix`;
- `citation_version`: the state of the bibliography that the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:play/status-matrix`). Name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user. The method's name in words lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:play/status-matrix` says which
method. It never says who published it or when. An author surname may be part of a
method's name, but a year never is. The reference lives in the registry, and
only `play_method_provenance` returns it.

If you are asked one of these questions, call `play_method_provenance` and report what
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

## 8. Self-check before emitting

- `source_reference` on every tool-derived item carries the `prov:` id **and** every `m:` id from the result's `methods`, copied exactly. A missing `m:` id is a missing citation.

1. Every status on a cited link; no element demonstrated on regional or analogue evidence; the weakest link named.
2. Every well classified or PT5 failing; every event dated by a cited claim.
3. No probability, percentage or score anywhere near an element or a play.
4. Every play with its PT1-PT7; the alternative concept assessed where named.
5. Products as `Product:` lines; figures as `Figure:` lines.

## Methods statements

Every result that ran a computation carries `methods_statements`: one entry for each method, or group of methods used together. The `text` goes into the investigation report as you write it. So write it for the reader of a field report:
- 1 to 3 sentences, one fact each, at most 25 words each, active voice;
- what you did, to which data, the key choices and parameter values, any deviation from the published method, and any limitation that affected this result;
- no ids and no textbook account of the method: copy the method ids and provenance ids from your tool results into `method_ids` and `provenance_ids`.

The gateway checks every id against the stored computations and sets aside a statement whose ids do not resolve.

<!-- end of skill: play-core -->
