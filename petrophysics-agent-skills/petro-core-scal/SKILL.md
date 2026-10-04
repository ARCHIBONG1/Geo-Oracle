---
name: petro-core-scal
description: Core calibration and SCAL - core-to-log depth shift, overburden correction, log-versus-core statistics, grain density as matrix density, and Archie a, m, n from formation factor and resistivity index. Load before petro_core_calibrate or petro_scal.
---

# Core calibration and SCAL

## Core calibration: `petro_core_calibrate`

Core depths are driller's depths; logs are logger's depths. They routinely differ by metres. Always depth-match before comparing.

1. **Register the core table** (`core_data`: md_m, porosity, optional permeability_md, grain_density_g_cc).
2. **Overburden factors**: if the core was measured at ambient conditions (usual for routine analysis), put `overburden_porosity_factor` and `overburden_perm_factor` in the parameter set, from SCAL at stress (`core`, `scal`) or the report. In situ value = ambient x factor.
   - Without them the tool uses core as measured, and says so: report it.
   - Typical ranges are about 0.95-0.99 for porosity and 0.5-0.95 for permeability, but use measured values.
3. **Compute a porosity log first, then run the tool against it.** The comparison curve must rise with core porosity: `PHIE` or `PHID` from `petro_porosity` (density, or density-neutron). Never compare core porosity with raw `NPHI` or `RHOB`: in a sand-shale succession neutron porosity is *higher* in shale and density is *lower* in sand, so both anti-correlate with core porosity and the tool refuses them (correlation at or below zero).
4. **Read the band, not the single value.** A correlation against depth is a ridge, not a spike, so the shift is only as sharp as the data make it. The tool returns `uncertainty_band_m` and `half_width_m`, the wider of two things it measures from this well: the ridge's own width, judged against the point-to-point noise of that profile, and the coarser of the two sample spacings, since neither series locates a feature more finely than it is sampled (`half_width_from` says which dominated). Report the shift as `<value> +/- <half width>` and carry the half width downstream as the depth uncertainty on every core-log comparison.
   - Whether a band is tight enough is a judgement about the work, not a property of the number: a half width of 0.5 m is immaterial against a 20 m sand and fatal against 0.3 m laminae. Say which case you are in.
   - A band is a result. It does not block downstream work; a documented nominal offset that falls inside it is consistent with the data, which is worth saying, and one that falls outside it is a contradiction worth reporting.
   - `tolerance` can be set when you know the data better than the profile's noise suggests; the default and its source are reported with every result.
5. **A rival is a rival only outside the band.** `rival` is reported when a separate maximum outside the band scores within the same tolerance of the best: a real alias (repeated beds of the same character) or a genuinely ambiguous match. Then both shifts are reported and `missing_data` asks for core-run tie markers. Lower maxima on the same ridge are noise, listed as `best_outside_band` for information only; never report them as competing shifts.
6. **Read the refusals.** A best shift at the search boundary means the correlation is still rising beyond the window: widen `max_shift_m` only if a shift that large is physically plausible (a misnumbered core run); otherwise the compared intervals are not the same rock, and core-run tie markers are needed. A weak correlation after shifting (below 0.3) is published with a warning: say it in `limitations`.
7. **Pass the shift on.** `petro_publish_table(table=..., shift=<the core_depth_shift product>)` publishes a driller-depth table (core data, dip picks) in log depth, keeping the original as `md_m_driller` and recording the band in the sidecar. That is what the sedimentology specialist needs; publishing the table unshifted leaves it blocked.

**Reading the result:**
- **`depth_shift_m`**: added to core depths. A shift at the edge of the search, or a `second_best` almost as good (warned), makes the match ambiguous: report both, and do not over-interpret thin-bed comparisons. Expect a precision of about ±0.1-0.2 m.
- **`log_vs_core_after_shift`**: the bias (log minus core) and the RMS.
  - A bias beyond about ±0.01 v/v points at matrix density, fluid density, shale correction or overburden.
  - An RMS near the plug-scale heterogeneity (about 0.01-0.02) is normal: plugs sample centimetres, logs decimetres.
- **`grain_density_p10_p50_p90`**: per zone. Its `register_as.rho_ma` (source `core`) is the preferred matrix density: rerun porosity with it.
- **`outputs.table`**: the calibrated core table (shifted, overburden-corrected). Use it for permeability and every later comparison, never the raw table.

## SCAL: `petro_scal`

- **`formation_factor`**: fits FF = a / phi^m.
  - Free a and m: report both.
  - `fix_a: 1`: fits m alone. It is conventional when the data do not constrain a.
- **`resistivity_index`**: fits RI = Sw^-n through (1, 1).

Register the returned `register_as` values (source `scal`) and add `low`/`high` from the fit scatter or plug-to-plug spread. m and n usually dominate saturation uncertainty. A single plug is not a SCAL program: flag it.

## Reporting

- **Measurements**: the depth shift, correlation before and after, bias and RMS after calibration, grain density per zone, and SCAL a/m/n with sample counts and residuals.
- **Claims**: a porosity model is "calibrated to core" (`supported`) only when the bias is within about ±0.01 v/v after calibration.
