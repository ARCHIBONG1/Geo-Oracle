---
name: petro-permeability
description: Permeability - core porosity-permeability transforms with 10th-90th percentile bands, flow units (RQI/FZI), Timur's estimate, extrapolation limits and the no-calibration rule. Load before petro_permeability.
---

# Permeability

## Rule

**No core, no transform.** Permeability from logs is always a transform calibrated on core, or on NMR, which is not yet available. The tool refuses `core_transform` without core permeability. Timur's estimate needs a sourced `swirr` and is flagged `UNCALIBRATED` without core.

## Core transform

`petro_permeability(method="core_transform", core=<calibrated core table>, well=<well with PHIE>)`:
- **The fit**: log10 k = intercept + slope x porosity on the calibrated plugs (always run `petro_core_calibrate` first).
- **Curves written**: `PERM` (50th percentile) with `PERM_P10` and `PERM_P90` (10th/90th percentiles, from the residual scatter). Report the band, not just the median: permeability uncertainty is large.
- **`fit.r2`, `sd_log10`**: sd_log10 of about 0.3 means a factor of about 2.6 between the 10th and 90th percentiles.
- **`extrapolated_fraction`**: where log porosity lies outside the core porosity range, the transform is extrapolated. Say so for those intervals.
- **One transform per rock type**: a sandstone transform does not apply to a limestone. Fit per zone or facies, by giving a core table restricted to it, when rock types differ, and state which transform applies where.

## Flow units

`flow_units: N` groups the core plugs by flow-zone indicator (FZI = RQI / phi_z, with RQI = 0.0314 sqrt(k/phi) and phi_z = phi/(1 - phi)). Units are ordered 1 (poorest) to N.
- **Report**: each unit's FZI and its median porosity and permeability.
- **Use**: units explain scatter in the poro-perm plot; mixing units is a common cause of a poor fit.
- **Limits**: mapping units onto the log needs a link (e.g. electrofacies or lithology). Propose it as a follow-up; the tool does not guess it.

## Timur

k = 0.136 phi^4.4 / Swirr^2 (percent units). Valid only above the transition zone, where the formation is at irreducible saturation.
- **Swirr**: from SCAL or capillary pressure (source `scal`), never assumed silently.
- **Without core**: report the result as an estimate (`proposed`), never `supported`.
- **With core**: the result includes `vs_core_log10` bias and RMS. Report them.

## Figure

`petro_render(kind="poro_perm", core=<calibrated core table>, well=...)` draws the plugs, the transform and its band, and the well's log porosity range, which shows where the logs extrapolate.

## Reporting

- **Measurements**: e.g. "Permeability (core transform on 118 calibrated plugs, R2 0.97) median 290 md in SAND_B (10th-90th percentile band 200-460 md)", with the provenance id.
- **Limitations**: extrapolated intervals, transform scope, and plug-scale versus log-scale support.
