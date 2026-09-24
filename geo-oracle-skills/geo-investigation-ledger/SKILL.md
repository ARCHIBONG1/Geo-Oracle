---
name: geo-investigation-ledger
description: Format and update rules for Geo Oracle's investigation ledger (investigation/ledger.md in the sandbox). The ledger holds the frame, evidence inventory, specialist tasks and job ids, the claims relied on, hypotheses and their status, contradictions, uncertainties, dependencies and open questions. Use at the start of every investigation, after every reviewed specialist result, whenever an interpretation changes status, at every checkpoint or end of turn, and immediately after context compaction or when resuming an investigation in a new turn.
---

# Investigation ledger

Long investigations outgrow the context window. Compaction summarises away ids and statuses, and turns end at a time limit. The ledger keeps the reasoning auditable and resumable. It is the source of truth for what was run, what was found, and how each conclusion depends on the evidence.

## Rules

- **Location.** Keep the file at `investigation/ledger.md`. If one session holds several investigations, use `investigation/<investigation_id>/ledger.md`.
- **When to update.** Update after every reviewed result, before calling the next specialist.
- **Global ids.** Use them exactly as returned, e.g. `T03-seis/E2` or `T05-struct/C1`. Your own items use `H#` (hypotheses), `X#` (contradictions), `U#` (uncertainties) and `Q#` (open questions).
- **Sources.** Every row needs a source. Record only what exists.
- **Size.** Keep it compact: one-line statements, referenced by id. Do not paste specialist output; the gateway can re-fetch any result by `job_id`.
- **History.** Mark items as superseded; do not delete them. The history is part of the audit trail.
- **Resuming.** When resuming after a new turn or compaction:
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
| Task ID | Tool | Question (short) | Mode | Depends on | Job ID | Session | Exec status | Sci status | Disposition |
|---------|------|------------------|------|------------|--------|---------|-------------|------------|-------------|

## Claims relied on
| ID | Class | Statement (1 line) | Source | Supported by | Independent lines | Status |
|----|-------|--------------------|--------|--------------|-------------------|--------|

## Hypotheses
| H# | Hypothesis | Supporting | Contradicting | Key assumptions | Discriminating test | Status |
|----|------------|------------|---------------|-----------------|---------------------|--------|

## Contradictions
| X# | Between (ids) | Type | Possible explanations | Discriminating evidence | Task | Status |
|----|---------------|------|-----------------------|-------------------------|------|--------|

## Uncertainties
| U# | Source | Affects (ids) | Consequence | Severity | Reducible? | How / by whom |
|----|--------|---------------|-------------|----------|------------|---------------|

## Dependencies
- H1 (Main Sand structural play) ← T05-struct/C1 (closure at H3) ← T04-seis/E2 (H3 pick) ← T03-strat/C1 (tie to Main Sand top)

## Open questions
- Q1 ...

## Next steps
- ...

## Change log
- <time> T08-strat reanalysis: W2 Main Sand = two lobes → T04-seis/E2 and T05-struct/C1 need review.
```

## Hypothesis status

Hypothesis status is your judgement after review. A specialist's own claim status (`proposed`, `supported`, …) is an input to that judgement, not the verdict.

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
3. For each dependent item that is material to the objective, schedule a reanalysis task, or record why it is not needed.
4. Never let a downstream conclusion keep a status that its upstream support has lost.

## Checkpoint block

At the end of a turn, or when stopping, make sure *Next steps* contains:
- the job ids that are still running;
- the questions that are still open;
- the next planned tasks;
- anything waiting on the user.

Then give the user a short summary drawn from the ledger.
