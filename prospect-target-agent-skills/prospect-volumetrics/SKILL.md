---
name: prospect-volumetrics
description: Quantities in place for every domain - gross rock volume from a closure's cells, the sourced inputs and their refusals, sampling with a stated shape, a correlation only with its source, the tornado sensitivity, the percentile convention, and what each quantity is not. Load before any volume.
---

# Volumetrics

## The order

1. `pt_resolve_inputs`: every input the domain needs, each as `{product, field, value}`, `{range: [p10, p50, p90], source}` or `{samples, source}`. A bare number is refused; a missing input is refused with the owner to ask. The file it writes is what `pt_readiness` reads for R4.
2. `pt_readiness` with that file: a prospect or a target may carry a quantity, nothing lower may.
3. `pt_grv` per scenario: the cell area times the summed column above the contact, exact on the grid, with the base of the reservoir at the top plus its gross vertical thickness (`reservoir_thickness_m`, sourced: an isochore from stratigraphy or the wells), so a base above the contact truncates the volume (Samson et al. 1996). Without a thickness the tool assumes the base lies below the contact everywhere and says so: give it whenever the column can exceed the reservoir. The spread is a GRV per realised surface from the structural specialist's realisations, at the same contact and thickness, never fitted and never scaled by area; a closure published before the realised cells were carried gives the base case only, and the request is to republish it.
4. `pt_quantity` per scenario: the domain's quantity from the GRV file and the inputs file, by seeded sampling.
5. `pt_sensitivity`: the tornado swing, which names the input whose owner's next product narrows the range most.

## The quantities

| Domain | Formula | Unit | What it is not |
|---|---|---|---|
| hydrocarbon | GRV x NTG x phi x (1 - Sw) / FVF | million m3 in place | recoverable |
| co2_storage | GRV x NTG x phi_e x rho_CO2 x E_v E_d | Mt static capacity | injectable: the pressure limit and injectivity are element statuses, not multipliers |
| natural_hydrogen | GRV x NTG x phi x (1 - Sw) / Bg | million m3 free hydrogen now | a renewable rate, or a persistent stock unless preservation is demonstrated; the generation and preservation statuses travel with it (the elements file is required) |
| hydrogen_storage | GRV x NTG x phi x (1 - Sw) x rho_H2 | kt total capacity; working gas only with a sourced cushion-gas fraction | deliverability, which is an element status |
| geothermal | V x rho c x (T_reservoir - T_reference) | PJ heat in place; the volume given as a sourced input | producible energy |

Densities, formation volume factors, the CO2 displacement efficiency, the cushion fraction, the heat capacity and the reference temperature are inputs like any other: from the PVT table, a petrophysics product, or the literature specialist with a citation. None has a default.

The CO2 input is `displacement_efficiency`, the displacement terms E_v E_d alone (Goodman et al. 2011, Table 9: P10-P90 7.4-24 % for clastics, 10-21 % for limestone, 16-26 % for dolomite), with the porosity effective. Never give their whole saline factor E_saline (Table 8, about 0.4-5.5 %): it already holds the net-to-gross and porosity terms, which enter here separately, so it would count them twice. A value below 7.4 % draws a warning; a percentage is refused.

## Sampling, stated

- Shape: `lognormal` through p10 and p90 by default (the usual volumetric convention), or `quantile_linear`; the shape is reported with the result. A realization set is sampled as it is. A single value is a point and the result says so.
- The tool warns when a stated p50 sits more than 5 % from the lognormal median through p10 and p90: the shape may not fit that input, and the warning goes in `limitations`.
- A correlation between inputs is applied only with its source, and the result is then reported with and without it. An assumed correlation is an invented input.
- Seed and sample count are recorded; an identical call is answered from the record.

## Percentiles

Non-exceedance, always labelled: p10 is the value 10 % of cases fall below. Report a quantity as `<candidate>, <scenario>: <quantity> p10 / p50 / p90 <unit> (non-exceedance)`.

## Sensitivity

The tornado centre is the formula at every p50; each input swings between its p10 and p90 with the others at p50. Deterministic, and the ranking is the maturation advice: the input with the largest swing is the one to ask its owner about first.

## Reporting

A quantity is a `measurement` with `source_type` derived and the tool's `prov:` id; the `volume_ranges` product carries the input table with every source, the seed, the shape and the caveat for the domain.
