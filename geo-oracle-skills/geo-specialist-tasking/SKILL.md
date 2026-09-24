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
| `known_constraints` | Assumptions that must not be made; fixed facts; methods required or excluded | Leaving them unstated |
| `known_unknowns` | Missing data and open questions | Hiding gaps |
| `competing_interpretations` | Rival models the specialist must test | Only the favoured model |
| `required_outputs` | Concrete deliverables | "A comprehensive analysis" |
| `downstream_use` | Which task or decision consumes the result | Leaving it out, so the specialist cannot judge the precision needed |
| `previous_task_id`, `reason_for_reanalysis` | For every non-initial mode | Leaving them empty |
| `instructions` | Leave empty: it replaces the default brief | Duplicating the output format, which the gateway already enforces |
| `specialist_session_id` | Only to continue the same specialist's thread | Using it for validation or challenge |
| `wait_seconds` | Default for single calls; `0` for wide fan-outs | Very short waits that force extra polling |

## 2. Session choice

- **Continue the session** (pass `specialist_session_id`) for clarifications, missing items, format repairs, or a narrow extension of the same analysis. The specialist keeps its context, which saves tokens and time.
- **Start a new session** (omit `specialist_session_id`) for `validation` and `challenge`. A specialist re-examining its own reasoning in the same conversation is anchored by it; a fresh session gives a more independent test. Pass the claims under test as `upstream_findings`.
- **Reanalysis after an upstream change** usually uses a new session with the changed inputs, plus `previous_task_id` pointing to the superseded run.

A session runs one task at a time. The gateway refuses a follow-up while that session's previous job is still running or waiting for an action.

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

**Validation** (new session; test with independent evidence)
- `analysis_mode`: `validation`
- `previous_task_id`: the task being validated
- `geological_question`: "Does independent evidence (X) support or contradict claim `T04-seis/C2`?"
- `upstream_findings`: the claim under test, plus the evidence it used
- `known_constraints`: "Do not rely on `T04-seis/E1`–`E3`; they are the evidence being validated."

**Challenge** (new session; the strongest alternative)
- `analysis_mode`: `challenge`
- `geological_question`: "What is the strongest alternative to `T05-struct/C1`, what observations would discriminate between them, and do the supplied data contain any of those observations?"
- `known_constraints`: "Do not assume `T05-struct/C1` is correct."

**Reanalysis** (upstream changed)
- `analysis_mode`: `reanalysis`
- `previous_task_id`: the superseded task
- `reason_for_reanalysis`: what changed and why, e.g. "`T03-strat` revised: the Main Sand in W2 is two lobes (`T08-strat/C1`)"
- `upstream_findings`: the changed findings

**Follow-up** (same session)
- `specialist_session_id`: from the earlier result
- `analysis_mode`: `follow_up`
- `previous_task_id`: the earlier task
- `geological_question`: the specific clarification or extension

## 4. Running jobs

- **Parallel calls.** Put independent calls in one step. TrueForge runs them concurrently, and each waits up to `wait_seconds` (default about 3 minutes).
- **Jobs still running.** If a result says `running`, call `get_specialist_result(job_id=...)`. Each call waits a few minutes. Work on something else useful, such as the ledger, between polls if you can.
- **Wide fan-outs.** For more than about three jobs in a step, start them with `wait_seconds: 0`, then collect each with `get_specialist_result`. This keeps each step's combined tool output within TrueForge's size limits.
- **Trimmed results.** Results are size-limited. When `omitted` lists sections you need, call `get_specialist_result(job_id=..., sections=["claims"], offset=N)` as `next_action` suggests. If TrueForge replaced a result with a "too big" preview, re-fetch it the same way.
- **Unneeded work.** Cancel jobs you no longer need with `cancel_specialist_job`.
- **Lost ids.** `list_specialist_jobs` recovers job ids, e.g. after context compaction.

## 5. Rejections and warnings

| Message | Meaning | Action |
|---|---|---|
| "Arguments repaired by the gateway: …" | Your call shape was fixed | Use the canonical shape next time |
| "task_id … was already used" | Duplicate id | Use a new id per run |
| "Session … is still busy" | A follow-up to a running or paused session | Wait for, or cancel, the earlier job |
| "belongs to …, not …" | Wrong `specialist_session_id` for this tool | Use a session from the same specialist, or none |
| "No saved agent named …" | A configuration problem | Report it to the user; do not substitute your own analysis |
| "… jobs are already running (limit …)" | The concurrency cap was reached | Collect or cancel jobs first |
| "Unresolved evidence reference" | The specialist cited an id that does not exist | Treat that link as unsupported; see `geo-result-review` |

## 6. Retries

Retry an execution failure (`error`, `timeout`, `protocol_error`) at most once, with a new `task_id` and a stated reason. Where time was the problem, narrow the scope. If it fails again, record the analysis as not executed and tell the user. Never fill the gap yourself.
