---
name: seismic-core
description: Core operating procedure for every seismic interpretation task - task reading, tool-result handling, provenance citation, evidence classification, claim status, missing-data format and the findings JSON. Load at the start of every task.
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
| follow_up | Answer the narrower question. Reuse earlier results through `seismic_get_record` if their provenance ids are given. |
| validation | Re-measure the specific claims named. Report agreement or disagreement per claim. |
| challenge | Actively look for evidence against the named interpretation. Report what would falsify it. |
| reanalysis | Redo the work with the changed inputs or parameters named. Set `supersedes_task_ids`. |

## 2. Handle every tool result the same way

1. Check `execution_status`.
   - `completed`: the result is usable.
   - `running`: poll the matching `get_*_job_result` tool.
   - `rejected`: fix the arguments.
   - `error`: correct the call once, then record the failure in `limitations`.
2. Read `qc_flags` and `warnings`. They describe the conditions under which the numbers hold.
3. Note the `provenance_id`. It is how you cite the result.
4. Use `report_hint` to decide the section and classification. You may downgrade a hint (for example, treat an observation as unusable because of QC), but never upgrade it.

Never paste raw tool output into findings. Summarise the value and cite the provenance id.

## 3. Evidence items

| Classification | Use for | source_type |
|---|---|---|
| data | What files contain: geometry, headers, supplied tables | project_data |
| observation | Direct measurements on a section, from describe_section | project_data |
| measurement | Computed quantities: QC, attribute statistics, resolution | derived |
| interpretation | Geological meaning of measurements | derived |
| inference | Conclusion from interpretations | derived |
| hypothesis | Untested explanation | derived |

Good statements are specific and located. For example: "Reflector continuity falls to 0.28 between XL 248 and 250 over 300–700 ms on IL 120 (E3)." Every interpretation lists the evidence ids it rests on in `supporting_evidence_ids`, and its `assumptions` and `limitations`.

`source_reference` has this form: `prov:<id> <tool> <key args>`. For task-supplied evidence, use the task's own reference.

## 4. Claims and independence

A claim is a material geological statement Geo Oracle will reason with. Status rules:

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

## 7. Before you return

- [ ] Every number traces to a tool result or the task.
- [ ] Every `required_outputs` entry is covered or explained.
- [ ] Every interpretation has an alternative or a stated reason for excluding one.
- [ ] Claim statuses obey the independence rule.
- [ ] The output is one JSON object with no prose around it, under about 10,000 characters.
