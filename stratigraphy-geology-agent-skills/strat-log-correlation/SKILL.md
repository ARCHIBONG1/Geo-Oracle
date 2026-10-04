---
name: strat-log-correlation
description: Ranking tie points, correlating wells by dynamic time warping constrained to the anchors, building and testing correlation panels, the alternative panel, and reading the C6 support. Load before stg_rank_ties, stg_dtw_pair, stg_build_panel, stg_consistency or stg_compare_panels.
---

# Log correlation

## Order of work

1. `stg_rank_ties` with the logs, the registered tables and the seismic ties: every candidate tie per well with its rank; `shared_between_wells` shows which can anchor a pair. Reported tops are listed for reference, not as ties, unless you argue a top marks a key surface (rank 4).
2. Choose the reference well (the most complete, or the one with the most rank 1-3 ties) and pick the surfaces there, with their basis.
3. `stg_dtw_chain` once, from the reference to every other well: `surfaces` = the surfaces' TVDSS in the reference well, `anchors_by_well` = the rank 1-3 ties each well shares with the reference, `tie_rank` 2 or 3 for mappings between such anchors (5 unanchored). It returns `picks` in the shape `stg_build_panel` takes and `c6_results` in the shape `stg_consistency` takes. `stg_dtw_pair` is for one pair or an interval-specific rerun.
4. `stg_build_panel` with those picks and the `well_tops` products.
5. `stg_consistency` with the C6 results and any thickness explanations. Then the alternative panel, then `stg_compare_panels`.

## DTW: what it does and does not do

- Aligns one curve (GR by default) between two wells within a band (`band_m`, about the largest plausible thickness change), by segments between the anchors, so the path passes through every anchor. The mapping is monotonic.
- `mean_cost` is the mean z-scored mismatch along the path; `c6.percentile` is where that cost sits among rigid shifts of the same curve. Below 25 % passes C6; above, the pattern match is no better than a shift and the surfaces between the anchors are rank-5 at best.
- A DTW match between rank 1-3 anchors inherits rank 2-3 for surfaces close to the anchors and rank 5 for surfaces far from them; say which.
- Repeating patterns (parasequence sets) can be matched one cycle off: anchors are what prevent it. Without anchors, build two panels one cycle apart and let C2 and C4 decide.

## Panels

- Picks carry `tie_rank`, `basis` and `source` (the `prov:` id of the DTW or the table). Reported tops enter tagged reported from the `well_tops` products, never as picks.
- `datum`: the surface to hang the rendered panel on; a time line (rank 1-2) is the right datum; hanging on a rock line manufactures thickness changes.
- C4 anomalies: a thickness change above 50 % and 10 m between neighbouring wells; give `thickness_explanations` keyed `<top>-<base>@<wellA>-<wellB>` when a fault cut-out (structural product), erosion or growth explains it; otherwise the panel fails.

## The alternative

A lithostratigraphic correlation (the operator's tops joined as surfaces) tested beside the time-line panel; if it crosses a time line it fails C2 and is reported as the contradicted alternative with the reason. Two panels passing the same tests are `unresolved`.

## Reporting

- `[measurement, derived]`: "DTW W1-W3 (GR, anchored on LO Globotruncana and LO Nannoconus): cost 0.05, best quarter of rigid shifts (C6 pass)", `prov:` id.
- `[interpretation, derived]`: "FS2 correlates to 1280.0 m TVDSS (1320.7 m MD) in W3, rank 2; panel chrono-v1: C1 pass, C2 pass, C3 not tested, C4 pass, C5 not tested, C6 pass, C7 not tested", product `correlated_tops`.
- `[hypothesis]`: the alternative, with its failing test.
