---
name: structural-modelling
description: How to build the structural model (fault planes, horizons per fault block). It also covers the validity tests V1 and V7, seeded realizations within pick uncertainty, and closures and spill points with fault-bounded cases. It covers the model's V1 and V7 carried into the closure, and the structural_model and trap_geometry products. Load before sg_build_model or sg_closure_spill.
---

# Structural modelling, closures and spill

## `sg_build_model`

- Fault planes go through the traces at the fault_set's dips. The tool interpolates the horizon per fault block by thin-plate RBF through the picks (depth only).
- **V7 data honouring**: the spline passes through every pick, so the misfit at the picks is zero by construction, and is not the test. V7 rebuilds the model without every tenth pick. It passes when both of these hold:
  - the model predicts those held-out picks with an RMS within twice the pick uncertainty;
  - every pick lies in a modelled block. A fault block with fewer than 4 picks is not modelled, and fails V7.

  Read `holdout` in the result.
- **V1 geometric consistency**: cutoffs present on both sides along each fault, with a consistent downthrown side; two horizons do not cross (give `second_horizon`).
- `trace_vs_strike_misfit` flags a trace that is not perpendicular to the dip azimuth. The plane follows the trace, and the result gives the misfit.
- The product `structural_model` carries the ledger (V1, V7 and the untested ones honestly "not tested") and the model grid file. Later tools read the ledger and propagate failures.

## Alternatives

Build one model per plausible interpretation (one fault or two segments; different fault subsets), and compare their ledgers. If both pass the same tests, report both as `unresolved` hypotheses, and request the discriminating picks.

## `sg_closure_spill`

- **Give the model.** Build `sg_build_model` on the horizon first, then call `sg_closure_spill` on the same horizon with `model=<that structural_model>` whenever a prospect will use the closure. The model's V1 and V7 then carry into the `trap_geometry` validity record (`validity_from_model` in the result), and prospect maturity (R2) needs them passed. Without `model` they stay "not tested" and the candidate stays a lead. A model built on another horizon is refused: each horizon needs its own model.
- The tool floods in order of depth from a crest.
  - If no `crest` is given, the tool tries every local crest of the surface, shallowest first. A local crest is a cell shallower than every cell within 3 cells on its side of any fault, not on the data edge.
    - `closures` lists each closure found: crest, spill, relief, area, four-way and fault-bounded, `kind` "four-way" or "fault-bounded only", `held_by` relief or size.
    - The largest four-way closure is the base case. If none closes four-way, the largest fault-bounded one is the base case (`crest_basis` says so).
- **Every closure, not only the largest.** Report each entry of `closures` with its own spill and area, and say which is the base case. A large closure of modest relief can be missed when pick-noise pimples take every candidate place and its relief is under the relief rule. So a basin holds by relief (`min_relief_m`, default 6 x the pick uncertainty) or by size: at least 2 x the pick uncertainty of relief over at least `large_closure_cells` cells (default 250; `size_rule` in the result). The tool lists candidates below both in `weak_closures`, with `why_weak`. Report them as weak, never drop them.
- **Every horizon asked for.** Run closures on each mapped horizon that the brief names (each of them, not the first alone). Name a horizon not run in limitations, with the reason.
  - A dipping surface has no local crest and returns none: its shallowest cell is the up-dip edge, not a trap.
  - Give `crest` for one closure in particular.
  - The spill is the saddle where the closure merges with another high of its own relief, or the grid edge.
  - If a crest's basin reaches the data edge after less than `min_relief_m` of relief, the crest is within pick noise. The result gives it as not closed. Say so in your findings.
  - The tool computes both the four-way closure and the fault-bounded closure (fault traces as barriers). `fault_dependent` says whether they differ beyond the pick uncertainty.
- **Realizations**: seeded, correlated perturbations within the pick uncertainty. They give ranges of spill, area and relief, and whether the closure survives in all realizations. The result records the seed and count. The same seed reproduces the ranges exactly.
- Depth only: the tool refuses a time-domain surface. Closures at the grid edge are open: say so.
- Report ranges, never one number: "spill 528-532 m, closed area 0.24-0.27 km2 over 20 realizations (seed 7); closed in all".

## Reporting

- `[interpretation, derived]`: "Model dome-v1: F1 planar, 2 blocks; V1 pass, V7 pass (holdout RMS 1.1 m against 2 x 2.5 m; every pick modelled), V2 pass, V3-V6 not tested", product `structural_model`.
- `[measurement, derived]`: closure as ranges, product `trap_geometry` with the model's V1 and V7, for prospect_target and risk work. The ceiling: the horizon's pick uncertainty, the fault set and the model it depends on.
- `sg_closure_spill` draws the closure map itself when a closure is found (`result.figures`).

<!-- end of skill: structural-modelling -->
