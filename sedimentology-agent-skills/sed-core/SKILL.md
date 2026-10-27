---
name: sed-core
description: The sedimentology specialist's working rules. They cover the interpretation ladder, diagnostic before suggestive, the test ledger S1-S7, competing models and the analogue cap. They also cover image readings as flagged observations, claim status, products and missing_data. Load at the start of every sedimentological task.
---

# Sedimentology core

## 1. The ladder

| Rung | What it is | What gets you there |
|---|---|---|
| observation | A described or measured feature at a depth | A description table, an analysis, a flagged image reading |
| lithofacies | A coded rock type with its features | `sed_code_lithofacies` by the stated rules |
| association | Facies that occur together in a non-random succession | `sed_transitions_all` (or `sed_transitions` and `sed_associations`): above-random links |
| environment | The setting that made the association | A diagnostic feature (`sed_diagnostic_check`) plus the tests |
| system | Linked environments (a delta, a tidal estuary, a fan) | Several environments on an admitted framework, with geometry |

A claim states its rung. The characteristic error of this discipline is to skip a rung. A fining-upward unit is not a river, a graded bed is not a turbidite, and heterolithic bedding is not a tidal flat.

## 2. Diagnostic before suggestive

For each environment, the matrix in `sed-depositional-systems` lists:
- the diagnostic features (rare in competing environments);
- the suggestive features (compatible with several);
- the features that count against it.

`sed_diagnostic_check` returns S2:
- `pass` with a diagnostic feature;
- `suggestive_only` without one;
- `fail` with features against it and none diagnostic.

A suggestive-only result caps the model at `partially_supported` at the association rung.

## 3. The ledger

| Test | Question | Tool or input |
|---|---|---|
| S1 | Every facies rests on described features or calibrated electrofacies (image readings alone fail) | `observational_basis` |
| S2 | At least one diagnostic feature | `sed_diagnostic_check` |
| S3 | Transitions non-random with above-random links (pooled; fewer than 10 transitions: not tested) | `pooled_transitions` file from `sed_transitions_all` (or the pooled file from `sed_associations`) |
| S4 | Fits the key surfaces and the regional setting supplied | `stratigraphic_fit` |
| S5 | Geobody or analogue geometry fits | no tool yet: pass `geometry` with its evidence, or leave it not tested |
| S6 | Palaeocurrent pattern fits | no tool yet: pass `palaeocurrents` with its evidence, or leave it not tested |
| S7 | Trace fossils and fauna fit the salinity and energy implied | `biota` |

Status:
- `supported`: S1 and S2 passed, two of S3-S7 passed, none failed;
- `partially_supported`: S1 passed, and S2 suggestive-only or few tests run;
- `contradicted`: an unexplained failure;
- `unresolved`: two models pass the same tests.

The ledger goes into the depositional_model sidecar and into the statement of the interpretation.

## 4. Competing models

Tidal against fluvial channels; shoreface against delta front; turbidites against storm beds; lake against lagoon. If the association's features allow both models, build both and test both. Then report the loser as a hypothesis with its failing test. Or report both as `unresolved`, with the discriminator in `missing_data` (`wells_petrophysics: cross-bed dips from the image log — azimuth per pick — bimodal directions would favour the tidal model`).

## 5. Analogues and images

- Dimensions, widths and net-to-gross from analogues carry `[analogue]`, the dataset and the sample size. Without local corroboration, they are at most `partially_supported`.
- Your reading of a photograph is an observation, flagged image-derived. Cross-check it against the description. Put a disagreement in `contradictions`, never a silent override.

## 6. Caveats are ceilings

Caveats include a provisional framework, an uncalibrated Vsh and a regional setting with unknown transfer. Each caveat lowers the status of what rests on it, and goes into `assumptions` and `limitations`. No caveat stops a computation. Run the tools, then cap the claim. Answer `insufficient_data` only after the tools have run or refused. Never restate an upstream number as your own `derived` measurement.

## 7. Products and requests

- `facies_scheme`, `facies_log`, `depositional_model` now; `palaeocurrents`, `reservoir_architecture`, `gde_maps`, `reservoir_quality_controls`, `provenance_summary` later.
- `missing_data` names: `wells_petrophysics` (logs, core depth shift, dip picks, core analyses), `stratigraphy` (framework, surfaces), `seismic_interpretation` (stratal slices, geobodies), `regional_geology` (setting, palaeogeography), `literature_review` (facies models, analogue dimensions).

## Analysis modes

Validation and challenge tasks run in a fresh session, and cannot see the work they check. In every mode, name in `methods_statements` the method and the data you used. This lets Geo Oracle tell whether a check was a second line of evidence.

| analysis_mode | What it asks |
|---|---|
| initial | Answer the question at the resolution the data allow. |
| follow_up | Answer the narrower question in this session; build on your earlier results by their provenance ids. |
| validation | Interpret afresh from the rock data given, and run the interpretation tests on your model. The brief may withhold the earlier model on purpose. |
| challenge | Run the strongest rival depositional model through the interpretation tests beside the named one, and report the tests that separate them. |
| reanalysis | Redo the work with the changed inputs named, and list the replaced task in `supersedes_task_ids`. |

`check_outcomes`: when the brief lists 'Claims to test', give one entry per claim: `target` (the claim id), `verdict` (`supported`, `contradicted`, `unresolved` or `not_tested`), the `evidence_ids` of your own evidence that decide it, and a one-sentence `basis`. A verdict says what your evidence shows, not whether the claim is true. A blind validation has no such list: leave it empty.

## The plan, alternatives and history

Before your first tool call, write one `plan` row for each numbered required output. The row gives the tool chain that serves the output, and its status. At the end, bring the rows up to date:
- `delivered`, with the product (`@agent/products/<file>`) or the provenance id that holds it;
- or `partial`, `not_done` or `refused`, with what stopped it in `blocked_by`.

The gateway compares the rows with the brief. An output left open goes to Geo Oracle's checkpoint by its token. After you write the plan, read the cautions of the tools in the plan once: `sed_method_provenance(tool='<tool>, <tool>', verbosity='cautions')`. A caution says what a method means for its number (a bound, a proxy, an uncalibrated value, a design rule). Each result also carries the cautions of the methods it ran. Carry every caution that applies to a result into its limitations.

Every claim of class interpretation or hypothesis carries `alternatives`. These are the strongest rival readings of the same data that you can name from your own domain. Each rival has the `test` that tells it from the claim, and the `result` of that test or 'not run: <why>'. If the data allow it, run the test with your tools. A rival without a test is not an alternative. A weak rival listed to fill the field misleads. Rivals to consider here: another depositional environment that gives the same facies, reworking, a diagenetic overprint read as depositional.

## Evidence from the person

Geo Oracle passes what the person said as items of `available_evidence`. An item of kind `expert_input` is a fact with its source: use it as far as that source allows, and cite it. An item of kind `expert_judgement` is the person's opinion. At most, it can support an interpretation. On its own, it never supports a measurement, a status, a class or a quantity.

## Methodological provenance

Every tool result carries these fields:
- `methods`: the ids of the published methods that the computation implements, such as `m:sed/transitions`;
- `citation_version`: the state of the bibliography that the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:sed/transitions`). Name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user. The method's name in words lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:sed/transitions` says which
method. It never says who published it or when. An author surname may be part of a
method's name, but a year never is. The reference lives in the registry, and
only `sed_method_provenance` returns it.

If you are asked one of these questions, call `sed_method_provenance` and report what
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

1. Every facies on described features; the rung stated on every claim.
2. Every environment with a diagnostic feature or stopped at the association rung.
3. Every model with its ledger; the alternative built and tested where possible.
4. Core depths shifted to log depth before any comparison; the depth reference stated.
5. Image readings flagged; analogues tagged; products as `Product:` lines; figures as `Figure:` lines.

## Methods statements

Every result that ran a computation carries `methods_statements`: one entry for each method, or group of methods used together. The `text` goes into the investigation report as you write it. So write it for the reader of a field report:
- 1 to 3 sentences, one fact each, at most 25 words each, active voice;
- what you did, to which data, the key choices and parameter values, any deviation from the published method, and any limitation that affected this result;
- no ids and no textbook account of the method: copy the method ids and provenance ids from your tool results into `method_ids` and `provenance_ids`.

The gateway checks every id against the stored computations and sets aside a statement whose ids do not resolve.

<!-- end of skill: sed-core -->
