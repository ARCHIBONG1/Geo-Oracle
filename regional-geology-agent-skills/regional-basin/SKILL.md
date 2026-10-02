---
name: regional-basin
description: Backstripping and decompaction, McKenzie stretching fits, the basin-evidence table and the burial_history product; parameters that must be sourced and the forms of subsidence that distinguish basin classes. Load before reg_backstrip, reg_stretching_fit or reg_basin_evidence.
---

# Basin analysis

## Backstripping: `reg_backstrip`

- The column (youngest first): present depths below the sediment surface, lithology (or compaction parameters), ages top and base, palaeo-water depth and eustatic sea level per unit. Ages come from the chart or the synthesis (cite), depths from a well or a pseudo-well (cite), palaeo-water depths from facies evidence (cite, or state the assumption).
- Compaction: Sclater & Christie (1980) parameters per lithology (shale 0.63 / 0.51 per km, sandstone 0.49 / 0.27, limestone and chalk 0.70 / 0.71; salt does not compact). Give `phi0`, `c_per_km` and `rho_grain` when you have measured values.
- The result's `history` gives, at each unit's top age, the decompacted thickness, the water depth, the total subsidence and the **tectonic (water-loaded) subsidence** by Airy isostasy.
- **State the assumptions**: Airy (no flexure), the palaeo-water depths and sea levels, and the compaction parameters. A different water depth of 200 m moves tectonic subsidence by 200 m.

## Stretching: `reg_stretching_fit`

- McKenzie (1978), uniform instantaneous stretching: an initial fault-controlled subsidence plus thermal subsidence decaying with a 63 Myr time constant. Give the rift age (and end) from the chart; the fit uses post-rift points only.
- Read `beta`, `rms_misfit_m` and `beta_range_within_misfit`. A broad range means the history does not constrain beta; say so.
- Protracted or multi-phase rifting, flexure and magmatism break the model's assumptions: report the fit as the uniform-stretching equivalent, not the history.

## Basin classification: `reg_basin_evidence`

The table sets your measured values (sediment thickness from GlobSed, crustal thickness, heat flow, stress regime, the subsidence form from the burial history, seismicity) beside the published expectations for rift, passive margin, foreland, strike-slip and intracratonic basins. The class is your `interpretation`, with the alternatives that fit the same evidence carried in `competing_interpretations`; the table alone never decides.

## Products and transfer

- `burial_history` is a regional prior for subsurface_play (maturity timing). The sidecar carries the compaction and stretching parameters.
- T2 matters here: a burial history at a well 60 km away transfers only if the units are equivalent and no local event (salt, inversion) intervenes.
