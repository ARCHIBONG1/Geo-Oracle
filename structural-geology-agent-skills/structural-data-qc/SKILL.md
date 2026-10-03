---
name: structural-data-qc
description: Reading upstream products and their sidecars (domain, pick uncertainty, scope, validity, provenance), the time-versus-depth rule, supplied structural tables and their conventions. Load before reading any product or table.
---

# Structural data QC

## Products you receive

| Product | From | Sidecar facts to read | Use |
|---|---|---|---|
| `fault_set` | seismic_interpretation (`seismic_publish_structure`) | domain; `pick_uncertainty`; `depth_conversion`; each fault's dip, dip azimuth, strike, length, z range, confidence | Orientations, stability, Andersonian check; a time-domain set without a time-depth table has no dips: ask for a depth version |
| `horizon` | seismic_interpretation | domain, z unit, grid decimation, `pick_uncertainty` | Surface attributes (depth only); closures (depth only, phase 1) |
| `vshale_log` | wells_petrophysics (`petro_vshale`) | method, endpoints, depth columns (md_m, tvdss_m) | Fault seal; use `tvdss_m` unless the well is vertical |
| `pressure_profile`, `well_header` | wells_petrophysics | reference datum, method | Pp for the stress tensor; well positions |
| `stress_field` | regional_geology | scope (regional), `resolution` (distance to the nearest indicator), the transfer template, `shmax.mean_azimuth_deg`, `circular_sd_deg`, `regime_counts` | SHmax azimuth and regime; the spread sets the azimuth range; the transfer status sets the ceiling |
| `structural_elements` | regional_geology | scope, dataset | Expected regional structural grain, not mapped faults in the study area |

`sg_read_product` returns the sidecar and a preview. Read it before every use; the domain and the uncertainty decide what you may conclude. A product with no stated domain is flagged and cannot feed closures or stress analyses.

## Time and depth

- Dips, closures, spill points, stresses and seal need depth. A time-domain horizon or fault set is refused by those tools; report the refusal and request `seismic_interpretation: depth versions of H1 and the fault set — seismic_time_to_depth then seismic_publish_structure — closures and stability need depth`.
- Apparent geometry in time is still evidence of relationships (which fault offsets which horizon) and may be stated as such, labelled time-domain.

## Supplied tables (`sg_register_table`)

- `structural_measurements`: `dip_deg`, `dip_azimuth_deg` (the direction of dip, degrees from north; strike + 90), optional type, x, y, z_m, location, source. Column aliases (dip, dipdir, azimuth) are recognised; otherwise give a `columns` mapping.
- `fracture_picks`: `depth_m`, `dip_deg`, `dip_azimuth_deg`, optional set, aperture, type, well.
- `section_line`: x, y points.
- Record in `description` what the measurements are and how they were taken; outcrop, core and image-log measurements carry different biases, and sets from one domain are not averaged with another's.

## Pick uncertainty

The seismic sidecar states it (one sample interval by default). Throw and closure ranges (phase 1) are run within it; a conclusion finer than the uncertainty is not supported.
