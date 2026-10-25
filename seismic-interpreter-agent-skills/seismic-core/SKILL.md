---
name: seismic-core
description: Core operating procedure for every seismic interpretation task. It covers task reading, tool-result handling, provenance citation, evidence classification and claim status. It also covers the missing-data format and the findings JSON. Load at the start of every task.
---

# Seismic core procedure

## 1. Read the task before touching a tool

Take the following from the task:
- the **question**;
- the **required_outputs**, which are a checklist you must cover;
- the **analysis_mode**;
- the **evidence** supplied, meaning file paths, tables and upstream findings;
- the **constraints**;
- the **competing interpretations** to test.

What each mode asks of you:

| analysis_mode | What it asks |
|---|---|
| initial | Characterise the data and answer the question at the resolution the data allows. |
| follow_up | Answer the narrower question. If the task gives the provenance ids of earlier results, reuse them through `seismic_get_record`. |
| validation | Answer the question afresh. If the brief names a method, parameters or data different from the run being checked, use them. The brief may withhold the earlier answer on purpose. Report what you measured: Geo Oracle compares. |
| challenge | Actively look for evidence against the named interpretation. Report what would falsify it. |
| reanalysis | Redo the work with the changed inputs or parameters named. Set `supersedes_task_ids`. |

## 1a. What each deliverable takes

A `required_outputs` entry names a product, and the product has a chain. Start the chain early, because the attribute and tracking steps run as jobs. Export what the person is to receive.

| Deliverable | Chain | Export / figure |
|---|---|---|
| Data QC | `seismic_scan_segy` → `seismic_ingest_array` → `seismic_describe_volume` → `seismic_qc` (interval of interest) → `seismic_resolution_limits` | — |
| Fault volume, fault set, strikes and dips | `seismic_interpret_faults` on the seismic (z_range to the reflective interval) → read `qc` and persistence → `seismic_fault_throw` with a horizon where one exists → `seismic_publish_structure` | `seismic_export_volume` (labels); `faults_figure_markdown` |
| Channel or body volume (any stratigraphic feature or geobody) | envelope, semblance and chaos attributes (jobs) → `seismic_segment_volume` with `recipe: "channel"`, an `interval` hung on a tracked horizon, and `faults` → read `qc` → `seismic_publish_bodies` | `seismic_export_volume` (labels); `extraction_figure_markdown` |
| Horizon (time map) | `seismic_list_events` at the well → `seismic_fault_throw` for each fault between blocks → a seed per block at the throw-corrected time → `seismic_track_horizon` with `faults` (the fault set) → read `qc.faults` → `seismic_horizon_map` `structure` | `seismic_export_volume` on the map (csv); its own map figure; `seismic_render_section` with the horizon and faults |
| Depth map | the horizon above → `seismic_time_to_depth` (`method: table` with the well's time-depth, or the stated model) → `seismic_horizon_map` `structure` | export; its own map figure |
| Well tie | `seismic_convert_coordinates(well_header)` → `seismic_trace_events` (seabed and markers) → `seismic_extract_wavelet` → `seismic_synthetic` (`elastic_logs` + `time_depth`) → `seismic_well_tie` → `seismic_depth_function_check` where a second well exists | all three `*_figure_markdown` lines, always |
| Closure | the depth map above → `seismic_closure` → read `qc` (realisations) | its own figure: closures on the horizon |
| Distance from a well to each feature | `seismic_feature_distance` on the fault volume and the body volume, with the well header, its track and the time-depth table | — |
| Attribute volume | window → the attribute tool → `seismic_describe_section` on it | `seismic_export_volume` |
| Facies, key surfaces | `seismic_detect_terminations` on two orientations; `seismic_classify_facies` on an interval | its own class-map figure; `seismic_render_section` with the horizons |
| AVO, DHI, 4D, inversion | `seismic-qi` | per the tool |

**An initial full interpretation runs in this order**: QC → faults → channels and bodies (with the faults) → horizons (seeds in each fault block) → tie, when well products exist → depth map → closure → distances from each well to each feature. Each step feeds the next. If you track a horizon before the faults, or extract channels without them, you have to redo that step.

**Under the turn budget, deliverables come first**: the labelled volumes, maps and tie, each exported with its figure. They come before QC commentary and extra figures (3D views, further sections). A figure never substitutes for a product.

**A deliverable is delivered at its status.** A fault volume, channel volume or depth map from seismic alone is `partially_supported`. A depth map from one well's time-depth carries the unknown lateral velocity as its uncertainty. Say so in the claim and the limitations, and deliver the product. The status is the answer. It is never a reason to do any of these:
- withhold the product;
- call bodies something other than what the recipe measured;
- replace a map with an argument that a better one would need more data.

## 2. Handle every tool result the same way

1. Check `execution_status`.
   - `completed`: the result is usable.
   - `running`: poll the matching `get_*_job_result` tool.
   - `rejected`: fix the arguments.
   - `error`: correct the call once, then record the failure in `limitations`.
2. Read `qc_flags` and `warnings`. They describe the conditions under which the numbers hold.
3. Note the `provenance_id`. It is how you cite the result.
4. Use `report_hint` to decide the section and classification. You may downgrade a hint (for example, treat an observation as unusable because of QC). Never upgrade it.

Never paste raw tool output into findings. Summarise the value and cite the provenance id.

The tool shortens a long result to fit the reply. The warning says which parts (`result.<key>: replaced by a note`). The note keeps the verdict lines (`notes`, flags, the short lists that name what was found). Read the verdicts from the note. For the numbers behind them, call `seismic_get_record` with the result's `provenance_id` and `field` naming the part (`result.qc`, `result.qc.stability`, `result.bodies`). The part comes back whole and small. Never fetch the whole record of a large result, and never read a saved tool result with code in your sandbox.

## 3. Evidence items

| Classification | Use for | source_type |
|---|---|---|
| data | What files contain: geometry, headers, supplied tables | project_data |
| observation | Direct measurements on a section, from describe_section | project_data |
| measurement | Computed quantities: QC, attribute statistics, resolution | derived |
| interpretation | Geological meaning of measurements | derived |
| inference | Conclusion from interpretations | derived |
| hypothesis | Untested explanation | derived |

Good statements are specific and located. For example: "Reflector continuity falls to 0.28 between XL 248 and 250 over 300–700 ms on IL 120 (E3)." Every interpretation lists in `supporting_evidence_ids` the evidence ids it rests on. It also lists its `assumptions` and `limitations`.

`source_reference` has this form: `prov:<id> <tool> <key args> | m:<domain>/<name>`. It holds the `prov:` id and then every id from the result's `methods` field, copied exactly. For an attribute result, that is the one method actually run, for example `m:seis/edge-semblance`. The `m:` id is not optional: without it, Geo Oracle cannot say which method produced the number. For task-supplied evidence, use the task's own reference.

## 4. Claims and independence

A claim is a material geological statement that Geo Oracle will reason with. Status rules:

- **supported**: two or more independent lines agree, and none disagrees. State the independence explicitly.
- **partially_supported**: one line of evidence, or several lines that are not independent.
- **proposed**: plausible, but not yet tested against the data.
- **contradicted**: the data you measured disagrees with it.
- **unresolved**: the evidence conflicts and you cannot discriminate.

Independence test: would one error invalidate both lines? Coherence and curvature from the same volume share acquisition, processing and migration errors, so they are not independent. Seismic plus a well tie, seismic plus an upstream structural result, or two different surveys are independent.

## 5. Missing data

Each entry has exactly this form: `<specialist>: <item> — <form> — <why>`

- `<specialist>` is the gateway specialist that would supply it: wells_petrophysics, structural_geology, stratigraphy, sedimentology, regional_geology, literature_review, or `data_provider` for raw data.
- `<form>` is concrete, such as a file type, columns, units or coordinate system.
- `<why>` names the claim or required output it unblocks.

## 6. Mandatory limitations

Every result states the following:
- the vertical resolution (tuning thickness) and lateral resolution at the target, from `seismic_resolution_limits`, with the velocity used and its source;
- polarity (stated or unknown) and apparent phase (from `seismic_qc`, known only modulo 180°);
- the domain; time-domain geometry is not true geometry;
- the QC flags that bear on the question;
- which required outputs were not produced, and why.

## The plan, alternatives and history

Before your first tool call, write one `plan` row for each numbered required output. The row gives the tool chain that serves the output, and its status. At the end, bring the rows up to date:
- `delivered`, with the product (`@agent/products/<file>`) or the provenance id that holds it;
- or `partial`, `not_done` or `refused`, with what stopped it in `blocked_by`.

The gateway compares the rows with the brief. An output left open goes to Geo Oracle's checkpoint by its token. After you write the plan, read the cautions of the tools in the plan once: `seismic_method_provenance(tool='<tool>, <tool>', verbosity='cautions')`. A caution says what a method means for its number (a bound, a proxy, an uncalibrated value, a design rule). Each result also carries the cautions of the methods it ran. Carry every caution that applies to a result into its limitations.

Every claim of class interpretation or hypothesis carries `alternatives`. These are the strongest rival readings of the same data that you can name from your own domain. Each rival has the `test` that tells it from the claim, and the `result` of that test or 'not run: <why>'. If the data allow it, run the test with your tools. A rival without a test is not an alternative. A weak rival listed to fill the field misleads. Rivals to consider here: a processing or migration artefact, tuning, a velocity effect (pull-up or push-down), a multiple, the event on the wrong loop.

## Evidence from the person

Geo Oracle passes what the person said as items of `available_evidence`. An item of kind `expert_input` is a fact with its source: use it as far as that source allows, and cite it. An item of kind `expert_judgement` is the person's opinion. At most, it can support an interpretation. On its own, it never supports a measurement, a status, a class or a quantity.

## Methodological provenance

Every tool result carries these fields:
- `methods`: the ids of the published methods that the computation implements, such as `m:seis/edge-semblance`;
- `citation_version`: the state of the bibliography that the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:seis/edge-semblance`). Name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user. The method's name in words lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:seis/edge-semblance` says which
method. It never says who published it or when. An author surname may be part of a
method's name, but a year never is. The reference lives in the registry, and
only `seismic_method_provenance` returns it.

If you are asked one of these questions, call `seismic_method_provenance` and report what
it returns:
- which equation was used;
- what publication supports a method;
- what the method assumes;
- how far it has been verified.

Call it with the `method_id` from the result, or with a tool name for every method that
tool implements.

Never write that a method's equation, parameters, sources or verification are "not recorded" or "not retained" unless you have read that method's record at `verbosity: "full"` by its `method_id`. A summary or tool-level lookup is not the record. Its `full_record` line says where the record is. Never report "not retrieved" as "not recorded".

Do not answer such a question from your own knowledge. An unverified reference presented
as this system's provenance is worse than an admitted gap. It is the one failure that this
mechanism exists to prevent. If the tool reports that no source has been recorded, say
exactly that.

A reference states where a method was published. Status states how far the system's
current tool implementation of this method has been verified against that reference.
Report the two separately. Never let a citation stand in for a demonstration that the
implementation is correct.

## 7. Before you return

- [ ] Every number traces to a tool result or the task.
- [ ] Every `required_outputs` entry is covered or explained.
- [ ] Every interpretation has an alternative or a stated reason for excluding one.
- [ ] Claim statuses obey the independence rule.
- [ ] If any figure was drawn, the reply starts with each figure's markdown line and a one-line caption. Then comes the findings JSON, with no other prose, under about 10,000 characters.

## Methods statements

Every result that ran a computation carries `methods_statements`: one entry for each method, or group of methods used together. The `text` goes into the investigation report as you write it. So write it for the reader of a field report:
- 1 to 3 sentences, one fact each, at most 25 words each, active voice;
- what you did, to which data, the key choices and parameter values, any deviation from the published method, and any limitation that affected this result;
- no ids and no textbook account of the method: copy the method ids and provenance ids from your tool results into `method_ids` and `provenance_ids`.

The gateway checks every id against the stored computations and sets aside a statement whose ids do not resolve.

<!-- end of skill: seismic-core -->
