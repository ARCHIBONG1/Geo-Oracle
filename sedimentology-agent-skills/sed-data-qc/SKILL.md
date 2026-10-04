---
name: sed-data-qc
description: Registering core and cuttings descriptions, analyses, analogue tables and photo indexes; the depth rule (core depth shift before comparison with curves); reading petrophysics, stratigraphy, regional and seismic products. Load before reading any product or registering any table.
---

# Sedimentological data QC

## Products you receive

| Product | From | Sidecar facts to read | Use |
|---|---|---|---|
| `correlation_logs` | wells_petrophysics | curves, deviation, depth ranges | Curves beside the graphic log; electrofacies (phase 1) |
| `core_depth_shift` | wells_petrophysics (`petro_core_calibrate`) | shift_m, correlation, method | `sed_apply_depth_shift` on every driller's-depth table of that well |
| `dip_picks` | wells_petrophysics (`petro_publish_table`) | type, quality, structural dip removed or not | Palaeocurrents (phase 1) |
| `stratigraphic_framework`, `correlated_tops` | stratigraphy | admitted panel, surfaces, ledger | Key surfaces for the Walther exclusion; S4 |
| `regional_framework` | regional_geology | setting, transfer | S4 |
| horizon extractions | seismic_interpretation | domain, attribute | Geometry (phase 2) |

A well file or seismic volume listed by `sed_list_data` cannot be opened here: ask the owner for the product.

## Depth

- Core and photograph depths are driller's depths unless the table says otherwise. Before any comparison with curves, picks or the framework's surfaces, apply the well's `core_depth_shift` (`sed_apply_depth_shift`); without one, ask `wells_petrophysics: core depth shift for <well> — petro_core_calibrate on the core porosity — compare core with curves` and compare nothing.
- Register each table with its `depth_reference` (driller, log, unknown); the tools carry it through to the products and figures.

## Tables (`sed_register_table`)

| Kind | Columns | Notes |
|---|---|---|
| `core_description` | well, top_m, base_m, lithology; grain_size, sorting, structures, contacts, bioturbation_index (0-6), colour, fossils, notes | The describer's words are kept; the vocabulary is standardised afterwards |
| `cuttings_description` | well, top_m, base_m, lithology; grain_size, percent, notes | Cuttings mix intervals: structures are unreliable, lithology proportions usable |
| `grain_size_data` | well, depth_m, phi5 ... phi95 | Folk & Ward (phase 1) |
| `point_count_data`, `xrd_data` | well, depth_m, quartz/feldspar/lithics or mineral/percent | Composition (phase 1) |
| `analogue_table` | system, body_type, thickness_m, width_m, length_m, source, n | Always `[analogue]` |
| `photo_index` | file, well, top_m, base_m, lighting, scale_bar_mm | Images without a scale cannot support size statements (phase 1) |

Describe in `description` who described or analysed the rock and how; it travels into the products.

## Reporting

Descriptions as `[observation, project_data]` with the table's provenance id; analyses as `[measurement, project_data]`; products read as `[data, upstream_specialist]`.
