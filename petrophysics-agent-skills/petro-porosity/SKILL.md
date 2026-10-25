---
name: petro-porosity
description: Porosity. It covers choosing density, neutron-density or sonic methods by lithology, fluid and hole condition, and sourcing matrix/fluid/shale points. It also covers the gas correction, total versus effective porosity, and checks against core. Load before petro_porosity or petro_net_pay.
---

# Porosity

## The tool

`petro_porosity(well, method, parameter_set, shale_correction=True, tops)`:
- **Output**: writes `PHIT` (total) and `PHIE` (effective = total minus the shale's apparent porosity, using `VSH`) into a new well file. Run `petro_vshale` first.
- **Gas correction**: with `neutron_density`, the result lists `gas_corrected_intervals`, where the correction acted.
- **Figure**: with `neutron_density`, the tool draws the neutron-density crossplot into `result.figures`.

## Methods

| Method | Equations | Needs | Use when |
|---|---|---|---|
| neutron_density | phiD = (rho_ma - RHOB) / (rho_ma - rho_fl); phiN = (NPHI - nphi_ma) / (1 - nphi_ma); combined as their mean, or root-mean-square (Gaymard-Poupon) where phiD exceeds phiN by more than the gas tolerance (0.02 v/v by design, or the parameter nd_gas_tolerance with its source; the result states which) | rho_ma, rho_fl, nphi_ma (+ rho_sh, nphi_sh) | Default in sands and mixed lithology; handles gas |
| density | phiD alone | rho_ma, rho_fl (+ rho_sh) | Known single mineralogy and liquid-filled; overstates porosity in gas |
| sonic_wyllie | (DT - dt_ma) / (dt_fl - dt_ma) | dt_ma, dt_fl (+ dt_sh) | Bad hole (the sonic is less affected by washouts), consolidated rock; reads mostly intergranular porosity |
| sonic_rhg | The published Raymer-Hunt-Gardner (1980) transform, inverted: V = (1 - phi)^2 Vma + phi Vf below 37 % porosity, the suspension form above 47 %, a linear blend between (their matrix velocities: sandstone 17,850, limestone 20,500, dolomite 22,750 ft/s, i.e. dt_ma 56, 49, 44 us/ft) | dt_ma, dt_fl, rho_ma, rho_fl (+ dt_sh) | Consolidated to unconsolidated sands without a compaction factor; the paper's stated advantage |
| sonic_field | sonic_c (DT - dt_ma) / DT, the field approximation that is NOT in the RHG paper | dt_ma, sonic_c (+ dt_sh) | Only when a sourced C is in hand: the literature range is 0.625-0.70, 0.67 most accepted, 0.60 in gas (AAPG Wiki); register sonic_c with its source |

## Parameters

| Parameter | Preferred source | Otherwise |
|---|---|---|
| rho_ma | Core grain density (`core`) | Known mineral (Schlumberger 2013 Appendix B: quartz 2.64, calcite 2.71, dolomite 2.85; grain densities 2.65/2.71/2.87) as `task` or `assumption` with a range |
| rho_fl | Mud filtrate in the flushed zone (about 1.0-1.1 g/cm3 for water-based mud) | `assumption` with a range |
| nphi_ma | Neutron matrix point on the tool's scale: limestone-calibrated sandstone about -0.035, limestone 0, dolomite about +0.02 | — |
| rho_sh, nphi_sh, dt_sh | Thick in-gauge shale (`log_crossplot`) | — |
| dt_ma, dt_fl | Mineral and fluid slowness (Schlumberger 2013 Appendix B: quartz 56.0, calcite 49.0, dolomite 44.0 us/ft; brine about 189) | — |
| sonic_c | The field-approximation constant, only for sonic_field; cite its source (AAPG Wiki: 0.625-0.70, most accepted 0.67, 0.60 for gas) | — |
| nd_gas_tolerance | Crossover tolerance for the gas correction; omit to use the design value 0.02 v/v | `assumption` with a range if changed |

A wrong matrix density is the commonest porosity error: 0.05 g/cm3 in rho_ma moves density porosity by about 0.03. In mixed lithology, use the crossplot or core, and state it.

## Traps

- **Gas**: density porosity alone reads too high. The neutron-density gas correction brings it back. Report the gas-corrected intervals as observations that support a gas interpretation, together with the crossover.
- **Washouts**: density and neutron read porous (a washout mimics gas). Gas-corrected intervals inside washouts (`petro_qc_logs`) are artefacts. `petro_net_pay` excludes bad hole by default.
- **Shale correction** needs `VSH`. Without it, PHIE = PHIT, and shaly intervals are overstated. Every method, sonic included, removes Vsh times the shale point's apparent porosity by the same transform. So the sonic methods need dt_sh.
- **Carbonates**: sonic porosity misses vugs and fractures. A difference between neutron-density and sonic porosity is secondary porosity. Report it as an interpretation.

## Calibration

If core porosity is supplied, calibrate with `petro_core_calibrate` (skill `petro-core-scal`): depth shift, overburden correction, then log-versus-core bias and scatter. Take the grain density as `rho_ma`, and rerun. Porosity is "calibrated to core" only when the bias is within about ±0.01 v/v. Without core, state that it is not calibrated.

## Reporting

- `measurements`: give each with `source_type` `derived` and the provenance id. For example: "Effective porosity (gas-corrected neutron-density, rho_ma 2.65 from core grain density) median 0.25 in SAND_B (10th-90th percentile 0.24-0.26)".
- `assumptions`: every matrix, fluid and shale point, with its source.

<!-- end of skill: petro-porosity -->
