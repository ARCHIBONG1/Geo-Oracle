---
name: geo-specialist-tasking
description: Brief templates and rules for Geo Oracle's specialist tools (literature_review, regional_geology, seismic_interpretation, structural_geology, stratigraphy, sedimentology, wells_petrophysics, subsurface_play, prospect_target, risk_uncertainty) and their job tools (get_specialist_result, cancel_specialist_job, list_specialist_jobs). Use whenever you are about to call a specialist, especially for follow-up, validation, challenge or reanalysis tasks, parallel fan-outs, or when a call was rejected or returned warnings about its arguments.
---

# Specialist tasking

A specialist's answer can only be as good as its brief. A good brief gives the specialist:
- one geological question;
- the evidence needed to answer it;
- the upstream findings it builds on, with their ids;
- what must not be assumed;
- what output is needed and why.

## 1. Field guide

| Field | Put here | Common mistake |
|---|---|---|
| `objective` | What this task must achieve, in one sentence | Restating the whole investigation |
| `geological_question` | The specific, answerable question | "Analyse the data" |
| `investigation_id` | The same id for the whole investigation | A new id per call |
| `task_id` | A new, meaningful id per run, e.g. `T07-seis-challenge` | Reusing an id; the gateway renames duplicates |
| `analysis_mode` | `initial`, `follow_up`, `validation`, `challenge` or `reanalysis` | Leaving `initial` on reruns |
| `project_context` | Area, target interval, resource type, decision context | Pasting the full chat history |
| `available_evidence` | The evidence itself, or data-store references (see `geo-evidence-inventory`) | Sandbox paths; retyped large tables |
| `upstream_findings` | `'<global id> [<classification>] <statement>'`, one per item | Paraphrasing without ids; promoting interpretations to observations |
| `upstream_task_ids` | The tasks those findings came from | Omitting them |
| `known_constraints` | Assumptions that must not be made; fixed facts; methods required or excluded; limits on the work (a tool-call budget) | Leaving them unstated; turning an upstream caveat into a prohibition (see 1a) |
| `known_unknowns` | Missing data and open questions | Hiding gaps |
| `competing_interpretations` | Rival models the specialist must test | Only the favoured model |
| `required_outputs` | Concrete deliverables, each unconditional: "a depth map of the horizon at W1, converted with its time-depth table" | "A comprehensive analysis"; a conditional ("a depth map if a valid conversion exists; otherwise state cannot-determine"): the specialist takes the exit and tracks nothing. The chain behind a product is the specialist's job; the only acceptable "cannot" names a missing input. For a seismic volume with a well the standing set is: fault volume, body volume, a horizon at the well, the well tie, the depth map, and each feature's distance to the well. For a closure a prospect will use: `sg_build_model` on that horizon, then `sg_closure_spill` with it as `model` (without it the candidate stays a lead). Every analysis task also gets the standing line "figures for every product you publish that has a renderer" |
| `downstream_use` | Which task or decision consumes the result | Leaving it out, so the specialist cannot judge the precision needed |
| `previous_task_id`, `reason_for_reanalysis` | For every non-initial mode | Leaving them empty |
| `check_targets` | Validation or challenge: the claim ids tested (ledger; a plan's `settles`) | Prose |
| `instructions` | Leave empty: it replaces the default brief | Duplicating the output format, which the gateway already enforces |
| `specialist_session_id` | Leave it empty: the gateway continues each specialist's session for the investigation itself (section 2). Name one only to continue a session from an earlier investigation | Copying an id from your notes or ledger; using it for validation or challenge (refused) |
| `wait_seconds` | Omit it on a single call, so the gateway's default applies; `0` whenever another specialist starts in the same wave; never above the gateway's cap (its description states it: the harness's per-call timeout minus a margin) | Very short waits that force extra polling; a wait that outlasts the harness, which returns a server-execution-timeout and nothing else |

## 1a. Caveats are ceilings, not prohibitions

An upstream result that is uncalibrated, bounded, assumed or regional is still an input. Pass it with its caveat in the evidence `description` and the finding's own status in `upstream_findings`. The specialist computes with it, flags the assumption, and caps its claim (`partially_supported` at most) through the dependency ceiling. Do not write constraints such as "do not calculate stability without measured stress magnitudes" or "do not assess seal from an uncalibrated Vsh". They stop the specialist from using its tools and leave the question unanswered. Reserve `known_constraints` for facts that must not be assumed (an age, a location, a fluid) and for methods the user excludes.

Never invent a product reference. If a producer failed to publish a product, pass its findings in `upstream_findings` (with their ids and status), and name the missing product in `known_unknowns`. A fabricated `@agent/products/...` reference is a false evidence item. If the failure was a tool fault (a dataset that would not read, a format the tool rejected), retry that producer once, with the fault named in `reason_for_reanalysis`. Only then move on without its product.

## 1b. A specialist that returns partially_completed

A specialist that ran out of its own turn budget returns `partially_completed`. Its `conclusions` list the products it published, and its `recommended_followups` name the remaining work. Continue it as a `follow_up`, with the task as `previous_task_id` (the gateway continues its session) and the remaining items as the `geological_question`. Its records, products and context are still there. Do not start a new session for this. Do not treat the partial result as a failure to retry from scratch.

## 2. Session choice

The gateway keeps one session per specialist per investigation, and chooses it from `analysis_mode`. You never pass or copy a session id. Each result's `session_route` says what the gateway did.

| `analysis_mode` | Session | What the brief carries |
|---|---|---|
| `initial`, `follow_up`, `reanalysis` | the investigation's session for that specialist, opened by its first task | as usual; `previous_task_id` for every non-initial mode |
| `challenge` | a fresh session | the claims in `check_targets`; how they were made (method, inputs, `prov:` ids) in `upstream_findings`; not the reasoning |
| `validation` | a fresh session | blind: the question the claim answers and the data, not the earlier answer, and a different method, parameters or data from the run being checked |

- **Why one session.** It keeps the specialist's context: the files it loaded, what it tried, what failed. A follow-up, a correction or a reanalysis needs exactly that.
- **Why a check is fresh.** A model that re-examines its own answer, with that answer in its context, tends to defend it. So a check in the same session is not independent. A fresh session alone is not independent either. The specialist's tools return the earlier result for identical inputs and parameters, and a brief that states the earlier conclusion anchors the check. Independence comes from what the task sees and the method it uses.
- **`previous_task_id`** naming an earlier task of the same specialist continues that task's session. So a follow-up on a challenge's result stays with the challenge.
- **One task at a time.** The gateway queues a task for a busy session (`session_route`: "queued behind task …"). The task starts when the turn ahead ends. Collect it with `get_specialist_result` as usual. Do not resend it.

## 2a. Relaying the person's reaction to a result

When the person rejects or questions a result ("this is wrong", "coherence does a better job", "there are more faults than that"), the brief carries their words, not your theory of the fix. A remark briefed as a method ("use coherence as the principal input") can send run after run onto the wrong input. The real fault is then never looked for.

1. **Split the reaction into three parts**, each in its own field:
   - what they saw, word for word, in `reason_for_reanalysis` (e.g. "The person: 'This tells me the binary fault interpretation was faulty.'");
   - the outcome they want, in `objective` and `required_outputs` (e.g. "a fault volume, 0 = no fault, 1 = F1, ..., that captures the faults visible on the coherence slices");
   - a method only if they named one, quoted in `known_constraints` as theirs ("The person asked: 'use the coherence attribute'"). Never add a method of your own, and never write "rather than <method>" unless they said it.
2. **Ask when a reaction can be read two ways.** "Coherence does a better job" may describe what they see, or tell you how to compute. Before you spend a run, ask one question with both readings as options, e.g. "Find out why the fault volume misses what you see in coherence (Recommended), or build the fault volume from coherence instead?"
3. **Diagnose before rerunning.** Send a rejected result back to the same session, as a `follow_up` that asks why: "The person says <their words>. From your own tool calls and your skills' procedure, explain why the result misses it, then fix it." The specialist can read its own run; a fresh session cannot.
4. **Carry what was done, not a method to keep using.** Put earlier runs in `upstream_findings` and `project_context`, as what each did and produced, with `prov:` ids. Unless the person approved it, do not call an earlier run's approach "the workflow", "the guided volume" or the like. A guess restated as context becomes a premise.
5. **A number the person expects is a hypothesis to test, never a target.** "The person expects four faults: test whether the data support four and report what you find, with the persistence table." Never "can three be separated?", and never a count in `required_outputs`.
6. **Corrections of the same question are `follow_up` tasks** with `previous_task_id`. They run in the specialist's session automatically. A fresh session is only for validation or challenge (section 2).

## 3. Templates

**Initial task**
```json
{
  "investigation_id": "INV-2026-NorthBlock",
  "task_id": "T03-strat",
  "analysis_mode": "initial",
  "objective": "Establish the Main Sand correlation across W1-W3.",
  "geological_question": "Is the Main Sand in W1, W2 and W3 the same correlatable unit, and how does its thickness vary?",
  "project_context": "Hydrocarbon exploration, North Block. Target: Main Sand (Paleocene?). Decision: whether a Main Sand play is worth mapping on seismic.",
  "available_evidence": [{"id": "W1-W3-tops", "kind": "well_tops", "description": "...", "excerpt": "..."}],
  "upstream_findings": ["T02-wells/E4 [measurement] Main Sand net reservoir 38 m in W1 (GR cutoff 60 API)"],
  "upstream_task_ids": ["T02-wells"],
  "known_constraints": ["Do not assume the Paleocene age; no biostratigraphy was supplied."],
  "known_unknowns": ["Deviation surveys for W2 and W3"],
  "competing_interpretations": ["Single continuous sand sheet", "Two stacked lobes separated by a shale in W2"],
  "required_outputs": ["Correlation per well with confidence and alternatives", "Isochore trend"],
  "downstream_use": "Well ties and horizon identity for T04-seis."
}
```

**Validation** (fresh session, chosen by the gateway; blind)
- `analysis_mode`: `validation`
- `previous_task_id`: the task validated (required; the check runs in a fresh session)
- `geological_question`: the question the claim answers, asked afresh: "How many faults does the volume hold, and what are their strikes?", not "Is `T04-seis/C2` right?". You compare the two answers in review.
- `upstream_findings`: nothing from the task being validated **when the same specialist validates** (it would anchor on its own answer).
- **A different specialist validating** gets the products it must examine, without the conclusions drawn from them: the fault set or body set (`@seismic-interpreter-agent/products/...` from `seismic_publish_structure` / `seismic_publish_bodies`), the horizon maps, the well products. It has no seismic tools. A structural specialist handed only the SEG-Y path returns `insufficient_data`, and the validation is a wasted wave. Before you dispatch, check that `available_evidence` names the products.
- `known_constraints`: "Use a method, parameters or data different from `T04-seis`'s (`m:` id from its `source_reference`); do not rely on `T04-seis/E1`–`E3`."
- A check that repeats the same method on the same data gets the same numbers back from the tools' records. It is not a second line of evidence (`geo-result-review` section 4).

**Challenge** (fresh session, chosen by the gateway; the strongest alternative)
- `analysis_mode`: `challenge`
- `check_targets`: the claim under challenge; `upstream_findings`: how it was made (method, inputs, `prov:` ids)
- `geological_question`: "What is the strongest alternative to `T05-struct/C1`, what observations would discriminate between them, and do the supplied data contain any of those observations?"
- `known_constraints`: "Do not assume `T05-struct/C1` is correct."

**Reanalysis** (upstream changed; the investigation's session)
- `analysis_mode`: `reanalysis`
- `previous_task_id`: the superseded task
- `reason_for_reanalysis`: what changed and why, e.g. "`T03-strat` revised: the Main Sand in W2 is two lobes (`T08-strat/C1`)"
- `upstream_findings`: the changed findings

**Correction from the person** (the investigation's session; section 2a)
- `analysis_mode`: `follow_up`
- `previous_task_id`: the corrected task
- `reason_for_reanalysis`: the person's words, verbatim
- `geological_question`: "Why does `T01-seis`'s fault volume miss what the person sees ('<their words>')? Check your tool calls against your skills' procedure, then fix it."
- `known_constraints`: a method only if the person named one, quoted as theirs

**Follow-up** (the investigation's session)
- `analysis_mode`: `follow_up`
- `previous_task_id`: the earlier task
- `geological_question`: the specific clarification or extension

**Provenance question** — only after the user has said yes (see the system prompt, *Questions about a specialist's methods*). Narrow by construction: one question, one method or tool, no analysis.
```json
{
  "investigation_id": "INV-2026-NorthBlock",
  "task_id": "T09-seis-prov",
  "analysis_mode": "initial",
  "objective": "Report the methodological provenance of m:seis/edge-semblance as returned by seismic_method_provenance: the method's name, governing equation, assumptions, every recorded source with its DOI and locator, the verification status, and any recorded gaps. Do not compute anything and do not supply references the tool does not return.",
  "geological_question": "What published method, and what verification, lies behind the semblance attribute used in T03-seis/E2?",
  "required_outputs": ["the lookup tool's result, verbatim, as evidence with source_reference 'prov:<id of T03-seis/E2> | m:seis/edge-semblance'"],
  "known_constraints": ["about 3 tool calls"]
}
```
Route by the method id's domain (`m:seis/` → `seismic_interpretation`, `m:petro/` → `wells_petrophysics`, and so on). If nothing has run yet, route by subject. An `m:core/` method belongs to the shared tool layer, and any specialist can answer for it: ask the one whose result raised the question.

## 4. Running jobs

- **Starting several specialists.** This chat makes one tool call per step. A call waits up to `wait_seconds` (default the gateway's cap) before the next can start. So start independent specialists with `wait_seconds: 0`, one after another, and they run in parallel. Then collect each result with `get_specialist_result`.
- **No result at all** (your call or poll errors out: "other side closed", "read ECONNRESET", "Failed to process successful response", a timeout) is the connection, not the specialist. The job runs on its own thread, and the job id stays valid. Poll it again. Never re-dispatch, and never record it as a failed specialist: that is how a finished wave gets thrown away.
- **A result reporting `execution_status: error`** ("the specialist's turn failed") is a dead turn. Retry with a new task id, as the `next_action` says. If the turn had run long, narrow the scope. Record it as an execution failure, not a scientific one.
- **Jobs still running.** If a result says `running`, call `get_specialist_result(job_id=...)` with `wait_seconds = min(cap, remaining turn time - 120)` (see `geo-investigation-planning`, section 7). The call returns as soon as the specialist finishes, so a long wait costs nothing. A wait that outlasts the turn costs the checkpoint. If a poll returns `running` again, poll again, and update the ledger between polls. A poll that fails is the connection (see **No result at all** above).
- **Collecting a wave.** Each `get_specialist_result` returns one result in its own step, so TrueForge's single-result limit (about 6,000 tokens) applies. **Trimmed results** below covers what exceeds it.
- **Trimmed results.** Results are size-limited. When `omitted` lists sections you need, call `get_specialist_result(job_id=..., sections=["claims"], offset=N)`, as `next_action` suggests. If TrueForge replaced a result with a "too big" preview, re-fetch it the same way.
- **Unneeded work.** Cancel jobs you no longer need with `cancel_specialist_job`. TrueForge asks the person to approve each cancel first.
- **Lost ids.** `list_specialist_jobs` recovers job ids, e.g. after context compaction.

## 5. Rejections and warnings

| Message | Meaning | Action |
|---|---|---|
| "Arguments repaired by the gateway: …" | Your call shape was fixed | Use the canonical shape next time |
| "task_id … was already used" | Duplicate id | Use a new id per run |
| "Queued behind task …" | The specialist's session is busy with an earlier task | Nothing: it starts when that turn ends; collect it with `get_specialist_result` |
| "Session … is still busy" | A session named from outside is running or paused | Wait for, or cancel, the earlier job |
| "… is not a … session of investigation …" | A `specialist_session_id` that is not this investigation's session (usually a mistyped copy) | Call again without it: the gateway continues the right one |
| "A validation/challenge task runs in a fresh … session" | A session id passed with an independent mode | Omit it, or use `follow_up` if you meant to continue that work |
| "belongs to …, not …" | Wrong `specialist_session_id` for this tool | Omit it |
| "No saved agent named …" | A configuration problem | Report it to the user; do not substitute your own analysis |
| "… jobs are already running (limit …)" | The concurrency cap was reached | Collect or cancel jobs first |
| "Unresolved evidence reference" | The specialist cited an id that does not exist | Treat that link as unsupported; see `geo-result-review` |

## 6. Retries

Retry an execution failure (`error`, `timeout`, `protocol_error`) at most once, with a new `task_id` and a stated reason. If time was the problem, narrow the scope. If it fails again, record the analysis as not executed and tell the user. Never fill the gap yourself.

<!-- end of skill: geo-specialist-tasking -->
