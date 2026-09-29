---
name: petro-core
description: Core procedure for every petrophysics task - reading the SpecialistTask and its analysis mode, domain emphasis (hydrocarbon, CO2 storage, geothermal), writing evidence items and claims with provenance and depth references, naming alternatives for log features, missing_data format, and the final self-check. Load at the start of every task.
---

# Petrophysics core procedure

## 1. Read the task

Every field is used.

| Field | What to do with it |
|---|---|
| `objective`, `geological_question` | Restate as petrophysical sub-questions: which wells, which zones, which properties |
| `required_outputs` | Your checklist. Each item is either produced, or named in `limitations` with the reason |
| `available_evidence` | Files (paths or uploads) and small tables (CSV in `excerpt`). The excerpt carries facts the file may lack: datum, units, mud, temperatures |
| `upstream_findings` | Other specialists' results, e.g. tops from stratigraphy. Use them at their stated classification and cite them by full id; never restate them as your own |
| `known_constraints` | Honour them exactly, e.g. "use Rw = 0.04 ohm.m at 90 degC from the water sample" becomes a parameter with source `task` |
| `competing_interpretations` | Test each one; report which data support or contradict it |
| `downstream_use` | Sets precision and format: averages for prospect work, depth-indexed tables for seismic |
| `project_context` | Sets the domain emphasis (section 3) |

## 2. Analysis modes

- **initial**: the full workflow in your system prompt.
- **follow_up**: reuse earlier results by their paths and provenance ids (`petro_get_record` re-reads any stored result), add the new data, and answer the new question completely. Identical calls are answered from the cache, so re-running a step costs nothing and returns the same numbers.
- **validation**: test an upstream claim against well data. For example, does a seismic amplitude anomaly coincide with log evidence of hydrocarbons at that depth? Report `supported`, `contradicted` or `unresolved`, with the discriminating data.
- **challenge**: seek well evidence against the favoured interpretation. Consider alternative parameters, alternative explanations of a feature, and bad-hole effects.
- **reanalysis**: rerun with the new data or parameters, list the replaced task in `supersedes_task_ids`, and mark replaced claims `superseded`.

## 3. Domain emphasis

| Domain | Emphasise | Watch for |
|---|---|---|
| Hydrocarbon | Net pay, hydrocarbon saturation, fluid type, contacts | Low-salinity water mimicking pay; residual hydrocarbons; invasion by oil-based mud |
| CO2 storage | Reservoir porosity and permeability (injectivity); caprock shale volume; brine salinity; pressure and temperature (CO2 phase and density) | CO2 density changes sharply near its critical point; capillary entry pressure needs mercury-injection data |
| Geothermal | Temperature and gradient, porosity and permeability, fractures | Raw bottom-hole temperatures read low; conductive clays make resistivity-based saturation meaningless |

## 4. Writing evidence

Each item states one fact, with its depth reference and its source:

```json
{"evidence_id": "E4", "statement": "Washout (caliper more than 1 in over bit size) at 1320.1-1330.2 m MD (1242.6-1251.3 m TVDSS); density and neutron there are unreliable",
 "classification": "measurement", "source_type": "derived", "source_reference": "prov:9c1e0f3a7b2d4e61 petro_qc_logs"}
```

- **Header facts** (datum, mud, bit size) are `data` / `project_data`, cited as the loaded file plus the load's provenance id.
- **Log features** from `petro_describe_interval` are `observation` / `derived`. What they mean goes in `interpretations` (`interpretation` / `derived`), naming the alternatives (section 5).
- **Parameters** go in `assumptions`, with value and source, e.g. `m = 2.0 (assumption, range 1.8-2.2; sensitivity required)`, or `Rw = 0.05 ohm.m at 75 degC (water sample, W2 DST)`. Cite the `petro_parameters` provenance id.
- **Upstream facts** are cited by their full id (e.g. `T01-strat/E3`), with `source_type` `upstream_specialist`.
- **Never paste curves or tables into statements.** Export them (`petro_export`) and cite the file.

## 5. Features have alternatives

| Feature | Common meaning | Alternatives to name |
|---|---|---|
| Strong neutron-density crossover (over 0.06 v/v) | Gas | Light oil or condensate; very low porosity sandstone on a limestone scale; washout (check caliper); an uncorrected neutron |
| Mild crossover (0.03-0.05 v/v) | Clean water-bearing sandstone read on the limestone scale | Gas in low porosity; light hydrocarbons |
| Wide neutron-density separation | Shale or bound water | Heavy minerals; a washout on the density |
| Deep resistivity high relative to the interval | Hydrocarbons | Tight (low-porosity) rock; fresh formation water; a resistive mineral (anhydrite, coal) |
| Shallow above deep resistivity | Invasion by mud filtrate more resistive than formation water (Rmf > Rw) in a permeable bed | Oil-based mud; salinity contrast rather than hydrocarbons |
| Low gamma ray | Clean sand or carbonate | Clean but tight rock; radioactive-mineral-free shale is rare but possible |
| High gamma ray | Shale | Radioactive feldspar, mica or uranium in a clean reservoir: check with density, neutron and spectral gamma if present |

A fluid claim built only on log features stays `proposed` or `partially_supported` until pressures, tests, samples or calibrated saturation support it.

## 6. Missing data

Each entry is exactly `<specialist>: <item> — <form> — <why>`:
- `data_provider: W1 deviation survey — CSV md_m,inc_deg,azi_deg — TVDSS of tops and contacts`
- `data_provider: formation water sample or Rw — ohm.m at a stated temperature — water saturation cannot be computed without it`
- `stratigraphy: correlated tops for W3 — MD per top — zone averages consistent with W1 and W2`
- `regional_geology: formation-water salinity range for Unit B — ppm NaCl equivalent — bound Rw where no sample exists`

## 7. Products for other specialists

Much of your value reaches users through other specialists, so products follow fixed formats:
- **`time_depth`**: CSV twt_ms, depth_m below the SRD.
- **`elastic_logs`**: CSV depth_m, vp_m_s, vs_m_s, rhob_g_cc (+ twt_ms).
- **`well_header`**: JSON.
- **`zone_summary`**: JSON.
- **`pressure_profile`**: CSV. **`fluid_contacts`**: JSON.
- **`temperature_profile`**: CSV.

Each has a JSON sidecar with units, datum, source and provenance.

Handling rules:
- **Refer to a product only by its reference**, `@petrophysics-agent/products/<file>`, copied from `result.products[].conclusion_line` into `conclusions` as `Product: <kind> — <ref>`.
- **Never paste a product's rows** into statements.
- **State the datum** (SRD) and the fluid case in the same conclusion, or in a measurement citing the product.

## 8. Self-check before emitting

1. Every number has a `prov:` id in `source_reference`.
2. Every depth names MD or TVDSS, and the datum is stated (or stated as unknown).
3. Every parameter has a source; assumptions are listed with a range.
4. Bad-hole intervals that bear on the question are stated.
5. No property is claimed that needs uncomputed or uncalibrated data.
6. Every `required_outputs` item is produced, or listed in `limitations` with the reason.
7. Figures and downloads the user asked for appear in `conclusions` as `Figure: …` / `Download: …` lines, and every product another specialist needs appears as a `Product: …` line.
8. The object is under about 10,000 characters, with no curves or tables in statements.
