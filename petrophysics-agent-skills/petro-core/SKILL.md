---
name: petro-core
description: Core procedure for every petrophysics task. It covers the SpecialistTask and its analysis mode, the domain emphasis (hydrocarbon, CO2 storage, geothermal), and evidence items and claims with provenance and depth references. It also covers the alternatives to name for log features, the missing_data format and the final self-check. Load at the start of every task.
---

# Petrophysics core procedure


## What a figure is and is not

A figure illustrates a claim that a tool computed. It never carries a claim. You never read a number, depth, area or boundary off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested. That is the one way a picture can damage an argument.

You do not see the figures you render: the tool returns a URL, not an image. Check the numbers that the tool returns instead. Figures are for the person, Geo Oracle's answer and the report. So the caption must say what the figure shows, from which product and version, well enough to be read without you.

## 1. Read the task

Use every field.

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
- **follow_up**: reuse earlier results by their paths and provenance ids (`petro_get_record` re-reads any stored result). Add the new data, and answer the new question completely. The cache answers identical calls, so a step that you run again costs nothing and returns the same numbers.
- **validation**: test an upstream claim against well data. For example, does a seismic amplitude anomaly coincide with log evidence of hydrocarbons at that depth? Report `supported`, `contradicted` or `unresolved`, with the discriminating data.
- **challenge**: seek well evidence against the favoured interpretation. Consider alternative parameters, alternative explanations of a feature, and bad-hole effects.
- **reanalysis**: rerun with the new data or parameters. List the replaced task in `supersedes_task_ids`, and mark replaced claims `superseded`.
`check_outcomes`: when the brief lists 'Claims to test', give one entry per claim: `target` (the claim id), `verdict` (`supported`, `contradicted`, `unresolved` or `not_tested`), the `evidence_ids` of your own evidence that decide it, and a one-sentence `basis`. A verdict says what your evidence shows, not whether the claim is true. A blind validation has no such list: leave it empty.

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
 "classification": "measurement", "source_type": "derived", "source_reference": "prov:9c1e0f3a7b2d4e61 petro_qc_logs | m:petro/qc-logs"}
```

- **Header facts** (datum, mud, bit size) are `data` / `project_data`. Cite them as the loaded file plus the load's provenance id.
- **Log features** from `petro_describe_interval` are `observation` / `derived`. Put what they mean in `interpretations` (`interpretation` / `derived`), and name the alternatives (section 5).
- **Parameters** go in `assumptions`, with value and source. For example: `m = 2.0 (assumption, range 1.8-2.2; sensitivity required)`, or `Rw = 0.05 ohm.m at 75 degC (water sample, W2 DST)`. Cite the `petro_parameters` provenance id.
- **Upstream facts**: cite them by their full id (for example `T01-strat/E3`), with `source_type` `upstream_specialist`.
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

Much of your value reaches users through other specialists. So your products follow fixed formats:
- **`time_depth`**: CSV twt_ms, depth_m below the SRD.
- **`elastic_logs`**: CSV depth_m, vp_m_s, vs_m_s, rhob_g_cc (+ twt_ms).
- **`well_header`**: JSON.
- **`zone_summary`**: JSON.
- **`pressure_profile`**: CSV. **`fluid_contacts`**: JSON.
- **`temperature_profile`**: CSV.
- **`correlation_logs`** and **`well_tops`**: what the stratigraphy specialist correlates on. `petro_publish_correlation_logs` publishes them per well. `petro_prepare_wells` publishes them for a list of wells in one call (load, locate, publish): use it first for any multi-well task. If a task asks for a correlation, a panel or a framework downstream, it is incomplete until these are published.

Each has a JSON sidecar with units, datum, source and provenance.

Handling rules:
- **Refer to a product only by its reference**, `@petrophysics-agent/products/<file>`. Copy it from `result.products[].conclusion_line` into `conclusions` as `Product: <kind> — <ref>`.
- **Never paste a product's rows** into statements.
- **State the datum** (SRD) and the fluid case in the same conclusion, or in a measurement that cites the product.

## The plan, alternatives and history

Before your first tool call, write one `plan` row for each numbered required output. The row gives the tool chain that serves the output, and its status. At the end, bring the rows up to date:
- `delivered`, with the product (`@agent/products/<file>`) or the provenance id that holds it;
- or `partial`, `not_done` or `refused`, with what stopped it in `blocked_by`.

The gateway compares the rows with the brief. An output left open goes to Geo Oracle's checkpoint by its token. After you write the plan, read the cautions of the tools in the plan once: `petro_method_provenance(tool='<tool>, <tool>', verbosity='cautions')`. A caution says what a method means for its number (a bound, a proxy, an uncalibrated value, a design rule). Each result also carries the cautions of the methods it ran. Carry every caution that applies to a result into its limitations.

Every claim of class interpretation or hypothesis carries `alternatives`. These are the strongest rival readings of the same data that you can name from your own domain. Each rival has the `test` that tells it from the claim, and the `result` of that test or 'not run: <why>'. If the data allow it, run the test with your tools. A rival without a test is not an alternative. A weak rival listed to fill the field misleads. Rivals to consider here: a shale or mineral effect on the log, a borehole or invasion effect, a fluid effect read as lithology, a depth mismatch.

## Evidence from the person

Geo Oracle passes what the person said as items of `available_evidence`. An item of kind `expert_input` is a fact with its source: use it as far as that source allows, and cite it. An item of kind `expert_judgement` is the person's opinion. At most, it can support an interpretation. On its own, it never supports a measurement, a status, a class or a quantity.

## Methodological provenance

Every tool result carries these fields:
- `methods`: the ids of the published methods that the computation ran, such as `m:petro/archie-saturation`;
- `citation_version`: the state of the bibliography that the answer reflects.

If a tool selects its method by a parameter (`petro_vshale`,
`petro_saturation`, `petro_porosity`, ...), it lists only the methods that the call used, in the
order they ran. For example, a `petro_saturation` result with `model: archie` and Rw corrected along the
well carries `m:petro/archie-saturation`, `m:petro/formation-temperature-linear` and
`m:petro/arps-resistivity-temperature`.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:petro/archie-saturation`). Name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user. The method's name in words lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:petro/archie-saturation` says which
method. It never says who published it or when. An author surname may be part of a
method's name, but a year never is. The reference lives in the registry, and
only `petro_method_provenance` returns it.

If you are asked one of these questions, call `petro_method_provenance` and report what
it returns:
- which equation was used;
- what publication supports a method;
- what the method assumes;
- how far it has been verified.

Call it with the `method_id` from the result, or with a tool name for every method that
tool implements.

Do not answer such a question from your own knowledge. An unverified reference presented
as this system's provenance is worse than an admitted gap. It is the one failure that this
mechanism exists to prevent. If the tool reports that no source has been recorded, say
exactly that.

A reference states where a method was published. Status states how far the system's
current tool implementation of this method has been verified against that reference.
Report the two separately. Never let a citation stand in for a demonstration that the
implementation is correct.

## 8. Self-check before emitting

1. Every number has a `prov:` id **and** the result's `m:` method id(s) in `source_reference`. A missing `m:` id is a missing citation. Evidence fields list evidence ids (E#, or full upstream ids), never claim ids. Claims link to claims only through `depends_on_claim_ids`.
2. Every depth names MD or TVDSS, and the datum is stated (or stated as unknown).
3. Every parameter has a source. Assumptions are listed with a range.
4. Bad-hole intervals that bear on the question are stated.
5. No property is claimed that needs uncomputed or uncalibrated data.
6. Every `required_outputs` item is produced, or listed in `limitations` with the reason.
7. Every figure drawn (by `petro_render` or in a tool's `result.figures`) appears in `figures`, and in `conclusions` as a `Figure: <caption> — <url> — supports <claim ids>` line. Every download the user asked for appears as a `Download: …` line. Every product that another specialist needs appears as a `Product: …` line.
8. The object is under about 10,000 characters, with no curves or tables in statements.

## Methods statements

Every result that ran a computation carries `methods_statements`: one entry for each method, or group of methods used together. The `text` goes into the investigation report as you write it. So write it for the reader of a field report:
- 1 to 3 sentences, one fact each, at most 25 words each, active voice;
- what you did, to which data, the key choices and parameter values, any deviation from the published method, and any limitation that affected this result;
- no ids and no textbook account of the method: copy the method ids and provenance ids from your tool results into `method_ids` and `provenance_ids`.

The gateway checks every id against the stored computations and sets aside a statement whose ids do not resolve.

<!-- end of skill: petro-core -->
