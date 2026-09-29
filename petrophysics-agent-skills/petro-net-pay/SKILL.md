---
name: petro-net-pay
description: Net reservoir, net pay and zone summaries - sourcing cutoffs, running the integrated evaluation with bad-hole exclusion, reading the seeded uncertainty percentiles and the sensitivity ranking, and reporting zone summaries for prospect, play and risk work. Load before petro_net_pay.
---

# Net pay and zone summaries

## The tool

`petro_net_pay(well, tops, parameter_set, vshale_method, porosity_method, saturation_model, zones, exclude_bad_hole=True, uncertainty_runs=300)`:
- **What it computes**: it recomputes shale volume, porosity and saturation from the parameter set (the same equations as the single-property tools), then applies the cutoffs zone by zone.
- **Background job**: with uncertainty runs it runs in the background. If it returns `running`, collect it with `get_petro_job_result`.
- **The product**: it saves a `zone_summary` file (`outputs.zone_summary`), the product other specialists consume.

## Before running

1. **Methods chosen and justified** with the single-property tools (skills `petro-shale-volume`, `petro-porosity`, `petro-saturation`), on the same zones.
2. **One parameter set** holding every parameter those methods need, plus the three cutoffs:
   - `vsh_cutoff`, `phi_cutoff` and `sw_cutoff` come from core or test data (`core`, `test`), or the task (`task`), or are declared provisional (`assumption` with a range).
   - Never use an unstated "standard" cutoff: the cutoffs often move net pay more than any other choice.
3. **Ranges (`low`, `high`)** on every uncertain parameter (a, m, n, Rw, matrix density, endpoints, temperature gradient, cutoffs), so the uncertainty covers them.
4. **A trajectory**, so thicknesses are also true vertical (TVT), not only along-hole.

## What it reports per zone

| Field | Meaning |
|---|---|
| `gross_md_m` / `_tvt_m` | Zone thickness |
| `net_reservoir_*` | VSH at or below its cutoff and PHIE at or above its cutoff |
| `net_pay_*` | Net reservoir with SW at or below its cutoff |
| `net_to_gross` | Net reservoir / gross (MD) |
| `excluded_md_m` | Bad hole (washout, high density correction) or missing data: counted in gross, never in net |
| `reservoir_*_avg`, `pay_*_avg` | Thickness-weighted porosity and shale volume; pore-volume-weighted Sw |
| `hydrocarbon_column_m` | Sum of PHIE (1 - SW) over pay (TVT when available): the hydrocarbon thickness |

## Uncertainty

- **`percentiles`**: the 10th, 50th and 90th percentiles of each metric over the seeded runs. Each ranged parameter is drawn from a triangular distribution (low, value, high), independently. The runs are reproducible: the same call gives the same numbers.
- **`drivers_by_hydrocarbon_column`**: parameters ranked by how far their own low-to-high range moves the hydrocarbon column, others held at their values. The first one or two are what to measure next: e.g. m and n point to SCAL, and Rw to a water sample. Put that in `recommended_followups`.
- **Reporting**: say "10th percentile", "50th percentile" and "90th percentile", never bare P10/P90 (the industries use them in opposite senses). The base case (central values) and the 50th percentile differ when ranges are asymmetric; report both.
- **What it does not capture**: uncertainty from model choice (e.g. Archie against Simandoux) or depth errors. Run the alternative model separately, and state the difference.

## Checks before reporting

- **Unexpected pay near a contact or in a washout**: check `petro_qc_logs`. Pay inside excluded intervals is never counted, but pay just outside them deserves a look.
- **Pay depends on a cutoff**: if a zone's net pay changes a lot between the cutoff's low and high (see the sensitivity), say the result is cutoff-sensitive.
- **Fluid contacts**: a contact read from where pay ends is an `interpretation`. It is supported only by pressures (gradient intersection) or tests, or by a saturation change that coincides with a density-neutron and resistivity change, and even then only `partially_supported`.

## Reporting

- **Measurements**: one per zone, e.g. "SAND_B: net pay 21.9 m TVT (22.0 m MD), average porosity 0.25 and Sw 0.25 (Simandoux), hydrocarbon column 4.0 m (10th-90th percentile 3.9-4.1 m over 200 seeded runs)", with the provenance id.
- **Cutoffs**: in `assumptions`, with their sources.
- **Conclusions**: `Product: zone_summary — <path>` so Geo Oracle can forward it to prospect, play or risk work.
- **Downstream consumers need ranges, not single numbers**: give the percentiles.
