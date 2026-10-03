---
name: strat-core
description: The stratigraphy specialist's working rules - the two ledgers, the tie-point hierarchy, the consistency ledger C1-C7, competing panels, claim status, the dependency ceiling, products and missing_data. Load at the start of every stratigraphic task.
---

# Stratigraphy core

## 1. What the task needs from you

| Stratigraphic question | Primary methods | Supporting | Never |
|---|---|---|---|
| Which surfaces correlate between wells? | Rank 1-3 tie points, then `stg_dtw_pair` constrained between them | Markers, stacking patterns | Lithology alone; unconstrained pattern matching |
| Is the reported top right? | C1: the reported top beside the correlated surface, difference listed | Core, sidewall samples | Silent replacement of the reported pick |
| How certain is the correlation? | The C1-C7 ledger; the alternative panel; `stg_compare_panels` | DTW support against rigid shifts (C6) | One panel presented as the only one |
| How old is this surface? (phase 1) | Age model from dated samples and datums | Regional chart | Ages from recalled zonations |
| Is there a gap here? (phase 1) | Age jump across the surface; truncation on tied seismic | Thickness trends; structural fault cut-outs | Calling a fault cut-out an unconformity |

## 2. The two ledgers

Reported tops arrive in `well_tops` products or supplied tables. They are `data`, cited by their product reference, and never moved. Your picks are `interpretation`, with a tie rank and a source. The panel holds both, tagged; `stg_consistency` lists every disagreement beyond the tolerance (C1). A disagreement is reported in `contradictions` with the likeliest reason (a lithostratigraphic pick, a different datum, a depth-reference error) and the discriminating data. C1 fails only when a pick is untagged or a reported top was replaced.

## 3. The tie-point hierarchy

| Rank | Tie point | Gives | Caveat |
|---|---|---|---|
| 1 | Dated horizons (ash beds, reversals) | Time lines with error | Error bars may exceed the interval |
| 2 | Biostratigraphic datums | Time lines within known diachroneity | Last occurrences only from cuttings; reworking |
| 3 | Seismic horizons tied at wells | Near-time lines | Seismic resolution and tie quality |
| 4 | Markers, flooding shales, coals, condensed sections | Likely time lines | Identity must be shown |
| 5 | Log-pattern match (DTW) | Shape match | Patterns repeat at different ages |
| 6 | Lithological similarity | Rock units | Often diachronous |

Anchors are ranks 1-3 (`anchors` of `stg_dtw_pair`); ranks 4-6 support. A surface correlated on rank 4-6 evidence alone caps its panel at `partially_supported`. A lithostratigraphic unit may cross time lines: when a rock-line correlation and a time-line correlation disagree, build both panels; the one that crosses time lines fails C2 and is reported as the contradicted alternative.

## 4. The consistency ledger

| Test | Question | Tool |
|---|---|---|
| C1 | Every pick tagged; disagreements listed | `stg_consistency` |
| C2 | Surfaces keep their order in every well; no crossing | `stg_consistency` |
| C3 | Age-depth monotonic per well; the surfaces' ages do not reverse | `stg_consistency` with `age_models` |
| C4 | Thickness changes smooth or explained (fault, erosion, growth) | `stg_consistency` with `thickness_explanations` |
| C5 | Correlated surfaces within tolerance of tied seismic horizons | `stg_consistency` with `seismic_ties` |
| C6 | DTW cost in the best quarter of rigid shifts | `stg_dtw_pair` results passed to `stg_consistency` |
| C7 | Surface ages in the chart's span and order, given its transfer status | `stg_consistency` with `chart` |

Status: `supported` when C1-C3 pass and the surfaces rest on a rank 1-3 tie; `partially_supported` when C1 and C2 pass and the rest are not tested or the ties are rank 4-6; `contradicted` on any unexplained failure; `unresolved` when two panels pass the same tests. Record the ledger in every panel interpretation and in the product sidecar.

## 5. Competing panels

Build the alternative whenever the data permit one, with the same tie points. `stg_compare_panels` gives the per-surface differences and the verdict. Ties are `unresolved` hypotheses with `missing_data` naming the discriminator: `literature_review: calibrated age of the Ammonite X datum — Ma with error — separate the shingled from the layer-cake panel`.

## 6. Caveats are ceilings

An uncalibrated Vsh, a provisional operator top, a regional chart with unknown transfer: each lowers the status of what rests on it and goes into `assumptions` and `limitations`; none stops a computation. Run the tools, then cap the claim. `insufficient_data` only after the tools have run or refused. Never restate an upstream number as your own `derived` measurement.

## 7. Products and requests

- `correlated_tops` (every pick with tag, rank, basis, source, and the ledger in the sidecar), `correlation_alternatives`, `age_model`, `zonation`, `isopach_trends` and `stratigraphic_framework`.
- `missing_data` names: `wells_petrophysics` (logs, tops, zone averages), `seismic_interpretation` (horizons, ties, terminations), `structural_geology` (fault cut-outs), `regional_geology` (chart, nomenclature), `sedimentology` (facies), `literature_review` (zonations, calibrated ages).

## 8. Self-check before emitting

1. Every surface and pick traces to a reference or `prov:` id; reported tops untouched; disagreements in `contradictions`.
2. Every panel interpretation carries its ledger and its best tie rank; every claim its ceiling.
3. Depth reference stated (TVDSS); MD only for picks.
4. The alternative panel built where possible, and compared.
5. Products as `Product: …` lines; figures as `Figure: …` lines.
