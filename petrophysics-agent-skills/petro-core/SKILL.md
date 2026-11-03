---
name: petro-core
description: Core procedure for every petrophysics task. It covers the SpecialistTask and its analysis mode, the domain deliverables (hydrocarbon, CO2 storage, geothermal, natural hydrogen, hydrogen storage), the steps that apply to each kind of task, the reconcile tolerances, evidence items with provenance and depth references, the alternatives for log features, the missing_data format and the final self-check. Load at the start of every task.
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

- **initial**: the procedure in your system prompt, with the steps that section 3a marks for the task.
- **follow_up**: reuse earlier results by their paths and provenance ids (`petro_get_record` re-reads any stored result). Add the new data, and answer the new question completely. The cache answers identical calls at no cost.
- **validation**: test an upstream claim against well data. For example, does a seismic amplitude anomaly coincide with log evidence of hydrocarbons at that depth? Report `supported`, `contradicted` or `unresolved`, with the discriminating data. A blind validation names no claim: answer the question of the brief from your own evidence.
- **challenge**: seek well evidence against the favoured interpretation. Consider alternative parameters, alternative explanations of a feature, and bad-hole effects.
- **reanalysis**: rerun with the new data or parameters. List the replaced task in `supersedes_task_ids`, and mark replaced claims `superseded`.
`check_outcomes`: when the brief lists 'Claims to test', give one entry per claim: `target` (the claim id), `verdict` (`supported`, `contradicted`, `unresolved` or `not_tested`), the `evidence_ids` of your own evidence that decide it, and a one-sentence `basis`. In a challenge, also give `rival_claim_id` (your claim that states the alternative). A verdict says what your evidence shows, not whether the claim is true. A blind validation has no such list: leave it empty.

## 3. Domain emphasis

| Domain | Deliverables | Watch for |
|---|---|---|
| Hydrocarbon | Net pay, hydrocarbon saturation, fluid type, contacts | Low-salinity water mimicking pay; residual hydrocarbons; invasion by oil-based mud |
| CO2 storage | Porosity, net thickness and permeability (injectivity); caprock shale volume and porosity; brine salinity; pressure, temperature and CO2 density (`petro_fluid_properties`); seal capacity from MICP (`petro_saturation_height`, method `seal_capacity`, `buoyant_fluid` `co2`) | A flagged sensitive state needs low and high cases. Pure CO2: no impurities, no dissolution. Low permeability is not a seal-capacity measurement |
| Geothermal | Temperature and gradient with their error (`petro_temperature`); thermal conductivity, heat capacity, heat production and interval heat flow (`petro_thermal`); porosity and matrix permeability | Raw bottom-hole temperatures read low; conductive clays and alteration make resistivity-based saturation meaningless; brine relations extrapolate above 100 degC (the tools warn); logs give matrix, not fracture, permeability |
| Natural hydrogen | Host-rock description: shale volume, porosity, lithology, spectral gamma ray (K, TH, U); temperature; H2 density | Neutron-density crossover is light gas of unknown composition. In serpentinised or ultramafic hosts, neutron-based shale volume and porosity and Archie Sw are unreliable (bound hydroxyl, conductive magnetite): ask for core and XRD |
| Hydrogen storage | As CO2 storage, with `h2` as the fluid | H2-brine interfacial tension and contact angle need a source: register them as assumptions with ranges. Pure H2: dissolution and microbial loss are not modelled. A seal for methane is not shown to be a seal for H2 |

**Hydrogen.** Mark every H2 interpretation as emerging practice. Without a gas analysis, a log-based H2 claim stays `proposed`. Archie Sw and pressure gradients do not depend on the gas type, but H2 density comes only from `petro_fluid_properties` (`h2`). Before you use mud-gas H2, ask for the detector and bit types, and look for drilling markers (CO, ethene, propene). Report the zone temperature for the play specialist's microbial-window check. Route H2-brine interfacial tension to `literature_review` and the H2 source to `subsurface_play`.

## 3a. Which steps apply

M = mandatory. C = conditional (the letter gives the condition). U = unnecessary: no plan row. A task of several kinds takes, for each step, the strongest mark of its columns (M over C over U). A `ref` 0 row is only for a step that the objective needs but that is unreliable here or blocked: one short line (`output` at most 60 characters, `blocked_by` at most 80).

| Step | Tie | Net pay | Pressure, contacts | Storage, geothermal, H2 | Validation, challenge |
|---|---|---|---|---|---|
| Load, QC, depth references | M | M | M | M | M |
| Zones and features | C(a) | M | C(b) | M | C(c) |
| Shale volume, two physics | U | M | C(b) | M | C(c) |
| Matrix and lithology | U | C(d) | U | C(d) | C(c) |
| Porosity | U | M | C(b) | M | C(c) |
| Core shift, grain density, SCAL; porosity again | U | C(e) | U | C(e) | C(c) |
| Temperature model | U | M | C(f) | M | C(c) |
| Rw and Sw | U | M | C(b) | C(g) | C(c) |
| Permeability, saturation-height, seal capacity | U | C(e) | C(b) | C(e) | C(c) |
| Net pay with uncertainty | U | M | U | C(h) | C(c) |
| Pressure gradients and contacts | U | C(i) | M | C(i) | C(c) |
| Fluid properties | U | C(i) | C(f) | M | C(c) |
| Elastic logs, time-depth, well header | M | U | U | C(j) | C(c) |
| Thermal properties | U | U | U | C(k) | C(c) |

Conditions: (a) tops exist, for marker times. (b) a log contact or fluid reading is compared with the pressures. (c) the tested claim rests on it; a challenge also runs the rival parameters and models. (d) PEF, core mineralogy or mixed lithology. (e) core, SCAL or capillary-pressure data exist (seal capacity: MICP). (f) fluid densities or phases are in question. (g) a gas, CO2 or H2 saturation is in question; not for geothermal. (h) net thickness for storage; not for geothermal. (i) pressure data exist. (j) monitoring or a seismic tie is asked for. (k) geothermal. Reconcile (section 3b) is mandatory wherever two estimates of one quantity exist.

## 3b. Reconcile: pairs and tolerances

| Pair | Tolerance |
|---|---|
| Core and log porosity, after the depth shift | Bias within about ±0.01 v/v |
| Water-sample and log Rw (SP or Pickett) at one temperature (`petro_rw` `report_at_temp_c`, the same value for each source) | About 20 % |
| Rw of different wells at one temperature (`petro_rw` `report_at_temp_c`) | About 20 % |
| Gamma-ray and neutron-density or SP shale volume | About 0.05 v/v |
| Pressure free-water level and log contact | The contact's `uncertainty_m` plus the capillary entry height |
| Log SW and SW_SHF, above the transition zone | About 0.05 v/v |
| Alternative Sw models | No tolerance: report the spread as model uncertainty |
| Lithology alternatives, fluid indicators | No number: name the data that discriminate |
| Direct and empirical values (core and Timur permeability; measured and predicted Vs) | The band that the empirical result states |

Each pair's `measurement` gives the two values, their difference, the tolerance and both provenance ids. A contradiction's `description` names the pair. Prefer the method that suits the setting, for example core over logs, or a shaly-sand model over Archie in shaly rock.

## 4. Writing evidence

Each item states one fact, with its depth reference and its source:

```json
{"evidence_id": "E4", "statement": "Washout (caliper more than 1 in over bit size) at 1320.1-1330.2 m MD (1242.6-1251.3 m TVDSS); density and neutron there are unreliable",
 "classification": "measurement", "source_type": "derived", "source_reference": "prov:9c1e0f3a7b2d4e61 petro_qc_logs | m:petro/qc-logs"}
```

- **Classification** is the evidence type: data, observation, measurement, interpretation, inference, hypothesis, assumption or unknown. The support level (`supported`, `partially_supported`, `unresolved`, ...) goes in the claim's `status`, never in `classification`.
- **Header facts**: cite the loaded file plus the load's provenance id.
- **Log features** from `petro_describe_interval` are `observation` / `derived`. Put what they mean in `interpretations` (`interpretation` / `derived`), and name the alternatives (section 5).
- **Parameters** go in `assumptions`, with value and source. For example: `m = 2.0 (assumption, range 1.8-2.2; sensitivity required)`, or `Rw = 0.05 ohm.m at 75 degC (water sample, W2 DST)`. Cite the `petro_parameters` provenance id.

## 5. Features have alternatives

| Feature | Common meaning | Alternatives to name |
|---|---|---|
| Strong neutron-density crossover (over 0.06 v/v) | Light gas, composition unknown | Light oil or condensate; very low porosity sandstone on a limestone scale; washout (check caliper); an uncorrected neutron |
| Mild crossover (0.03-0.05 v/v) | Clean water-bearing sandstone read on the limestone scale | Gas in low porosity; light hydrocarbons |
| Wide neutron-density separation | Shale or bound water | Heavy minerals; a washout on the density |
| Deep resistivity high relative to the interval | Hydrocarbons | Tight (low-porosity) rock; fresh formation water; a resistive mineral (anhydrite, coal) |
| Shallow above deep resistivity | Invasion by mud filtrate more resistive than formation water (Rmf > Rw) in a permeable bed | Oil-based mud; salinity contrast rather than hydrocarbons |
| Low gamma ray | Clean sand or carbonate | Clean but tight rock; radioactive-mineral-free shale is rare but possible |
| High gamma ray | Shale | Radioactive feldspar, mica or uranium in a clean reservoir: check with density, neutron and spectral gamma if present |

A fluid claim built only on log features stays `proposed` or `partially_supported` until pressures, tests, samples or calibrated saturation support it.

## 6. Missing data

Each entry is exactly `<owner>: <item> — <form> — <why>`, with one entry per owner. The owner `user` is for data only the person can supply. Join several items of one owner with `;`:
- `user: W1 deviation survey; a formation water sample or Rw — CSV md_m,inc_deg,azi_deg; ohm.m at a stated temperature — TVDSS of tops and contacts; water saturation needs an Rw source`
- `stratigraphy: correlated tops for W3 — MD per top — zone averages consistent with W1 and W2`
- `regional_geology: formation-water salinity range for Unit B — ppm NaCl equivalent — bound Rw where no sample exists`

## 7. Products for other specialists

Your products follow fixed formats:
- **`time_depth`**: CSV twt_ms, depth_m below the SRD.
- **`elastic_logs`**: CSV depth_m, vp_m_s, vs_m_s, rhob_g_cc (+ twt_ms).
- **`well_header`**: JSON.
- **`zone_summary`**: JSON.
- **`pressure_profile`**: CSV. **`fluid_contacts`**: JSON.
- **`temperature_profile`** and **`thermal_properties`**: CSV. **`seal_capacity`**: JSON.
- **`correlation_logs`** and **`well_tops`**: what the stratigraphy specialist correlates on (`petro_publish_correlation_logs`, or `petro_prepare_wells` for several wells). A task that asks for a correlation, a panel or a framework downstream is incomplete until these are published.

Each has a JSON sidecar with units, datum, source and provenance.

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
5. Two tiers: an absent input is in `missing_data`, and an output with no tool is in `limitations`. A computation that an uncalibrated or assumed input allows is run, ranged and capped.
6. Every computation the objective needs and you did not run is in the plan. Each section 3b pair that the data allow has its measurement, and a contradiction outside its tolerance.
7. Every `required_outputs` item is produced, or listed in `limitations` with the reason.
8. Every figure drawn is in `figures` and has a `Figure: …` line in `conclusions`. Every download the user asked for has a `Download: …` line, and every product that another specialist needs a `Product: …` line.
9. The object is under about 10,000 characters, with no curves or tables in statements.

## Methods statements

Every result that ran a computation carries `methods_statements`: one entry for each method, or group of methods used together. The `text` goes into the investigation report as you write it. So write it for the reader of a field report:
- 1 to 3 sentences, one fact each, at most 25 words each, active voice;
- what you did, to which data, the key choices and parameter values, any deviation from the published method, and any limitation that affected this result;
- no ids and no textbook account of the method: copy the method ids and provenance ids from your tool results into `method_ids` and `provenance_ids`.

The gateway checks every id against the stored computations and sets aside a statement whose ids do not resolve.

<!-- end of skill: petro-core -->
