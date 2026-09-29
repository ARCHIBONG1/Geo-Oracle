---
name: petro-shale-volume
description: Shale volume (VSH) - choosing gamma-ray, neutron-density or SP methods, sourcing the endpoints, comparing at least two methods, and the traps (radioactive sands, washouts, SP limits). Load before petro_vshale or petro_net_pay.
---

# Shale volume

## The tool

`petro_vshale(well, methods=[...], parameter_set, endpoints, md_range, tops)`:
- **Methods**: the first method is written as `VSH` into a new well file; the others are compared with it (median and largest difference).
- **Endpoints**: `endpoints: "percentiles"` picks the gamma-ray endpoints as the 5th and 95th percentiles over `md_range`.
- **Per-zone statistics**: add `tops` to get the 10th, 50th and 90th percentiles per zone.

## Methods

| Method | Formula (IGR = (GR - GR_clean) / (GR_shale - GR_clean)) | Use when |
|---|---|---|
| gr_linear | VSH = IGR | Default upper bound; always compute it |
| larionov_tertiary | 0.083 (2^(3.7 IGR) - 1) | Young, unconsolidated rocks (Tertiary) |
| larionov_older | 0.33 (2^(2 IGR) - 1) | Older, consolidated rocks |
| clavier / stieber | Non-linear corrections | Alternatives; state why chosen |
| neutron_density | (phiN - phiD) / (phiN_sh - phiD_sh) | Liquid-filled, known lithology; independent of gamma-ray radioactivity |
| sp | 1 - PSP / SSP | Water-based mud and good SP only; water-bearing beds |

The non-linear methods read below `gr_linear`. The choice changes the net sand, so the choice needs a reason: the rock's age or consolidation, or agreement with core or the neutron-density method.

## Endpoints

These are parameters, so each needs a source:
- **`gr_clean` and `gr_shale`**: from the task (`task`), from core-calibrated clean sand and shale (`core`), or from the log itself (`log_percentile`: run with `endpoints: "percentiles"` over a stated interval, then register the reported values with that source and the tool's provenance id as reference).
- **Neutron-density endpoints**: `rho_ma`, `rho_fl`, `rho_sh`, `nphi_ma` and `nphi_sh`. Shale points (`rho_sh`, `nphi_sh`) come from a thick, in-gauge shale (`log_crossplot`); matrix points from core grain density or the known lithology.
- **Scope**: endpoints are local. Use one set per zone or formation when the shale character changes, and state it.

## Always compare two methods

Run at least a gamma-ray method and neutron-density (or SP) together, as `methods: ["gr_linear", "neutron_density"]`:
- **Agreement within about 0.05 v/v** supports the choice.
- **Disagreement is evidence, not noise:**
  - **Gamma ray high, neutron-density low:** radioactive minerals (K-feldspar, mica, uranium, heavy minerals) in a clean reservoir. The gamma ray overstates shale. Say so, and prefer the neutron-density method if the lithology is known.
  - **Neutron-density high, gamma ray low:** gas (it pulls the neutron down and the density porosity up, shrinking the separation, so the neutron-density shale volume is too low), or non-radioactive clays such as kaolinite.
  - **Inside washouts**, the density and neutron are unreliable: use the gamma-ray method there.

## Reporting

- `measurements`: "VSH (gamma-ray linear, GR 20-120 API from the task) median 0.05 in SAND_B (10th-90th percentile 0.02-0.08)", with `source_type` `derived` and the provenance id.
- `assumptions`: the endpoints with their sources.
- `interpretations`: the reason for the chosen method, and what any disagreement between methods means.
