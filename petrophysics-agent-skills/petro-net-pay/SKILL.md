---
name: petro-net-pay
description: Net reservoir, net pay and zone summaries. It covers the cutoff sources, the integrated evaluation with bad-hole exclusion, the seeded uncertainty percentiles and the sensitivity ranking. It also covers zone summaries for prospect, play and risk work. Load before petro_net_pay.
---

# Net pay and zone summaries

## The tool

`petro_net_pay(well, tops, parameter_set, vshale_method, porosity_method, saturation_model, zones, exclude_bad_hole=True, uncertainty_runs=300)`:
- **What it computes**: it recomputes shale volume, porosity and saturation from the parameter set, with the same equations as the single-property tools. Then it applies the cutoffs zone by zone.
- **Background job**: if the call has uncertainty runs, the tool runs in the background. If it returns `running`, collect the result with `get_petro_job_result`.
- **The product**: it saves a `zone_summary` file (`outputs.zone_summary`). Other specialists use this product.
- **Figure**: it draws the composite log of the curves it summarised (VSH, PHIE, BVW, with the zone tops) into `result.figures`.

## Before running

1. **Methods chosen and justified** with the single-property tools (skills `petro-shale-volume`, `petro-porosity`, `petro-saturation`), on the same zones.
2. **One parameter set** that holds every parameter those methods need, plus the three cutoffs:
   - `vsh_cutoff`, `phi_cutoff` and `sw_cutoff` come from core or test data (`core`, `test`) or from the task (`task`). Or declare them provisional (`assumption` with a range).
   - Never use an unstated "standard" cutoff. The cutoffs often move net pay more than any other choice.
3. **Ranges (`low`, `high`)** on every uncertain parameter (a, m, n, Rw, matrix density, endpoints, temperature gradient, cutoffs). The uncertainty then covers them.
4. **A trajectory**, so that the thicknesses are also true vertical (TVT), not only along-hole.

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

- **`percentiles`**: the 10th, 50th and 90th percentiles of each metric over the seeded runs. The tool draws each ranged parameter independently from a triangular distribution (low, value, high). The runs are reproducible: the same call gives the same numbers.
- **`drivers_by_hydrocarbon_column`**: the parameters, ranked by how far each one moves the hydrocarbon column over its own low-to-high range. The other parameters stay at their values. The first one or two are the parameters to measure next. For example, m and n point to SCAL, and Rw points to a water sample. Put that in `recommended_followups`.
- **Reporting**: say "10th percentile", "50th percentile" and "90th percentile". Never say bare P10/P90, because the industries use them in opposite senses. If the ranges are asymmetric, the base case (central values) and the 50th percentile differ. Report both.
- **What it does not capture**: uncertainty from model choice (for example, Archie against Simandoux) or from depth errors. Run the alternative model separately, and state the difference.

## Checks before reporting

- **Unexpected pay near a contact or in a washout**: check `petro_qc_logs`. The tool never counts pay inside excluded intervals. But look at pay just outside them.
- **Pay depends on a cutoff**: check the sensitivity for each cutoff. If a zone's net pay changes a lot between the cutoff's low and high, say that the result is cutoff-sensitive.
- **Fluid contacts**: a contact read from where pay ends is an `interpretation`. Three kinds of evidence support it: pressures (gradient intersection), tests, or a saturation change that coincides with a density-neutron and resistivity change. With the saturation change alone, it is only `partially_supported`.

## Reporting

- **Measurements**: give one per zone, with the provenance id. For example: "SAND_B: net pay 21.9 m TVT (22.0 m MD), average porosity 0.25 and Sw 0.25 (Simandoux), hydrocarbon column 4.0 m (10th-90th percentile 3.9-4.1 m over 200 seeded runs)".
- **Cutoffs**: put them in `assumptions`, with their sources.
- **Conclusions**: give `Product: zone_summary — <path>`, so that Geo Oracle can forward it to prospect, play or risk work.
- **Downstream consumers need ranges, not single numbers**: give the percentiles.

<!-- end of skill: petro-net-pay -->
