---
name: structural-modelling
description: Building the structural model (fault planes, horizons per fault block), the validity tests V1 and V7, seeded realizations within pick uncertainty, closures and spill points with fault-bounded cases, and the structural_model and trap_geometry products. Load before sg_build_model or sg_closure_spill.
---

# Structural modelling, closures and spill

## `sg_build_model`

- Fault planes through the traces at the fault_set's dips; the horizon interpolated per fault block by thin-plate RBF through the picks (depth only).
- **V7 data honouring**: the maximum misfit to the picks against the pick uncertainty (interpolation through the picks gives zero; a smoothed model would not).
- **V1 geometric consistency**: cutoffs present on both sides along each fault with a consistent downthrown side; two horizons do not cross (give `second_horizon`).
- `trace_vs_strike_misfit` flags a trace that is not perpendicular to the dip azimuth; the plane follows the trace and the misfit is reported.
- The product `structural_model` carries the ledger (V1, V7 and the untested ones honestly "not tested") and the model grid file; later tools read the ledger and propagate failures.

## Alternatives

Build one model per plausible interpretation (one fault or two segments; different fault subsets) and compare their ledgers. When both pass the same tests, report both as `unresolved` hypotheses with the discriminating picks requested.

## `sg_closure_spill`

- Flooding in order of depth from the crest (the shallowest cell, or the `crest` given). The spill is the saddle where the closure merges with another high of its own relief, or the grid edge. Both the four-way closure and the fault-bounded closure (fault traces as barriers) are computed; `fault_dependent` says whether they differ beyond the pick uncertainty.
- **Realizations**: seeded, correlated perturbations within the pick uncertainty; ranges of spill, area and relief, and whether the closure survives in all realizations. The seed and count are recorded; the same seed reproduces the ranges exactly.
- Depth only: a time-domain surface is refused. Closures at the grid edge are open: say so.
- Report ranges, never one number: "spill 528-532 m, closed area 0.24-0.27 km2 over 20 realizations (seed 7); closed in all".

## Reporting

- `[interpretation, derived]`: "Model dome-v1: F1 planar, 2 blocks; V1 pass, V7 pass (max misfit 0 m, uncertainty 2.5 m), V2 pass, V3-V6 not tested", product `structural_model`.
- `[measurement, derived]`: closure as ranges, product `trap_geometry`, for prospect_target and risk work; the ceiling: the horizon's pick uncertainty and the fault set it depends on.
- Closure map with `sg_render(kind=closure, fault_set=...)`.
