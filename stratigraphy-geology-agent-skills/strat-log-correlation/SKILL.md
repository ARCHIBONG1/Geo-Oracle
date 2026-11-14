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
   - `anchors_by_well` = the rank 1-3 ties that each well shares with the reference. Or give `rank_ties` = the `provenance_id` of `stg_rank_ties`: the tool then takes the shared rank 1-3 ties as anchors itself;
   - each anchor's own `rank` (1 dated, 2 biostratigraphic, 3 seismic). An anchor without one takes `tie_rank`, and the tool warns. Only a surface within one step of an anchor takes the anchor's rank, in every well (the reference well too). Every other pick is rank 5.

   It returns `picks` in the shape that `stg_build_panel` takes, and `c6_results` in the shape that `stg_consistency` takes. A long reply is shortened: the complete picks and C6 results are in the file `outputs.picks` (`panels/...json`). Use `stg_dtw_pair` for one pair or an interval-specific rerun.
4. Run `stg_build_panel` with those picks and the `well_tops` products. It draws the panel figure itself, and returns it in `result.figures`.
5. Run `stg_consistency` with the C6 results and any thickness explanations. With `age_models`, it also draws the panel's Wheeler diagram in `result.figures`. Then build the alternative panel, then run `stg_compare_panels`.

## DTW: what it does and does not do

- DTW aligns one curve (GR by default) between two wells within a band (`band_m`, about the largest plausible thickness change). It aligns by segments between the anchors, so the path passes through every anchor. The mapping is monotonic.
- `mean_cost` is Sakoe & Chiba's symmetric time-normalised distance: the accumulated z-scored mismatch over the sum of the two lengths. On an unstretched path, it is the mean mismatch. It is the alignment's cost (`c6.alignment_cost`), not the cost that C6 ranks (`c6.dtw_cost`, on C6's own window).
- C6 tests whether the two logs are related over a window; it does not check the mapping that the tool returns (`c6.tests`). The window of A is the overlap of the wells (shifted by the anchors' mean offset). The window of B is the best rigid fit within `band_m` of it, found the same way for B and for each of 30 surrogates of B (`null: surrogate`): curves with B's values and nearly B's spectrum, with random phases. Both curves are detrended, so a shared linear depth trend does not count as a match. A shared curved trend or step stays. `c6.percentile` is where the DTW cost on the window (`c6.dtw_cost`) sits among the surrogates' costs. If the returned mapping's offset differs from the offset C6 found (`c6.window_offset_m`) by more than half of `band_m`, the tool warns. `best_rigid_cost` is information only.
- The level (`c6_percentile`, 5 by default) is a stated convention. With 30 surrogates, level 5 passes a pair when at most one surrogate fits as well. The nominal false-pass rate is 0.065; the rate measured on synthetic logs is higher, 6-13 % (`level_meaning` gives both, and the measured pass rate of related logs). Say which level you used. A C6 failure means that log similarity is not shown: the log-derived (rank-5) picks of that pair are not supported, and a panel that holds them is `proposed` (not contradicted). A panel of rank 1-3 ties only does not change. Measured cost: C6 failed no correct mapping at noise up to 60 % of the signal, and about 11-14 % of related pairs at noise equal to the signal. A window shorter than four times `band_m` leaves C6 not tested, and the tool says why.
- A DTW match is a log-pattern match: rank 5, also between rank 1-3 anchors. A surface takes the anchor's rank only at the anchor (within one step). To give a surface rank 1-3, tie it with a rank 1-3 tie in each well.
- **The aligned span** runs from the first anchor to the end of the reference log. Between anchors, and below the last anchor, surfaces are pattern-matched.
- **Above the first anchor** (or beyond the end of the reference log), nothing is pattern-matched. `stg_dtw_chain` extrapolates such a surface through the nearest anchors and the end of the alignment. It gives the surface rank 5, whatever `tie_rank` you asked for, lists it in `outside_aligned_span`, and warns. Add a tie point above it, or report the pick as an extrapolation.
- Below the last anchor (and without anchors), the tool tries two ends: tied to the other log's end (pinned), and open within `band_m` of the depth that the last anchor's offset gives (equal TVDSS without anchors). It keeps the one that fits better while the stretch the pinned end implies is at most 1.3. Between 1.3 and 2 it also keeps the pinned end when that fits better, and it warns (`pinned_beyond_cap`). Above 2 (or below 1/2) it never keeps the pinned end. If that end fits better, the tool warns (`pinned_beyond_stretch_limit`): the open end below may be wrong, so add a tie point below. A short overlap against a long log also gives this warning. `open_segments` gives `end_condition`, both scores and `end_offset_m`. If the two ends fit about equally well but map differently (`end_condition_undetermined`), the tool warns. Treat it as a caution: check the picks below the last anchor against an anchor or another line of evidence. It is not a reliable detector of a wrong end: under noise it fires mostly on correct results. An offset or stretch beyond `band_m` with ends that do not correspond is not followed by either end. With `interval_a` and no anchor in or next to the interval, the result assumes that the offset at the interval is within `band_m`, and the tool warns. Add an anchor there, or widen `band_m`. With one anchor only, check the deepest surfaces against another line of evidence.
- The tool refuses anchors whose depths reverse between the wells: a tie is misidentified, or the section is repeated or faulted.
- Repeating patterns (parasequence sets) can be matched one cycle off. Anchors are what prevent it. Without anchors, build two panels one cycle apart, and let C2 and C4 decide.

## Panels

- Picks carry `tie_rank`, `basis` and `source` (the `prov:` id of the DTW or the table). Reported tops enter tagged reported from the `well_tops` products, never as picks.
- A surface's name is the datum it follows, not a unit it may cross (see `strat-core`, section 2a). A time line mapped from a well's sand top is `time line at <well> <surface>`. Then, where rock and time lines diverge, the panel reads as a divergence, not as an error in someone's pick.
- `datum`: the surface to hang the rendered panel on. A time line (rank 1-2) is the right datum. If you hang the panel on a rock line, you manufacture thickness changes.
- C4 anomalies: a thickness change above 50 % and 10 m between neighbouring wells. If a fault cut-out (structural product), erosion or growth explains it, give `thickness_explanations` keyed `<top>-<base>@<wellA>-<wellB>` (either well order). Each explanation must cite its evidence: the full `prov:` id of a `stg_gap_vs_fault` or `stg_eroded_thickness` result, or an existing product reference. The tool checks that it resolves. Without it, or with no explanation, the panel fails.

## The alternative

The alternative is a lithostratigraphic correlation (the operator's tops joined as surfaces), tested beside the time-line panel. If it crosses a time line, it fails C2. Report it as the contradicted alternative, with the reason. Two panels that pass the same tests are `unresolved`.

## Reporting

- `[measurement, derived]`: "DTW W1-W3 (GR, anchored on LO Globotruncana and LO Nannoconus): window cost (c6.dtw_cost) 0.05, below all but one of 30 surrogates at level 5 (C6 pass: the logs are related over the window; the mapping is not tested by C6)", `prov:` id.
- `[interpretation, derived]`: "FS2 correlates to 1280.0 m TVDSS (1320.7 m MD) in W3, rank 5 (DTW between rank-2 anchors); panel chrono-v1: C1 pass, C2 pass, C3 not tested, C4 pass, C5 not tested, C6 pass, C7 not tested", product `correlated_tops`.
- `[hypothesis]`: the alternative, with its failing test.

<!-- end of skill: strat-log-correlation -->
