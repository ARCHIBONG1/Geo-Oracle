---
name: strat-log-correlation
description: How to rank tie points and correlate wells by dynamic time warping constrained to the anchors. It also covers how to build and test correlation panels, the alternative panel, and how to read the C6 support. Load before stg_rank_ties, stg_dtw_chain, stg_dtw_pair, stg_build_panel, stg_consistency or stg_compare_panels.
---

# Log correlation

## Order of work

1. Run `stg_rank_ties` with the logs, the registered tables and the seismic ties. It gives every candidate tie per well with its rank. `shared_between_wells` shows which ties can anchor a pair. The tool lists reported tops for reference, not as ties, unless you argue that a top marks a key surface (rank 4).
2. Choose the reference well: the most complete one, or the one with the most rank 1-3 ties. Pick the surfaces there, with their basis.
3. Run `stg_dtw_chain` once, from the reference to every other well:
   - `surfaces` = the surfaces' TVDSS in the reference well;
   - `anchors_by_well` = the rank 1-3 ties that each well shares with the reference;
   - `tie_rank` 2 or 3 for mappings between such anchors (5 unanchored).

   It returns `picks` in the shape that `stg_build_panel` takes, and `c6_results` in the shape that `stg_consistency` takes. Use `stg_dtw_pair` for one pair or an interval-specific rerun.
4. Run `stg_build_panel` with those picks and the `well_tops` products. It draws the panel figure itself, and returns it in `result.figures`.
5. Run `stg_consistency` with the C6 results and any thickness explanations. With `age_models`, it also draws the panel's Wheeler diagram in `result.figures`. Then build the alternative panel, then run `stg_compare_panels`.

## DTW: what it does and does not do

- DTW aligns one curve (GR by default) between two wells within a band (`band_m`, about the largest plausible thickness change). It aligns by segments between the anchors, so the path passes through every anchor. The mapping is monotonic.
- `mean_cost` is Sakoe & Chiba's symmetric time-normalised distance: the accumulated z-scored mismatch over the sum of the two lengths. On an unstretched path, it is the mean mismatch, the same measure as the rigid-shift costs.
- `c6.percentile` is where that cost sits among rigid shifts of the same curve. The tool gives the rigid costs themselves beside it.
- The bar (`c6_percentile`, a quarter by default) is a stated convention, not a measurement. Loosen it for a short or noisy interval, and say that you did. Above the bar, the pattern match is no better than a shift, and the surfaces between the anchors are rank-5 at best.
- A DTW match between rank 1-3 anchors inherits rank 2-3 for surfaces close to the anchors, and rank 5 for surfaces far from them. Say which.
- **The aligned span** runs from the first anchor to the end of the reference log. Between anchors, and below the last anchor down to the ends of both logs, surfaces are pattern-matched.
- **Above the first anchor** (or beyond the end of the reference log), nothing is pattern-matched. `stg_dtw_chain` extrapolates such a surface through the nearest anchors and the end of the alignment. It gives the surface rank 5, whatever `tie_rank` you asked for, lists it in `outside_aligned_span`, and warns. Add a tie point above it, or report the pick as an extrapolation.
- Below the last anchor, the match assumes that the two logs end at corresponding levels. With one anchor only, check the deepest surfaces against another line of evidence.
- The tool refuses anchors whose depths reverse between the wells: a tie is misidentified, or the section is repeated or faulted.
- Repeating patterns (parasequence sets) can be matched one cycle off. Anchors are what prevent it. Without anchors, build two panels one cycle apart, and let C2 and C4 decide.

## Panels

- Picks carry `tie_rank`, `basis` and `source` (the `prov:` id of the DTW or the table). Reported tops enter tagged reported from the `well_tops` products, never as picks.
- A surface's name is the datum it follows, not a unit it may cross (see `strat-core`, section 2a). A time line mapped from a well's sand top is `time line at <well> <surface>`. Then, where rock and time lines diverge, the panel reads as a divergence, not as an error in someone's pick.
- `datum`: the surface to hang the rendered panel on. A time line (rank 1-2) is the right datum. If you hang the panel on a rock line, you manufacture thickness changes.
- C4 anomalies: a thickness change above 50 % and 10 m between neighbouring wells. If a fault cut-out (structural product), erosion or growth explains it, give `thickness_explanations` keyed `<top>-<base>@<wellA>-<wellB>`. Otherwise, the panel fails.

## The alternative

The alternative is a lithostratigraphic correlation (the operator's tops joined as surfaces), tested beside the time-line panel. If it crosses a time line, it fails C2. Report it as the contradicted alternative, with the reason. Two panels that pass the same tests are `unresolved`.

## Reporting

- `[measurement, derived]`: "DTW W1-W3 (GR, anchored on LO Globotruncana and LO Nannoconus): cost 0.05, best quarter of rigid shifts (C6 pass)", `prov:` id.
- `[interpretation, derived]`: "FS2 correlates to 1280.0 m TVDSS (1320.7 m MD) in W3, rank 2; panel chrono-v1: C1 pass, C2 pass, C3 not tested, C4 pass, C5 not tested, C6 pass, C7 not tested", product `correlated_tops`.
- `[hypothesis]`: the alternative, with its failing test.

<!-- end of skill: strat-log-correlation -->
