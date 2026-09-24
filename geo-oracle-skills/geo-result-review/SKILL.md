---
name: geo-result-review
description: Checklist and decision rules for Geo Oracle to evaluate every specialist result before relying on it. It covers separating what was established from what was interpreted, catching overreach and unsupported assertions, detecting double-counted evidence, classifying contradictions, and deciding whether to accept, qualify, follow up, validate, challenge, re-run or reject. Use every time a specialist tool returns a completed result, whenever two results disagree, and before any result feeds a play, prospect or risk task.
---

# Reviewing specialist results

A specialist result is evidence to be evaluated, not a verdict. Detail, fluency and confidence are not quality signals. Review every result before it enters the ledger or feeds another task.

## 1. First pass: is it usable?

- **`execution_status`** must be `completed`. Anything else is an execution outcome (see the system instructions), never a finding.
- **`scientific_status`**:
  - `insufficient_data` or `cannot_determine`: record it as a result.
  - `requires_clarification`: answer it through a follow-up in the same session, or by asking the user.
  - `not_evaluated`: the reply could not be parsed. Read `raw_output` with extra care, since nothing in it has been validated.
- **`warnings`**:
  - Parse repairs mean the specialist did not follow the output contract; the content may be sloppy elsewhere too.
  - "Unresolved … reference" means a claim cites evidence that does not exist in the result, so treat that link as unsupported.
  - "Unrecognised keys" means some content is only in the raw reply.
- **`omitted`**: fetch the trimmed sections you need before judging the result. Do not judge from a partial digest.

## 2. What was actually established?

Sort the content by evidence class, and check that each classification is honest:

- **Observations that are really interpretations.** "Channel observed on seismic" is an interpretation of an amplitude geometry. "Fault seal" is always an interpretation.
- **Measurements without provenance.** A measurement needs a source, method and units. A number with no source is not a measurement.
- **Literature presented as local fact.** Published regional evidence is literature evidence or an analogue, not project data.
- **Upstream interpretations restated as the specialist's own observations.** Restating them this way hides a dependency.

Reclassify in the ledger where needed, and note why.

## 3. Overreach and quality checks

Ask of every material claim:

1. Was the evidence it relies on actually supplied or retrieved? Check its ids against what you sent.
2. Is the method appropriate for those data? Are the required measurements present?
3. Are units, depth references (MD, TVD, TVDSS), time or depth domain and coordinate systems consistent?
4. Are the assumptions explicit? Are calibration requirements met?
5. Is the resolution adequate for the claim? For example, compare bed thickness with seismic tuning thickness, or well spacing with the scale of a correlation.
6. Were alternatives considered?
7. Does the conclusion exceed the evidence? Is the uncertainty stated?
8. Is it consistent with other validated results? Could a known limitation overturn it?

Red flags:
- specific numbers, depths or coordinates that appear in no evidence you sent;
- references that were not returned by `literature_review`;
- claims about data the specialist never received;
- "confirmed" or "demonstrated" resting on a single line of evidence;
- an empty `missing_data` or `limitations` list despite obvious gaps;
- a high-confidence conclusion with no stated uncertainty.

A claim that fails a material check is not established evidence. Record it as unsupported, or send it for follow-up.

## 4. Independence and double counting

Trace each supporting id back to its origin:

- Two specialists who both rely on `T03-seis/E2` provide one line of evidence, not two.
- A local interpretation built on a literature analogue, plus that same analogue, is one line.
- Log-derived facies and core-described facies from the same interval are independent only if the log interpretation was not calibrated to that core description.

Record the number of independent lines for each load-bearing claim in the ledger.

## 5. Contradictions

When results disagree, first rule out false conflicts. Many are:
- reference-frame mismatches: MD vs TVDSS, time vs depth, different datums or CRS;
- scale or resolution mismatches;
- different definitions: net-reservoir cutoffs, formation names.

Then classify the conflict as a data conflict, an interpretation conflict or a model conflict, and record it in the ledger with:
- the items involved (by id);
- the possible explanations;
- the evidence that would discriminate between them;
- the task that could test it, if any.

If no available evidence can discriminate, keep both interpretations. Do not pick one by majority, plausibility or convenience.

## 6. Decide the disposition

| Disposition | When | Next step |
|---|---|---|
| Accept | Passes the checks; the classification is honest | Add to the ledger with its status |
| Accept with qualification | Usable, but has stated limitations | Add to the ledger with the qualification attached |
| Follow-up (same session) | A missing item, a clarification, or format problems | Brief it with `specialist_session_id` |
| Validate (new session, or another specialist) | A load-bearing claim with a single line of evidence, where independent evidence exists | Validation task |
| Challenge (new session) | An interpretation that plays or prospects depend on, with under-explored alternatives | Challenge task |
| Reanalysis | An upstream input changed, or a QC failure | Reanalysis task with `previous_task_id` |
| Reject | Fabricated, or unsupported on a material point | Record why; do not propagate |

A claim is **load-bearing** if reservoir, seal, trap, charge, timing, prospect geometry or a drilling implication depends on it. Load-bearing claims deserve validation whenever independent evidence exists.

## 7. Literature results

- Distinguish sources actually retrieved and inspected from sources only described or recalled. Cite only references the tool returned.
- Check that bibliographic details are complete enough to verify: authors, year, title, venue, DOI where available.
- Judge applicability to the study area: location, age, depositional or structural setting, data vintage.
- Keep published interpretations, analogues and local evidence apart.
- Preserve disagreement within the literature, and note gaps in coverage.

## 8. Record the review

Update the ledger:
- the task's disposition;
- the claims you now rely on, with classes, sources, independent lines and status;
- new contradictions and uncertainties;
- anything that must be re-checked downstream.
