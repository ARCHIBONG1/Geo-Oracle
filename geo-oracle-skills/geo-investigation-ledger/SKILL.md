---
name: geo-investigation-ledger
description: Format and update rules for Geo Oracle's investigation ledger (investigation/ledger.md in the sandbox). The ledger holds the frame, evidence inventory, specialist tasks and job ids, the claims relied on, hypotheses and their status, contradictions, uncertainties, dependencies and open questions. Use at the start of every investigation. Also load it in these cases — after every reviewed specialist result; whenever an interpretation changes status; at every checkpoint or end of turn; and immediately after context compaction, or when you resume an investigation in a new turn.
---

# Investigation ledger

Long investigations outgrow the context window. Compaction removes ids and statuses from the summary, and turns end at a time limit. The ledger keeps the reasoning auditable and resumable. It is the source of truth for what ran, what was found, and how each conclusion depends on the evidence.

## Rules

- **Before any inline question**, fill Pending question and save the ledger. A question blocks the turn until the person answers it. If the person walks away, the harness cuts the turn at its limit with no message. Then this block is the only thing that makes the work recoverable. Write it first, ask second. Clear it when you file the answer.
- **The investigation id** is new for every new objective, with the date and time in it (`INV-dome-co2-20261006T0504`). Use it for every task and every gateway call of that investigation. If you reuse a previous id, this investigation inherits that id's counters and its approved plan.
- **Location.** Keep the file at `investigation/ledger.md`. If one session holds several investigations, use `investigation/<investigation_id>/ledger.md`.
- **When to update.** Update after every reviewed result, before calling the next specialist.
- **Global ids.** Use them exactly as returned, e.g. `T03-seis/E2` or `T05-struct/C1`. Your own items use `H#` (hypotheses), `X#` (contradictions), `U#` (uncertainties) and `Q#` (open questions). Each row of the Figures table uses `F#`.
- **Sources.** Every row needs a source. Record only what exists.
- **Figures.** Record every figure a specialist returns (its `figures` entries, or its `Figure:` conclusion lines), with the claims it supports. Otherwise the URLs do not survive compaction. The final answer places figures by these rows.
- **Size.** Keep it compact: one-line statements, referenced by id. Do not paste specialist output. The gateway can fetch any result again by `job_id`.
- **History.** Mark items as superseded. Do not delete them. The history is part of the audit trail.
- **Rivals.** Record every alternative a specialist names, in the Hypotheses table or beside its claim, with its test and result. It stays until a test or a checkpoint resolves it.
- **Open outputs.** List each required output a specialist left open by its token (`T05-struct#2`) in *Next steps* until it is delivered or dismissed.
- **The close.** At the close, add the *Final status* block at the end (format in skill `geo-investigation-report`). Save, and give the ledger as a download. After that, the ledger is frozen: the formal report prints it unchanged as its appendix, with its hash. Edit it again only when the person asks for more work and the investigation opens again.
- **Resuming.** When you resume after a new turn or compaction:
  1. Read the ledger.
  2. For any task marked `running`, call `get_specialist_result(job_id)`.
  3. Continue from *Next steps*.

## Template

```markdown
# Investigation INV-... — <short title>
Updated: <time> | Turn: <n>

## Frame
Objective: ...
Decision supported: ...
Resource type: ...
Area / intervals: ...
Scope: screening | detailed

## Evidence inventory
| ID | Type | Content actually present | Availability | Frames / units | Limitations |
|----|------|--------------------------|--------------|----------------|-------------|

## Tasks
| Task ID | Tool | Question (short) | Mode | Depends on | Job ID | Session route | Exec status | Sci status | Disposition |
|---------|------|------------------|------|------------|--------|---------------|-------------|------------|-------------|

**Session route** is the result's `session_route` in a few words ("continued T01's", "fresh: validation", "queued behind T02"), never a session id. The gateway keeps each specialist's session and continues it itself. If you copy a session id into the ledger and back into a call, one investigation can come to hold two seismic sessions.

## Claims relied on
| ID | Class | Statement (1 line) | Source | Supported by | Independent lines | Status |
|----|-------|--------------------|--------|--------------|-------------------|--------|

The **Source** cell carries the specialist's tokens exactly as returned: the `prov:` id of the computation, and every `m:<domain>/<name>` method id from its `source_reference`. For example: `T03-seis/E2 · prov:2b551b72cc64912c · m:seis/edge-semblance`. The tokens are self-delimiting, so the risk specialist (which parses this ledger) can read them with any punctuation. Copy them. Never compose a method id or a source yourself.

## Hypotheses
| H# | Hypothesis | Supporting | Contradicting | Key assumptions | Discriminating test | Status |
|----|------------|------------|---------------|-----------------|---------------------|--------|

## Contradictions
| X# | Between (ids) | Type | Possible explanations | Discriminating evidence | Task | Status |
|----|---------------|------|-----------------------|-------------------------|------|--------|

## Uncertainties
| U# | Source | Affects (ids) | Consequence | Materiality (from risk_uncertainty's flip tests; blank until then) | Reducible? | How / by whom |
|----|--------|---------------|-------------|----------|------------|---------------|

## Figures
| ID | Caption | URL | From task | Supports (claim ids) | Interactive |
|----|---------|-----|-----------|----------------------|-------------|

## Dependencies
- H1 (Main Sand structural play) ← T05-struct/C1 (closure at H3) ← T04-seis/E2 (H3 pick) ← T03-strat/C1 (tie to Main Sand top)

## Open questions
- Q1 ...

## Pending question
*Written immediately BEFORE any inline question, cleared when the answer is filed. This is what a new turn resumes from if the person walks away and the turn is cut.*

| Field | Content |
|-------|---------|
| Asked at | |
| Wave | |
| Work line | n of N specialist runs used |
| Question | the question exactly as put |
| Options | the choices offered |
| Summary as put | done · why · contribution · next, as the person saw it |
| Plan awaiting decision | the specialists proposed |
| Jobs still running | job ids, which keep running and stay collectable |

## Next steps
- ...

## Change log
- <time> T08-strat reanalysis: W2 Main Sand = two lobes → T04-seis/E2 and T05-struct/C1 need review.
```

## Hypothesis status

Hypothesis status is your judgement after review. A specialist's own claim status (`proposed`, `supported`, …) is an input to that judgement. It is not the verdict.

| Status | Use when |
|---|---|
| untested | Proposed, and no evidence has been evaluated yet |
| weakly constrained | Consistent with some evidence, but alternatives fit equally well |
| partially constrained | Some elements are supported; others are untested or unknown |
| supported | At least one line of positive evidence, and no unresolved material contradiction |
| strongly supported | Two or more independent lines of positive evidence, and no unresolved material contradiction |
| contradicted | Material evidence conflicts with it |
| unresolved | Conflicting evidence, and nothing currently available can discriminate |
| superseded | Replaced by a revised hypothesis; keep it for the record |

Rules for status:
- The absence of contradicting evidence never upgrades a status.
- New material contradicting evidence downgrades the status immediately.
- Count independent lines of evidence, not agreeing agents (see `geo-result-review`).

## Propagating a change

When a claim or hypothesis changes status, or is superseded:
1. Find everything that depends on it in *Dependencies* and *Claims relied on*.
2. Mark those items `needs review`, and note the change in the *Change log*.
3. For each dependent item that is material to the objective, schedule a reanalysis task, or record why the task is not needed.
4. Never let a downstream conclusion keep a status that its upstream support has lost.

## Checkpoint block

At the end of a turn, or when stopping, make sure *Next steps* contains:
- the job ids that are still running;
- the questions that are still open;
- the next planned tasks;
- anything waiting on the user.

Then give the user a short summary drawn from the ledger.

<!-- end of skill: geo-investigation-ledger -->
