---
name: play-core
description: The play specialist's working rules - the ordinal status scale, the scope rule, the weakest link, dependencies, wells as tests, the play tests PT1-PT7, the no-probability rule, element claims in the fixed form, products and discriminating evidence. Load at the start of every play task.
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
| contradicted | study-area evidence against | contradicted |

Ordinal: demonstrated > inferred > analogue-supported > hypothesised > unknown. No number is attached to any status; `play_status_matrix` refuses keys that read as a chance, score or percentage.

## 2. The rules the matrix applies

- **Scope.** A link whose scope is regional or analogue caps the element at analogue-supported. Claiming more is refused with the reason.
- **The elements come from the template.** `play_element_template` gives, per domain, the critical and supporting elements, their dependencies and what evidence lifts each; link evidence to those elements and no others.
- **Weakest link.** The play's status is the minimum over its critical elements; the capping element is named.
- **Dependencies.** maturation needs source; migration needs maturation; timing needs trap and maturation; preservation needs timing (hydrocarbon). injectivity and trapping need the storage unit; the pressure limit needs caprock and fault containment (CO2). permeability needs the structural setting; recharge needs permeability (geothermal). generation needs a source; migration needs generation (natural hydrogen); cycling integrity needs caprock and well containment (hydrogen storage). A demonstrated product proves its prerequisites: a measured hydrogen flux lifts source to inferred. An element above its prerequisite is flagged; a discovery proves the charge chain.
- **A claimed status may lower an element, never raise it.** Lower with the reason in the note (a product's ledger fails, a transfer is unknown, a caveat upstream).

## 3. Wells as tests (`play_classify_outcomes`)

Every well in the play area: success (what it demonstrates), failure on a named element (the operator's stated reason mapped to an element and kept beside it; a dry well outside the mapped closure fails on trap), or undetermined. Undetermined wells fail PT5 until you explain them with cited evidence (`explanations`).

## 4. The play tests

| Test | Pass when | Tool |
|---|---|---|
| PT1 | every critical element has a status (unknown allowed, named) | `play_tests` |
| PT2 | no element demonstrated on regional or analogue evidence | the matrix's refusals |
| PT3 | no dependency flag, or each explained | the matrix's flags |
| PT4 | trap and seal before charge; containment through time; heat and recharge sustained | `play_events_chart`, `play_timing_check` |
| PT5 | every well explained | `play_classify_outcomes` |
| PT6 | no post-charge uplift, breach or cool exposure (phase 1 tool; your reasoned judgement with cited evidence meanwhile) | `preservation` |
| PT7 | fairway coherent (phase 2) | not tested |

Status: supported with PT1-PT5 passed and the weakest critical element inferred or better; proposed when a critical element is unknown; contradicted on any failure or contradicted element; partially_supported otherwise; unresolved between competing concepts that pass the same tests.

## 5. Element claims

`<play> | <element> — <status>: <evidence in brief>` with `supporting_evidence_ids` the upstream ids and `depends_on_claim_ids` the upstream claims. The play claim: `<play> — overall: <status>, capped by <element> (<status>)`, depending on every element claim. The dependency ceiling applies: an element claim is never stronger than the weakest upstream claim it rests on.

## 6. Discriminating evidence

`play_discriminating_evidence` ranks, for every element below demonstrated, what would raise it and who supplies it. Its items go into `missing_data` in the shared format and into `recommended_followups`; Geo Oracle plans the next wave from them.

## 7. Products and requests

`element_status_matrix`, `play_concepts`, `petroleum_system_chart`, `well_outcome_analysis`, `containment_assessment` (CO2), `discriminating_evidence`. `missing_data` names: `regional_geology`, `seismic_interpretation`, `wells_petrophysics`, `structural_geology`, `stratigraphy`, `sedimentology`, `literature_review`.

## Methodological provenance

Every tool result carries `methods` — the ids of the published methods that computation
implements, such as `m:play/status-matrix` — and `citation_version`, the state of the bibliography
the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:play/status-matrix`), and name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user, and the method's name in words is what lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:play/status-matrix` says which
method; it never says who published it or when. An author surname may be part of a
method's name; a year never is. The reference lives in the registry, and
only `play_method_provenance` returns it.

When you are asked which equation was used, what publication supports a method, what the
method assumes, or how far it has been verified, call `play_method_provenance` with the
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

## 8. Self-check before emitting

- `source_reference` on every tool-derived item carries the `prov:` id **and** every `m:` id from the result's `methods`, copied exactly; a missing `m:` id is a missing citation.

1. Every status on a cited link; no element demonstrated on regional or analogue evidence; the weakest link named.
2. Every well classified or PT5 failing; every event dated by a cited claim.
3. No probability, percentage or score anywhere near an element or a play.
4. Every play with its PT1-PT7; the alternative concept assessed where named.
5. Products as `Product:` lines; figures as `Figure:` lines.
