---
name: sed-core
description: The sedimentology specialist's working rules - the interpretation ladder, diagnostic before suggestive, the test ledger S1-S7, competing models, the analogue cap, image readings as flagged observations, claim status, products and missing_data. Load at the start of every sedimentological task.
---

# Sedimentology core

## 1. The ladder

| Rung | What it is | What gets you there |
|---|---|---|
| observation | A described or measured feature at a depth | A description table, an analysis, a flagged image reading |
| lithofacies | A coded rock type with its features | `sed_code_lithofacies` by the stated rules |
| association | Facies that occur together in a non-random succession | `sed_transitions` and `sed_associations` (above-random links) |
| environment | The setting that made the association | A diagnostic feature (`sed_diagnostic_check`) plus the tests |
| system | Linked environments (a delta, a tidal estuary, a fan) | Several environments on an admitted framework, with geometry |

A claim states its rung. Skipping a rung is the characteristic error of this discipline: a fining-upward unit is not a river, a graded bed is not a turbidite, heterolithic bedding is not a tidal flat.

## 2. Diagnostic before suggestive

The matrix in `sed-depositional-systems` lists, per environment, the diagnostic features (rare in competing environments), the suggestive ones (compatible with several) and those that count against it. `sed_diagnostic_check` returns S2: `pass` with a diagnostic feature, `suggestive_only` without one, `fail` with features against it and none diagnostic. Suggestive-only caps the model at `partially_supported` at the association rung.

## 3. The ledger

| Test | Question | Tool or input |
|---|---|---|
| S1 | Every facies rests on described features or calibrated electrofacies (image readings alone fail) | `observational_basis` |
| S2 | At least one diagnostic feature | `sed_diagnostic_check` |
| S3 | Transitions non-random with above-random links (pooled; fewer than 10 transitions: not tested) | pooled file from `sed_associations` |
| S4 | Fits the key surfaces and the regional setting supplied | `stratigraphic_fit` |
| S5 | Geobody or analogue geometry fits | phase 2 |
| S6 | Palaeocurrent pattern fits | phase 1 |
| S7 | Trace fossils and fauna fit the salinity and energy implied | `biota` |

Status: `supported` with S1 and S2 passed, two of S3-S7 passed, none failed; `partially_supported` with S1 passed and S2 suggestive-only or few tests run; `contradicted` on an unexplained failure; `unresolved` when two models pass the same tests. The ledger goes into the depositional_model sidecar and into the statement of the interpretation.

## 4. Competing models

Tidal against fluvial channels; shoreface against delta front; turbidites against storm beds; lake against lagoon. Build both whenever the association's features allow both; test both; report the loser as a hypothesis with its failing test, or both as `unresolved` with the discriminator in `missing_data` (`wells_petrophysics: cross-bed dips from the image log — azimuth per pick — bimodal directions would favour the tidal model`).

## 5. Analogues and images

- Dimensions, widths and net-to-gross from analogues carry `[analogue]`, the dataset and sample size, and are at most `partially_supported` without local corroboration.
- Your reading of a photograph is an observation flagged image-derived; it is cross-checked against the description, and a disagreement goes to `contradictions`, never a silent override.

## 6. Caveats are ceilings

A provisional framework, an uncalibrated Vsh, a regional setting with unknown transfer: each lowers the status of what rests on it and goes into `assumptions` and `limitations`; none stops a computation. Run the tools, then cap the claim. `insufficient_data` only after the tools have run or refused. Never restate an upstream number as your own `derived` measurement.

## 7. Products and requests

- `facies_scheme`, `facies_log`, `depositional_model` now; `palaeocurrents`, `reservoir_architecture`, `gde_maps`, `reservoir_quality_controls`, `provenance_summary` later.
- `missing_data` names: `wells_petrophysics` (logs, core depth shift, dip picks, core analyses), `stratigraphy` (framework, surfaces), `seismic_interpretation` (stratal slices, geobodies), `regional_geology` (setting, palaeogeography), `literature_review` (facies models, analogue dimensions).

## Methodological provenance

Every tool result carries `methods` — the ids of the published methods that computation
implements, such as `m:sed/transitions` — and `citation_version`, the state of the bibliography
the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:sed/transitions`), and name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user, and the method's name in words is what lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:sed/transitions` says which
method; it never says who published it or when. An author surname may be part of a
method's name; a year never is. The reference lives in the registry, and
only `sed_method_provenance` returns it.

When you are asked which equation was used, what publication supports a method, what the
method assumes, or how far it has been verified, call `sed_method_provenance` with the
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

1. Every facies on described features; the rung stated on every claim.
2. Every environment with a diagnostic feature or stopped at the association rung.
3. Every model with its ledger; the alternative built and tested where possible.
4. Core depths shifted to log depth before any comparison; the depth reference stated.
5. Image readings flagged; analogues tagged; products as `Product:` lines; figures as `Figure:` lines.
