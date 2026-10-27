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
  - `Process audit:` lines are the gateway's own reading of the specialist's run. Address each before you accept the result (section 3a).
  - `Id check:` lines name an evidence item or methods statement whose provenance or method id does not resolve to a stored computation. The gateway sets the item aside: do not rely on it. Ask the specialist to correct it in a follow-up of the same task. While any item stays uncorrected, the gateway refuses a checkpoint that approves no further work. Put an item you will not rely on in the record as `set_aside: [{evidence_id, reason}]`.
  - The gateway only flags a methods statement that writes ids in its text, or states numbers the cited computations do not hold. Check it against the tool results. If it is wrong, ask for a correction. Methods statements go into the investigation report.
- **`omitted`**: fetch the trimmed sections you need before judging the result. Do not judge from a partial digest. `referenced_items` carries the statement of every item an included claim cites from an omitted section. Such a reference is not unresolved. Never make it the subject of a follow-up task.

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
- a model, panel or depositional claim whose ledger (V1-V7, C1-C7, S1-S7) shows a failed test "explained" with no evidence cited, or a `[transfer]` statement without its T1-T5 record: the status is not earned;
- a high-confidence conclusion with no stated uncertainty.

A claim that fails a material check is not established evidence. Record it as unsupported, or send it for follow-up.

## 2a. Plan, rivals and history

- **The plan.** Each numbered required output has a `plan` row. The gateway warns about a missing row, a delivered row with no receipt it can find, and each output left open, by its token (`T05-struct#2`). At the checkpoint, carry each open token forward or dismiss it.
- **Rivals.** An interpretation comes with `alternatives`, each with a test and its result. Never drop a rival: record it in the ledger beside its claim until a test or a checkpoint resolves it. An interpretation without rivals, or a rival without a test, is a question for the specialist.
- **History.** A structural or stratigraphic interpretation comes with its `evolution`, oldest event first. Ask for it when it is missing.
- **Cautions.** `method_cautions` gives what the methods behind each claim mean for its numbers. A caution not reflected in the limitations is a question for the specialist.

## 3a. Method and process checks

Section 3 asks whether a claim follows from its evidence. These checks ask whether the specialist produced the evidence the right way. They are mechanical: do them every time.

1. **Process-audit flags** (`warnings` starting `Process audit:`):
   - *skills not read*: the specialist worked without those procedures. Check its method against them. When the method matters, ask it (same session) to read each skill on its own and redo the work.
   - *tool calls refused*: the tools blocked something. Check that the delivered result does not rest on a workaround.
   - *override in <tool>*: a tool ran on an input it does not normally accept. Accept it only if the person asked for exactly that. Otherwise, follow up.
   - *subset of faults kept*: read the stated reason. Reject a subset chosen to match an expected count.
   - *skills not opened*: in this session, the specialist never read a procedure its tools require (no failed read; it was simply never opened). Handle it as for *skills not read*.
   - *refused call repeated*: the specialist resent a refused call unchanged, instead of changing it. The turn it spent may have cost other parts of the task. Compare the result with what you asked for, and send the uncovered parts back (same session).
   - *qc suggestion not followed*: a tool's QC named a setting to rerun with, and no later call used it. Ask for the rerun (same session), or accept with the reason written down.
   - *qc fix not acted on*: an error flag named its fix (tool and change), and the run neither called it nor said why. Ask for the fix (same session), or accept with the reason.
   - *typed channel recipe before the generic one*: the specialist assumed a channel type (meandering, submarine) without the user naming it, a well establishing it, or a generic `channel` run first. Ask it (same session) to run the generic recipe and report the measured planform and fill. A type the data were not asked to show is not a finding.
   - *no tool calls*: the specialist ran nothing in that turn, and its result restates the brief. A validation that ran nothing has validated nothing. Send it back, or keep only what the brief itself established.
   - *sandbox unavailable*: the specialist's sandbox never started (for example, the sandbox service out of disk), so it read no skill. The cause is the infrastructure. Tell the person, and treat the method as unchecked.
   - *tool calls failed*: a tool errored, not refused. Find what the specialist did instead, in its assumptions and limitations. A substitute input (a guessed location, an assumed velocity) changes what the numbers mean.

   Put what you did about each in the checkpoint record (`done[].process_audit`). The gateway refuses the next wave until every flag is addressed.

   **Read every result's assumptions** before you rely on it. If the first view trimmed them, call `get_specialist_result(job_id, sections=["assumptions"])`. The checkpoint refuses a wave whose assumptions were never delivered to you. An unread assumption, such as "regional queries used an approximate point", stays under every number built on it.
   **Check every location a specialist used** against the investigation's coordinates: the AOI centroid in a regional result, the well position in a seismic one. A location that differs is a method error, whatever the result says.
2. **The method fits the deliverable.** The `m:` ids in `source_reference` name the methods used. A deliverable must carry the method that makes it:

   | Deliverable | Must include |
   |---|---|
   | fault volume, fault set, fault count | `m:seis/interpret-faults`, or `m:seis/ransac-plane-fault-extraction` with `m:seis/fault-surface-sampling` |
   | channels or other stratigraphic bodies, **including "none found"** | `m:seis/segment-volume` (the generic `channel` recipe unless the person or a well named the type) |
   | a well tie (correlation, phase, shift) | `m:seis/well-tie-shift-phase-scan`; a petrophysics "tie-ready" product is the input to a tie, not a tie |
   | a depth map | `m:seis/time-depth-conversion-laws` or a table conversion on a horizon map (`m:seis/correlation-horizon-autotracker` behind it); a converted volume is not a map |
   | a distance from a well to a feature | `m:seis/feature-distance`; a bounding-box overlap is not a distance |
   | channel type (meandering, submarine, tidal, ...) | `m:seis/recipe-channel` with the measured `planform` and fill, or a typed recipe whose `type_basis` names the user or a well |

   A fault volume without its extraction method is a thresholded attribute, not faults: follow up. "No channels" without `m:seis/segment-volume` means the channel workflow never ran. Send it back to run the generic recipe, over a `z_window` when there are no horizons.

   **A channel result is read with its QC**, not its count. Refuse (send back) a body result when:
   - a body's extents are the survey's (one channel does not span every inline and crossline);
   - `qc.extent` flags a body, or `qc.stability.merged_bodies` is not empty, and the result reports those bodies as separate channels;
   - faults were extracted in the same investigation, and the recipe ran without `faults` (`qc.fault_overlap` absent);
   - or `qc.selection.final_exceeds_cap` is true.

   "Found at 9/9 settings" is persistence, not identity: a merger of channels, sheets and fault zones is perfectly stable. For other deliverables, if the ids do not make the method plain, ask the specialist which method produced it.
3. **The input fits the method.** Read what each computation ran on (the evidence statements and `source_reference`). Fault likelihood, fault extraction and other trace-correlation methods run on seismic amplitudes. A result computed on an attribute (coherence, a likelihood, a mask) where seismic was expected is a method error, unless the person asked for it.
4. **Compare with earlier runs of the same question.** If a rerun changes a key number by a large factor (a likelihood percentile ten times smaller, a fault count from 3 to 9), find out why before you accept it. The inputs or the method changed, not the earth.
5. **Faults come with persistence.** `seismic_interpret_faults` reports how many of the nearby settings find each fault (`found_at`). Report the drawn faults with that table. Report a count that changes across settings as unstable; never tune it.
6. **After pushback, say what changed the method.** When a rerun follows the person's objection, tell them whether the evidence or their request changed the method. Also tell them what the specialist found about why the first result missed.

## 4. Independence and double counting

Trace each supporting id back to its origin:

- Two specialists who both rely on `T03-seis/E2` provide one line of evidence, not two.
- A local interpretation built on a literature analogue, plus that same analogue, is one line.
- Log-derived facies and core-described facies from the same interval are independent only if the log interpretation was not calibrated to that core description.
- A re-check by the same specialist is a second line only if it ran in a fresh session (`session_route` says so), was briefed blind (the question, not the earlier answer), and used a different method, parameters or data. A re-check in the investigation's session, or one that repeated the same method on the same data, is the same line. The specialist's tools return the earlier result for identical inputs.

- A validation or challenge result carries the gateway's `independence` class, read from the computations it cites. Count lines from the class, not from the result's prose:
  - validation: `same computation`, `same method and data, other parameters` and `no computation cited` are the same line. `new evidence` is a second line. `earlier computations of other tasks` re-uses other tasks' results, so it adds only the lines those tasks hold. `not classified` means the checked task is not stored: judge it from `session_route`, the brief and the `m:` ids;
  - challenge: `tested with new computation` ran a new test; `argued from existing results` tested the claim by argument only.
- A verdict in `check.outcomes` is one line, weighed by its class, never proof. Answer each `chk` item with your reading.

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

## 5a. A deliverable is delivered with its uncertainty, not withheld

**A process-audit flag sends the procedure back, not the measurement.** When the gateway flags a skill not opened, a refused call or an override, the follow-up asks the specialist to read the procedure and check its method against it. A number measured with the right tool on the right inputs stands, unless that check finds it wrong. A rerun that gives a different number is not "the audited one". Report both with their inputs: the difference is the finding. For example, a well tie of 0.98 on the supplied checkshots against 0.85 on a derived time-depth product says the derived product is the weaker input, not that 0.85 is the tie.

A specialist that returns an argument in place of a product the person asked for ("a survey-wide depth map is not defensible because the single-well time-depth is not a validated lateral velocity model"; "the bodies are candidates, so no channel volume") has not answered. The product exists at a stated uncertainty. A depth map from one well's time-depth is the standard first depth map, with the lateral velocity unknown. A channel volume is the extraction with its QC. Send it back as a `follow_up`, with the product in `required_outputs` and the uncertainty to state. Never relay "not defensible" as the result. When the person challenges "X was not produced", the answer to the challenge is X.

## 6. Decide the disposition

| Disposition | When | Next step |
|---|---|---|
| Accept | Passes the checks (sections 3 and 3a); the classification is honest | Add to the ledger with its status |
| Accept with qualification | Usable, but has stated limitations | Add to the ledger with the qualification attached |
| Follow-up (the investigation's session) | A missing item, a clarification, or format problems | Brief it as a `follow_up` with `previous_task_id`; the gateway continues the session |
| Validate (fresh session, or another specialist) | A load-bearing claim with a single line of evidence, where independent evidence exists | Validation task, briefed blind with a different method (`geo-specialist-tasking` section 2) |
| Challenge (fresh session) | An interpretation that plays or prospects depend on, with under-explored alternatives | Challenge task, briefed with the claims and how they were made |
| Reanalysis | An upstream input changed, or a QC failure | Reanalysis task with `previous_task_id` |
| Reject | Fabricated, or unsupported on a material point | Record why; do not propagate |

A claim is **load-bearing** if reservoir, seal, trap, charge, timing, prospect geometry or a drilling implication depends on it. Load-bearing claims deserve validation whenever independent evidence exists.

## 7. Literature results

- Distinguish sources actually retrieved and inspected from sources only described or recalled. Cite only references the tool returned.
- Check that bibliographic details are complete enough to verify: authors, year, title, venue, DOI where available.
- Judge applicability to the study area: location, age, depositional or structural setting, data vintage.
- Keep published interpretations, analogues and local evidence apart.
- Preserve disagreement within the literature, and note gaps in coverage.

## 7a. Figures

Record a result's `figures` entries (or its `Figure:` conclusion lines) in the ledger, with the claims each supports. Check the claims the usual way: a figure changes nothing about a claim's status. A claim you would not have accepted without the picture is not accepted with it. If you reject or lower a figure's `supports` claim, keep the figure in the ledger and do not show it in the answer.

Send a render-only follow-up in the same session for a result whose claims rest on a product with a renderer and that returned no figure for it, or for a `Process audit: figures not carried` flag: "Render the figures for <products>, and return each in `figures` with the claim ids it supports." Accept a result without them only with the reason (for example, the product has no renderer).

## 8. Record the review

Update the ledger:
- the task's disposition, and the method and process checks done (section 3a) with what you did about each process-audit flag;
- the claims you now rely on, with classes, sources, independent lines and status;
- for each, the `prov:` and `m:` tokens from the specialist's `source_reference`, copied exactly. If a result's `source_reference` carries no `m:` id, the specialist did not attribute it to a method. Note it as such; do not infer one;
- any item the id check set aside, and whether it was corrected or declared not relied on;
- new contradictions and uncertainties;
- anything that must be re-checked downstream.

<!-- end of skill: geo-result-review -->
