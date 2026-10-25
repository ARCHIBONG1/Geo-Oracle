---
name: sed-data-qc
description: How to register core and cuttings descriptions, analyses, analogue tables and photo indexes. It also covers the depth rule (core depth shift before comparison with curves), and how to read petrophysics, stratigraphy, regional and seismic products. Load before reading any product or registering any table.
---

# Sedimentological data QC

## Products you receive

| Product | From | Sidecar facts to read | Use |
|---|---|---|---|
| `correlation_logs` | wells_petrophysics | curves, deviation, depth ranges | Curves beside the graphic log (`sed_render`); no electrofacies tool yet |
| `core_depth_shift` | wells_petrophysics (`petro_core_calibrate`) | shift_m, correlation, method | `sed_apply_depth_shift` on every driller's-depth table of that well |
| `dip_picks` | wells_petrophysics (`petro_publish_table`) | type, quality, structural dip removed or not | No sedimentology tool reads it yet (no palaeocurrent tool): cite it, do not compute from it |
| `stratigraphic_framework`, `correlated_tops` | stratigraphy | admitted panel, surfaces, ledger | Key surfaces for the Walther exclusion (`sed_transitions_all(key_surfaces_product=...)` reads the correlated_tops picks); S4 |
| `regional_framework` | regional_geology | setting, transfer | S4 |
| horizon extractions | seismic_interpretation | domain, attribute | No tool yet |

Read every product's sidecar with `sed_read_product` before you use it. The depth reference, the shift, the domain and the provenance come from there, not from the file name. If `sed_list_data` lists a well file or seismic volume, you cannot open it here. Ask the owner for the product.

## Depth

- Core and photograph depths are driller's depths, unless the table says otherwise.
- Before any comparison with curves, picks or the framework's surfaces, apply the well's `core_depth_shift` (`sed_apply_depth_shift`).
- If the well has no shift, ask `wells_petrophysics: core depth shift for <well> — petro_core_calibrate on the core porosity — compare core with curves`, and compare nothing.
- Register each table with its `depth_reference` (driller, log, unknown). The tools carry it through to the products and figures.

## Tables (`sed_register_table`)

| Kind | Columns | Notes |
|---|---|---|
| `core_description` | well, top_m, base_m, lithology; grain_size, sorting, structures, contacts, bioturbation_index (0-6), colour, fossils, notes | The describer's words are kept; the vocabulary is standardised afterwards |
| `cuttings_description` | well, top_m, base_m, lithology; grain_size, percent, notes | Cuttings mix intervals: structures are unreliable, lithology proportions usable |
| `grain_size_data` | well, depth_m, phi5 ... phi95 | Registered only; no statistics tool yet |
| `point_count_data`, `xrd_data` | well, depth_m, quartz/feldspar/lithics or mineral/percent | Registered only; no composition tool yet |
| `analogue_table` | system, body_type, thickness_m, width_m, length_m, source, n | Always `[analogue]` |
| `photo_index` | file, well, top_m, base_m, lighting, scale_bar_mm | Images without a scale cannot support size statements; `sed_render(kind='core_photos')` draws them in depth order (shift to log depth to draw them beside the graphic log) |

In `description`, say who described or analysed the rock, and how. It travels into the products.

## Reporting

- Descriptions: `[observation, project_data]`, with the table's provenance id.
- Analyses: `[measurement, project_data]`.
- Products read: `[data, upstream_specialist]`.

<!-- end of skill: sed-data-qc -->
