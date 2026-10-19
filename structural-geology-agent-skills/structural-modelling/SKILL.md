---
name: structural-modelling
description: Building the structural model (fault planes, horizons per fault block), the validity tests V1 and V7, seeded realizations within pick uncertainty, closures and spill points with fault-bounded cases, and the structural_model and trap_geometry products. Load before sg_build_model or sg_closure_spill.
---

# Structural modelling, closures and spill

## `sg_build_model`

- Fault planes through the traces at the fault_set's dips; the horizon interpolated per fault block by thin-plate RBF through the picks (depth only).
- **V7 data honouring**: the spline passes through every pick, so the misfit at the picks is zero by construction and is not the test. V7 rebuilds the model without every tenth pick and passes when it predicts those held-out picks with an RMS within twice the pick uncertainty, and every pick lies in a modelled block (a fault block with fewer than 4 picks is not modelled and fails V7). Read `holdout` in the result.
- **V1 geometric consistency**: cutoffs present on both sides along each fault with a consistent downthrown side; two horizons do not cross (give `second_horizon`).
- `trace_vs_strike_misfit` flags a trace that is not perpendicular to the dip azimuth; the plane follows the trace and the misfit is reported.
- The product `structural_model` carries the ledger (V1, V7 and the untested ones honestly "not tested") and the model grid file; later tools read the ledger and propagate failures.

## Alternatives

Build one model per plausible interpretation (one fault or two segments; different fault subsets) and compare their ledgers. When both pass the same tests, report both as `unresolved` hypotheses with the discriminating picks requested.

## `sg_closure_spill`

- Flooding in order of depth from a crest. With no `crest` given, every local crest of the surface (a cell shallower than every cell within 3 cells on its side of any fault, not on the data edge) is tried, shallowest first: `closures` lists each closure found (crest, spill, relief, area, four-way and fault-bounded, `kind` "four-way" or "fault-bounded only", `held_by` relief or size) and the largest four-way closure is the base case, or the largest fault-bounded one when none closes four-way (`crest_basis` says so).
- **Every closure, not only the largest.** Report each entry of `closures` with its own spill and area, and say which is the base case. On 2026-10-19 the second closure of the Corvane key (15 m of relief over about 680 cells) was missed: pick-noise pimples took every candidate place, and its relief was under the relief rule. A basin now holds by relief (`min_relief_m`, default 6 x the pick uncertainty) or by size: at least 2 x the pick uncertainty of relief over at least `large_closure_cells` cells (default 250; `size_rule` in the result). Candidates below both are listed in `weak_closures` with `why_weak`: report them as weak, never drop them.
- **Every horizon asked for.** Closures on each mapped horizon the brief names (H1 and H2, not H1 alone); a horizon not run is named in limitations with the reason. A dipping surface has no local crest and returns none: its shallowest cell is the up-dip edge, not a trap. Give `crest` for one closure in particular. The spill is the saddle where the closure merges with another high of its own relief, or the grid edge; a crest whose basin reaches the data edge after less than `min_relief_m` of relief is within pick noise and is reported as not closed. Both the four-way closure and the fault-bounded closure (fault traces as barriers) are computed; `fault_dependent` says whether they differ beyond the pick uncertainty.
- **Realizations**: seeded, correlated perturbations within the pick uncertainty; ranges of spill, area and relief, and whether the closure survives in all realizations. The seed and count are recorded; the same seed reproduces the ranges exactly.
- Depth only: a time-domain surface is refused. Closures at the grid edge are open: say so.
- Report ranges, never one number: "spill 528-532 m, closed area 0.24-0.27 km2 over 20 realizations (seed 7); closed in all".

## Reporting

- `[interpretation, derived]`: "Model dome-v1: F1 planar, 2 blocks; V1 pass, V7 pass (holdout RMS 1.1 m against 2 x 2.5 m; every pick modelled), V2 pass, V3-V6 not tested", product `structural_model`.
- `[measurement, derived]`: closure as ranges, product `trap_geometry`, for prospect_target and risk work; the ceiling: the horizon's pick uncertainty and the fault set it depends on.
- Closure map with `sg_render(kind=closure, fault_set=...)`.

<!-- end of skill: structural-modelling -->
