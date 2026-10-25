---
name: regional-basin
description: Backstripping and decompaction, McKenzie stretching fits, the basin-evidence table and the burial_history product. Also the parameters that must be sourced, and the forms of subsidence that distinguish basin classes. Load before reg_backstrip, reg_stretching_fit or reg_basin_evidence.
---

# Basin analysis

## Backstripping: `reg_backstrip`

- **The tool reads a supplied column by its path**: `units_table="<path in the shared inputs folder>"`. It is a CSV with the unit fields as headers (name, top_below_surface_m, base_below_surface_m, lithology, age_top_ma, age_base_ma, water_depth_m, sea_level_m).
  - The depths are below the sediment surface (the seabed offshore), not TVDSS. Subtract the water depth from TVDSS tops first.
  - You cannot open the file yourself; the tool reads it.
  - Type `units` only for a column built from the literature or a pseudo-well.
- The column (youngest first) gives, per unit: present depths below the sediment surface, lithology (or compaction parameters), ages top and base, palaeo-water depth and eustatic sea level.
  - Ages come from the chart or the synthesis (cite).
  - Depths come from a well or a pseudo-well (cite).
  - Palaeo-water depths come from facies evidence (cite, or state the assumption).
- Compaction: Sclater & Christie (1980) Table A1a per lithology (shale 0.63 / 0.51 per km, sandstone 0.49 / 0.27, shaly sandstone 0.56 / 0.39, chalk 0.70 / 0.71). Salt does not compact.
  - The table gives no limestone or basalt parameters (chalk is not limestone). For those, and whenever you have measured values, give `phi0`, `c_per_km` and `rho_grain` with `compaction_source`.
  - The tool returns each unit's parameters with their basis.
- The result's `history` gives, at each unit's top age, the decompacted thickness, the water depth, the total subsidence and the **tectonic (water-loaded) subsidence** by Airy isostasy.
- **State the assumptions**: Airy (no flexure), the palaeo-water depths and sea levels, and the compaction parameters. A different water depth of 200 m moves tectonic subsidence by 200 m.
- The tool draws the subsidence curve (total and tectonic subsidence against age) and returns it in `result.figures`. With `rift_age_ma`, the curve also marks the McKenzie fit, beta and the rift interval. `reg_render(kind=subsidence, product=<burial_history>)` draws it again from the product.

## Stretching: `reg_stretching_fit`

- McKenzie (1978), uniform instantaneous stretching: an initial fault-controlled subsidence, plus thermal subsidence that decays with a 63 Myr time constant. Give the rift age (and end) from the chart. The fit uses post-rift points only.
- Read `beta`, `rms_misfit_m` and `beta_range_within_misfit`. A broad range means that the history does not constrain beta. Say so.
- Protracted or multi-phase rifting, flexure and magmatism break the model's assumptions: report the fit as the uniform-stretching equivalent, not the history.
- `reg_stretching_fit` publishes no product and draws no figure. For a figure of the fit, give `rift_age_ma` to `reg_backstrip`.

## Basin classification: `reg_basin_evidence`

The table sets your measured values beside the expectations for rift, passive margin, foreland, strike-slip and intracratonic basins. The measured values are sediment thickness from GlobSed, crustal thickness, heat flow, stress regime, the subsidence form from the burial history, and seismicity.

Each expectation has its `basis`: a publication (McKenzie 1978, Steckler & Watts 1978, Allen & Allen 2005 with page) or "Geo Oracle summary; no source recorded". Cite the latter as such, never as a published classification.

The class is your `interpretation`. Carry the alternatives that fit the same evidence in `competing_interpretations`. The table alone never decides.

## Products and transfer

- `burial_history` is a regional prior for subsurface_play (maturity timing). The sidecar carries the compaction and stretching parameters.
- T2 matters here. A burial history at a well 60 km away transfers only if the units are equivalent and no local event (salt, inversion) intervenes.

<!-- end of skill: regional-basin -->
