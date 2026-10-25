---
name: lit-investigation-report
description: How to write and render the formal investigation report from the frozen investigation bundle. Use when Geo Oracle's brief asks you to draft, render or revise the investigation report (a task named R1-lit-report, R2-lit-report or R3-lit-report). Covers reading the bundle with lit_read_bundle, the draft JSON, citations and hidden tags, the ASD-STE100 writing rules, the narrow rule for conclusions, rendering with lit_render_report, and what to return.
---

# The investigation report

You write the formal report of an investigation that is closed. Geo Oracle and the specialists did the work. Your job is to tell it as a clear field exploration report that a geoscientist can read from start to end. You do not do new analysis, you do not search for new literature, and you do not change a conclusion.

The report reads clean: no ids in the text. Each paragraph carries hidden tags that name the ledger items it rests on. The renderer prints the ledger, unchanged, as the appendix. A reader can then check every paragraph against the ledger.

## 1. Read the bundle

The gateway froze the investigation into a bundle when the person asked for the report. Read it with `lit_read_bundle`, one part at a time:

1. `part="overview"`: the objective, the HOTL setting, the executive summary Geo Oracle gave the person, one line per task, the figures, the data and the methods that ran.
2. `part="task"` with each `task_id` from the overview: the claims with their status, the evidence, the methods statements, the contradictions, the limitations and the conclusions.
3. `part="literature"`: the literature evidence you may cite.
4. `part="ledger"`: Geo Oracle's ledger. It is the reference for every status and every conclusion.

Use only the bundle. Do not use your general knowledge as a finding.

## 2. The draft

The draft is one JSON object, sent to `lit_render_report` as a string in `draft_json`:

```json
{
  "title": "Short title of the investigation",
  "executive_summary": [{"text": "...", "tags": ["H1"]}],
  "sections": {
    "introduction": [{"text": "...", "tags": []}],
    "geology": [{"text": "... [[lit:T03-lit/E1]].", "tags": ["T03-lit/E1"]}],
    "methodology": [{"text": "The specialist calculated ... [[m:petro/porosity-permeability-transform]].", "tags": ["T01-wells/E5"]}],
    "results_discussion": [{"text": "...", "tags": ["T01-wells/E5"]},
                           {"figure": "closure_map.png", "caption": "...", "tags": ["T05-struct/C1"]}],
    "conclusion_limitations": [{"text": "...", "tags": ["H1", "U2"]}]
  },
  "literature_references": [{"evidence_id": "T03-lit/E1", "doi": "10....", "authors": ["Surname, Given"],
                             "title": "...", "year": "2001", "venue": "..."}]
}
```

- **A block** is a paragraph `{text, tags}` or a figure `{figure, caption, tags}`. The figure name is one from the overview's `figures`.
- **Tags** are ledger ids: evidence and claim ids (`T03-seis/E2`, `T05-struct/C1`), task ids, and Geo Oracle's `H#`, `X#`, `U#`, `Q#`. They never print. Each paragraph in Results and Discussion and in Conclusion and Limitations needs at least one tag.
- **Citations in the text** are markers only. A method is `[[m:<domain>/<name>]]`, and the method must have run in this investigation (the overview's `methods_run`). A published source is `[[lit:<evidence id>]]`, with an entry in `literature_references`. The renderer turns markers into author-year citations from the registry and from your entries.
- **No other ids in the text.** No `prov:` ids, no method ids outside a marker, no task, evidence, claim or job ids, no product paths. The renderer refuses a draft that shows one.
- **Literature references** come only from the literature evidence in the bundle. The DOI must be the one that evidence records. Give authors, title and year. Do not add a source from memory.

## 3. What goes in each section

| Section | Content |
|---|---|
| Executive summary | Geo Oracle's executive summary, in ASD-STE100. Keep every conclusion and its strength. Add nothing. |
| Introduction | The objective, the decision the work supports, the scope, and the data the person supplied. |
| Geology of the study area | The regional setting from the literature evidence and from the regional, stratigraphy and sedimentology findings. |
| Data and Methodology | The data (from the overview's `data`), then each specialist's methods statements. |
| Results and Discussion | The claims and evidence by subject, with figures beside the claims they support. Contradictions and alternatives stay in. |
| Conclusion and Limitations | Geo Oracle's conclusions, the limitations, the material uncertainties and the next steps from the ledger. |

**Methods statements.** Each specialist wrote a short statement of what it ran. Carry each statement into Data and Methodology with light edits only. Keep every number and cite each method it names with its marker. The renderer lists a lost method or number as drift, and Geo Oracle reviews the drift.

**Numbers.** Copy each number and unit exactly as the ledger or the findings give it. Do not round, convert or average.

## 4. The rule for conclusions

You restate Geo Oracle's conclusions. You never raise, lower or add one.

- Each conclusion has the strength its ledger status gives it. "Weakly constrained" is not "supported". "Unresolved" is not "likely".
- A hypothesis the ledger keeps open stays open. A contradiction the ledger preserves stays in the report.
- A limitation in the ledger or the findings stays with the conclusion it limits.
- If you think the ledger is wrong, do not correct it in the report. Write the concern in your `limitations` for Geo Oracle.

## 5. ASD-STE100

Write all text in ASD-STE100 (Simplified Technical English):

- A sentence has 25 words or fewer. A paragraph has six sentences or fewer.
- Use the active voice: "The specialist measured the porosity", not "The porosity was measured".
- Use one topic in each sentence.
- Do not use a word ending in -ing as a noun ("the mapping" becomes "the map" or "to map").
- Use simple, common words, with one meaning for each word. Use the technical names of the geology as they are.

The renderer counts sentence and paragraph length and finds passive voice and -ing nouns. These come back as warnings. Correct them before the next round.

## 6. Render

1. Call `lit_render_report(investigation_id, draft_json, review_round, wait_seconds=240)`. Use the round your brief gives (1 for the first draft).
2. **Rejected:** the message lists every error. Correct all of them and call again with the same round.
3. **Running:** collect it with `get_literature_job_result`.
4. **`release_checks: failed`:** the PDF cannot go to the person. Correct the problems and render again with the same round.

## 7. Return

Put these in `conclusions`:

- `Report (round <n>): <download_url>`
- the release checks, the page count, and the number of style and drift warnings;
- for a revision, one line for each of Geo Oracle's review notes: what you changed, or why you did not change it.

Do not put the PDF in a `sandbox_artifacts` block: it is not in your sandbox. Record the render's `prov:` id in `source_reference` as usual.

## 8. Revisions

A revision brief carries Geo Oracle's review notes as evidence `R1`, `R2` and so on. Each note is a contradiction between your draft and the analysis. Read the bundle again for each note, correct the draft, and render with the round number the brief gives. There are three rounds at most, so correct every note in one round.

## Methods statements

This task has no methods statement of its own: the report reports the specialists' statements. Leave `methods_statements` empty.

<!-- end of skill: lit-investigation-report -->
