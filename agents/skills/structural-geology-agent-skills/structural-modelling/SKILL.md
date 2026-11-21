---
name: structural-modelling
description: How to build the structural model (fault planes, horizons per fault block). It also covers the validity tests V1 and V7, seeded realizations within pick uncertainty, and closures and spill points with fault-bounded cases. It covers the model's V1 and V7 carried into the closure, and the structural_model and trap_geometry products. Load before sg_build_model or sg_closure_spill.
---

# Structural modelling, closures and spill

## `sg_build_model`

- Fault planes go through the traces at the fault_set's dips. The tool interpolates the horizon per fault block by thin-plate RBF through the picks (depth only).
- A model without faults puts every pick in one block. A block with more than 3,000 picks is fitted locally from the nearest picks, and the result says so.
- **V7 data honouring**: the spline passes through every pick, so the misfit at the picks is zero by construction, and is not the test. V7 rebuilds the model without every tenth pick. It passes when both of these hold:
  - the model predicts those held-out picks with an RMS within twice the pick uncertainty;
  - every pick lies in a modelled block. A fault block with fewer than 4 picks is not modelled, and fails V7.

  Read `holdout` in the result.
- **V1 geometric consistency**: cutoffs present on both sides along each fault, with a consistent downthrown side; two horizons do not cross (give `second_horizon`).
- If the tool cannot read a throw on a fault (for example, another fault is too near), it still builds the model. V1 fails for that fault, and `throw_unreadable` names it. Merge the two faults if they are one fault zone. Leave a fault out with `fault_ids` only when its throw cannot be read. Report each fault that you leave out. The result and the product name it in `faults_excluded`.
- `trace_vs_strike_misfit` flags a trace that is not perpendicular to the dip azimuth. The plane follows the trace, and the result gives the misfit.
- The product `structural_model` carries the ledger (V1, V7 and the untested ones honestly "not tested") and the model grid file. Later tools read the ledger and propagate failures.

## Alternatives

Build one model per plausible interpretation (one fault or two segments; different fault subsets), and compare their ledgers. If both pass the same tests, report both as `unresolved` hypotheses, and request the discriminating picks.

## `sg_closure_spill`

- **Give the wells.** Pass `wells=[...]` with the `well_header` product of every well of the study (`@petrophysics-agent/products/<file>`). The result then says for each well whether it lies inside the closure, the horizon depth at the well, and its distance and bearing from the crest, and the closure map shows the wells. Report those in your findings: a closure that cannot be placed from the wells cannot be used to say where to drill. When no `well_header` product is in your brief, say so in `missing_data` with the owner `wells_petrophysics`. The positions are surface positions: say so for a deviated well.
- **Give the model.** Build `sg_build_model` on the horizon first, then call `sg_closure_spill` on the same horizon with `model=<that structural_model>` whenever a prospect will use the closure. The model's V1 and V7 then carry into the `trap_geometry` validity record (`validity_from_model` in the result), and prospect maturity (R2) needs them passed. Without `model` they stay "not tested" and the candidate stays a lead. A model built on another horizon is refused: each horizon needs its own model.
- The tool floods in order of depth from a crest.
  - The tool computes each closure twice: `four_way` (no barrier) and `fault_bounded` (the fault traces as barriers). `fault_dependent` is true when they differ beyond the pick uncertainty, or when an offset across a fault closes part of the closure's outline. A fold limb without an offset is not a fault step.
  - If no `crest` is given, the tool tries every local crest of the surface, shallowest first. A local crest is a cell shallower than every cell within 3 cells on its side of any fault, not on the data edge.
    - `closures` lists each closure found: crest, spill, relief, area, the `four_way` and `fault_bounded` readings, `kind`, `fault_dependent`, `held_by` relief or size.
    - `kind` is "four-way", "fault-bounded" or "fault-bounded only". A "fault-bounded" closure closes without the barrier. But on part of its outline, the horizon is offset across a fault, and the offset closes it there (`fault_contact` gives the throw there). `fault_perimeter_fraction` is the share of outline cells found sealed so. It changes with where the trace and the cells fall: read only whether it is above 0. When `kept_on_offset_only` is true, the sealing fault does not reach the horizon by its published depth range: the horizon may be folded, not cut, there. Report both readings. The numbers of a "fault-bounded" closure are under `four_way` (`kind_numbers_key`). A "fault-bounded only" closure closes only against the barrier. Its numbers are under `fault_bounded`. Both depend on fault seal.
    - The largest closure that closes without the barrier ("four-way" or "fault-bounded") is the base case. If none does, the largest "fault-bounded only" closure is the base case (`crest_basis` says so).
  - Give `crest` for one closure in particular. That crest is then the base case.
  - A dipping surface has no local crest and returns none: its shallowest cell is the up-dip edge, not a trap.
  - The spill is the saddle where the closure merges with another high of its own relief (`saddle`), a saddle into a low that reaches the data edge (`open_basin`: a real spill point when held by relief), or the data edge itself (`data_edge`: report minimum values). `spill_note` gives the reading.
  - If a crest's basin reaches the data edge after less than `min_relief_m` of relief, the crest is within pick noise. The result gives it as not closed. Say so in your findings.
- **Every closure, not only the largest.** Report each entry of `closures` with its own spill and area, and say which is the base case. A listed closure that is not the base case has single values, not ranges; its `realizations_closed` says in how many realizations it survives. Besides the base case, `counted_closures` listed closures are counted (default 10: those at a fault edge first, then the largest relief). Each other one has `realizations_not_run` instead: report its survival as not tested, or raise `counted_closures` when that closure matters. Only the base case has realization ranges. `at_fault_edge` (flag `CLOSURE_CREST_AT_FAULT_EDGE`) marks a closure most of whose cells lie within 2 cells of a fault trace: the picks there are the least reliable, so it can be a horizon picked across the fault edge. It is not the base case unless every closure is one. Report it as doubtful. A fault-bounded trap whose crest only touches the fault is not marked. A large closure of modest relief can be missed when pick-noise pimples take every candidate place and its relief is under the relief rule. So a basin holds by relief (`min_relief_m`, default 6 x the pick jitter measured on the surface, or 6 x the stated pick uncertainty when the jitter cannot be measured: `min_relief_basis`) or by size: at least 2 x the pick uncertainty of relief over at least `large_closure_cells` cells (default 250; `size_rule` in the result). The tool lists candidates below both in `weak_closures`, with `why_weak`. Report them as weak, never drop them.
- **Every horizon asked for.** Run closures on each mapped horizon that the brief names (each of them, not the first alone). Name a horizon not run in limitations, with the reason.
- **Realizations**: seeded, correlated perturbations within the pick uncertainty, run on the base case. The default count is 20. Each realization is judged by the jitter measured on it. They give ranges of spill, area and relief, and whether the closure survives in all realizations. The result records the seed and count. The same seed reproduces the ranges exactly.
  - By default the realizations hold pick noise only (`realization_basis`). Depth conversion usually gives the larger error. When the depth conversion has a known error (for example the well mistie residuals), give `depth_error_m` with `depth_error_basis` (and `depth_error_range_m`, default 1000 m). The result then also gives P10, P50 and P90 (`percentiles`). For these, use 100 realizations or more.
- Unpicked nodes inside the grid: the tool bridges an enclosed hole of up to `max_hole_cells` nodes (default 2). It floods twice, with each hole node filled from its own neighbours, at the deepest and at the shallowest of them. When the two differ, the headline is the smaller closure, and `fill_range` gives the range. `unbridged` is the reading without bridging. When it differs, `closure_range` gives the range over the readings. Report that range, not the headline alone. The closure usually lies in it: a crest wholly under a bridged patch is not seen, so the true closure can be the smaller reading. It can also be larger when an unpicked node at the spill point is deeper than its neighbours. A `lower_end` note means there is no closure without bridging. Say that the closure assumes the horizon is continuous at the named nodes. `CREST_HIDDEN_BY_UNPICKED_PATCH`: a crest beside a larger patch was not tried. Say that a closure there is not excluded.
- Depth only: the tool refuses a time-domain surface. Closures at the grid edge are open: say so.
- Report the base case as ranges, never one number: "spill 528-532 m, closed area 0.24-0.27 km2 over 20 realizations (seed 7); closed in all".

## Reporting

- `[interpretation, derived]`: "Model dome-v1: F1 planar, 2 blocks; V1 pass, V7 pass (holdout RMS 1.1 m against 2 x 2.5 m; every pick modelled), V2 pass, V3-V6 not tested", product `structural_model`.
- `[measurement, derived]`: closure as ranges, product `trap_geometry` with the model's V1 and V7, for prospect_target and risk work. The ceiling: the horizon's pick uncertainty, the fault set and the model it depends on.
- `sg_closure_spill` draws the closure map itself when a closure is found (`result.figures`).

<!-- end of skill: structural-modelling -->
