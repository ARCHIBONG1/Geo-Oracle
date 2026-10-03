---
name: strat-data-qc
description: Reading correlation_logs and well_tops products and their sidecars, the depth rule (TVDSS, deviated wells), registering biostratigraphic events, dated samples, operator tops, markers and chemostratigraphy, and the sample-type rules for biostratigraphy. Load before reading any product or table.
---

# Stratigraphic data QC

## Products you receive

| Product | From | Sidecar facts to read | Use |
|---|---|---|---|
| `correlation_logs` | wells_petrophysics (`petro_publish_correlation_logs`) | well, curves and units, step, `deviated`, trajectory basis, datum and elevation, MD and TVDSS ranges, QC note | All correlation; depth conversion for every table |
| `well_tops` | wells_petrophysics | `tag` reported, `source`, count | The reported ledger |
| `horizon` and well ties | seismic_interpretation | domain, pick uncertainty | Rank-3 ties (phase 1) |
| `chronostrat_chart` | regional_geology | time scale, sources, gaps | C7 (phase 1) |
| `fault_network` | structural_geology | abutments, throws | Gap versus fault (phase 1) |

`stg_read_product` returns the sidecar and a preview. A well file (LAS) listed by `stg_list_data` cannot be opened here: ask `wells_petrophysics: correlation_logs and well_tops for <well> — petro_publish_correlation_logs — correlation`.

## Depth

- Correlate in TVDSS. A deviated well without a trajectory-based TVDSS (sidecar `trajectory` "vertical (datum offset only)" with `deviated` true) needs `wells_petrophysics` to run `petro_trajectory` first; say so.
- `stg_register_table` converts every table's MD to TVDSS with the wells' logs (`logs=`). A row for a well without logs stays MD-only and is flagged.
- Report picks with both MD and TVDSS; the products carry both.

## Tables (`stg_register_table`)

| Kind | Columns | Rules |
|---|---|---|
| `biostrat_events` | well, md_m, taxon, event (FO, LO, ACME, FCO, LCO), sample_type (cuttings, core, sidewall, outcrop), confidence | A first occurrence from cuttings is never a datum (caving); it is kept as rank-5 support and flagged. A last occurrence above a younger datum suggests reworking: report it |
| `age_dates` | well, md_m, method, age_ma, error_ma, time_scale | Rank 1; the time scale must be stated |
| `reported_tops` | well, top, md_m, source | Reported ledger (a report's tops when no well_tops product exists) |
| `marker_table` | well, md_m, marker | Rank 4 |
| `chemostrat_data` | well, md_m, variable, value | Phase 2 |

Describe in `description` who produced the data and how; it travels into the products.

## Reporting

Reported tops: `[data, upstream_specialist or project_data]` with the product reference. Events and markers: `[observation, project_data]`. Ages: `[measurement, project_data]`.
