---
name: prospect-volumetrics
description: Quantities in place for every domain. It covers gross rock volume from a closure's cells, the sourced inputs and their refusals, sampling with a stated shape, and a correlation only with its source. It also covers the tornado sensitivity, the percentile convention, and what each quantity is not. Load before any volume.
---

# Volumetrics

## The order

1. `pt_resolve_inputs`: every input that the domain needs, each as `{product, field, value}`, `{range: [p10, p50, p90], source}` or `{samples, source}`. It refuses a bare number. It refuses a missing input, with the owner to ask. `pt_readiness` reads the file that `pt_resolve_inputs` writes, for R4.
2. `pt_readiness` with that file: a prospect or a target may carry a quantity, and nothing lower may.
3. `pt_grv` per scenario: the cell area times the summed column above the contact, exact on the grid.
   - The base of the reservoir is at the top plus its gross vertical thickness (`reservoir_thickness_m`, sourced: an isochore from stratigraphy or the wells). So a base above the contact truncates the volume (Samson et al. 1996).
   - Without a thickness, the tool assumes that the base lies below the contact everywhere, and says so. Give the thickness whenever the column can exceed the reservoir.
   - The spread is a GRV per realised surface from the structural specialist's realisations, at the same contact and thickness. It is never fitted and never scaled by area.
   - A closure published before the realised cells were carried gives the base case only. The request is then to republish it.
   - The tool draws the closure map at the contact in `result.figures`.
4. `pt_quantity` per scenario: the domain's quantity from the GRV file and the inputs file, by seeded sampling. It draws the exceedance curve of `volume_ranges` in `result.figures`.
5. `pt_sensitivity`: the tornado swing. It names the input whose owner's next product narrows the range most. It writes the sensitivity file and draws the tornado in `result.figures`.
6. Carry every figure in `result.figures` into the findings, as the `prospect-visualisation` skill says.

## The quantities

| Domain | Formula | Unit | What it is not |
|---|---|---|---|
| hydrocarbon | GRV x NTG x phi x (1 - Sw) / FVF | million m3 in place | recoverable |
| co2_storage | GRV x NTG x phi_e x rho_CO2 x E_v E_d | Mt static capacity | injectable: the pressure limit and injectivity are element statuses, not multipliers |
| natural_hydrogen | GRV x NTG x phi x (1 - Sw) / Bg | million m3 free hydrogen now | a renewable rate, or a persistent stock unless preservation is demonstrated; the generation and preservation statuses travel with it (the elements file is required) |
| hydrogen_storage | GRV x NTG x phi x (1 - Sw) x rho_H2 | kt total capacity; working gas only with a sourced cushion-gas fraction | deliverability, which is an element status |
| geothermal | V x rho c x (T_reservoir - T_reference) | PJ heat in place; the volume given as a sourced input | producible energy |

Densities, formation volume factors, the CO2 displacement efficiency, the cushion fraction, the heat capacity and the reference temperature are inputs like any other. They come from the PVT table, a petrophysics product, or the literature specialist with a citation. None has a default.

The CO2 input is `displacement_efficiency`: the displacement terms E_v E_d alone (Goodman et al. 2011, Table 9: P10-P90 7.4-24 % for clastics, 10-21 % for limestone, 16-26 % for dolomite), with the porosity effective. Never give their whole saline factor E_saline (Table 8, about 0.4-5.5 %). It already holds the net-to-gross and porosity terms, which enter here separately, so it would count them twice. A value below 7.4 % draws a warning, and the tool refuses a percentage.

## Sampling, stated

- Shape: `lognormal` through p10 and p90 by default (the usual volumetric convention), or `quantile_linear`. Give the shape with the result. A realization set is sampled as it is. A single value is a point, and the result says so.
- If a stated p50 sits more than 5 % from the lognormal median through p10 and p90, the tool warns: the shape may not fit that input. Put the warning in `limitations`.
- A correlation between inputs is applied only with its source. The result is then reported with and without it. An assumed correlation is an invented input.
- The seed and the sample count are recorded. The record answers an identical call.

## Percentiles

Non-exceedance, always labelled: p10 is the value 10 % of cases fall below. Report a quantity as `<candidate>, <scenario>: <quantity> p10 / p50 / p90 <unit> (non-exceedance)`.

## Sensitivity

The tornado centre is the formula at every p50. Each input swings between its p10 and p90, with the others at p50. The tornado is deterministic, and the ranking is the maturation advice: the input with the largest swing is the one to ask its owner about first.

## Reporting

A quantity is a `measurement` with `source_type` derived and the tool's `prov:` id. The `volume_ranges` product carries the input table with every source, the seed, the shape and the caveat for the domain.

<!-- end of skill: prospect-volumetrics -->
