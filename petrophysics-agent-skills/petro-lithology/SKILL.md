---
name: petro-lithology
description: Lithology and mineralogy from logs - matrix identification (quartz/calcite/dolomite), multimineral solves with a reconstruction residual, and electrofacies; their validity limits and how to name and report the results. Load before petro_lithology.
---

# Lithology and mineralogy

## The tool

`petro_lithology(well, method, parameter_set, minerals, curves, facies, tops)` writes curves into a new well file. Add `tops` for per-zone composition.

| Method | Output | Valid when | Needs |
|---|---|---|---|
| `matrix_id` | RHOMAA, UMAA, F_QUARTZ / F_CALCITE / F_DOLOMITE, LITH (1 quartz, 2 calcite, 3 dolomite) | Clean, liquid-filled rock | RHOB, NPHI, PEF; `rho_fl` |
| `multimineral` | V_<MINERAL>, PHI_MM, MM_RESID | Minerals present are known; hole in gauge | RHOB, NPHI, PEF (+ DT); `rho_fl`; shale point (`rho_sh`, `nphi_sh`, `pef_sh`, + `dt_sh`, `dt_fl` if DT is used) |
| `electrofacies` | FACIES (1 = lowest gamma ray) | Any interval; descriptive only | The chosen curves (default GR, RHOB, NPHI, RT) |

## Choosing and checking

- **Matrix identification**: shale and gas distort the apparent matrix values. Use it in clean, water-bearing intervals, and treat results in shaly or gas-bearing beds as unreliable.
- **Multimineral**:
  - **Unknowns**: at most one more than the number of logs (the unity constraint supplies the extra equation). With RHOB, NPHI and PEF, that is three minerals plus porosity.
  - **Mineral list**: choose it from the geology (the task, cuttings, core, regional context), never by trying lists until the residual is small.
  - **`MM_RESID`** is the misfit in units of tool precision. A median below about 1 means the model explains the logs. Where it is high, the model does not fit: another mineral, hydrocarbons in the flushed zone (the fluid point assumes water), or bad hole. Report those intervals.
  - **Reference values**: mineral properties are standard chart values, listed in the result; state them. The shale point is a parameter with a source.
- **Electrofacies**:
  - **Clusters**: groups of similar log response, from seeded k-means keeping the best of 10 starts, so results are reproducible. The number of facies is your choice: try 2-3 values and report the one whose facies are geologically distinct and stable.
  - **Bad hole forms its own facies**: a washout is a distinct log signature. Check facies against `petro_qc_logs` before naming them.
  - **Naming**: name facies by their log character (e.g. "low GR, high resistivity, crossover") and by the lithology tools. They are not depositional facies: those belong to sedimentology, and core or cuttings calibrate them. Say so, and recommend it.

## Reporting

- **Measurements**: mineral volumes and porosity per zone (`derived`), with the mineral list, the endpoints and the median residual.
- **Interpretations**: lithology names, and what the facies represent, with alternatives.
- **Limitations**: minerals assumed absent, water-filled flushed zone assumed, bad-hole intervals.
