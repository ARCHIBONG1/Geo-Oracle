---
name: lit-investigation-report
description: How to write and render the formal investigation report from the frozen investigation bundle. Use when Geo Oracle's brief asks you to draft, render or revise the investigation report (a task named R1-lit-report, R2-lit-report or R3-lit-report). It covers how to read the bundle with lit_read_bundle, the draft JSON, citations and hidden tags, and the ASD-STE100 writing rules. It also covers the narrow rule for conclusions, how to render with lit_render_report, and what to return.
---

# The investigation report

You write the formal report of an investigation that is closed. Geo Oracle and the specialists did the work. Your job is to tell it as a clear field exploration report that a geoscientist can read from start to end. You do not do new analysis, you do not search for new literature, and you do not change a conclusion.

The report reads clean: no ids in the text. Each paragraph carries hidden tags that name the ledger items it rests on. The renderer prints the ledger, unchanged, as the appendix. A reader can then check every paragraph against the ledger.

## 1. Read the bundle

The gateway froze the investigation into a bundle when the person asked for the report. Read it with `lit_read_bundle`, one part at a time:

1. `part="overview"` gives:
   - the objective, the HOTL setting and the executive summary Geo Oracle gave the person;
   - one line per task;
   - the figures, with the agent that made each, its caption and the claims it supports;
   - the data, the methods that ran, and what is not available;
   - `do_not_rely_on`: the superseded tasks and the evidence the gateway set aside.
2. `part="task"` with each `task_id` from the overview: the claims with their status, the evidence, the methods statements, the contradictions, the limitations and the conclusions.
3. `part="literature"`: the literature evidence you may cite.
4. `part="ledger"`: Geo Oracle's ledger. It is the reference for every status and every conclusion.

Use only the bundle. Do not use your general knowledge as a finding. Do not rest a result or a conclusion on a superseded task or on set-aside evidence. The overview and each task view mark them. Do not carry their methods statements.

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
  "literature_references": [{"evidence_id": "T03-lit/E1"}]
}
```

- **A block** is a paragraph `{text, tags}` or a figure `{figure, caption, tags}`. The figure name is one from the overview's `figures`. Tag a figure with the claims that its specialist marked it as supporting (`supports`). Start its caption from the specialist's caption.
- **Tags** are ledger ids:
  - evidence and claim ids (`T03-seis/E2`, `T05-struct/C1`);
  - task ids;
  - Geo Oracle's `H#`, `X#`, `U#`, `Q#` and `F#` (a figure id in the ledger's Figures table).

  Tags never print. Each paragraph in the Executive summary, in Results and Discussion and in Conclusion and Limitations needs at least one tag.
- **Citations in the text** are markers only. A method is `[[m:<domain>/<name>]]`. The method must have run in this investigation (the overview's `methods_run`). A published source is `[[lit:<evidence id>]]`, with an entry in `literature_references`. The renderer turns markers into author-year citations from the registry and from your entries.
- **No other ids in the text.** No `prov:` ids, no method ids outside a marker, no task, evidence, claim or job ids, no product paths. The renderer refuses a draft that shows one.
- **Complete and printable.** The renderer also refuses a draft that has no title, no executive summary, an empty section or an empty paragraph. It also refuses a draft with a character that no report font can print. Write such a character in words or in plain characters.
- **Literature references** come only from the literature evidence in the bundle, each source once, as `{evidence_id}`. The renderer prints each reference from what that evidence recorded: author, year, title, venue and DOI. It never prints what you type. A `doi`, `authors`, `year` or `title` that you add must agree with the record. Cite only a resolved source: its `source_reference` has `doi:`, `openalex:`, `url:` or `project:`. Describe an `UNVERIFIED` lead in the text as an unverified lead, without a citation. A source whose record names no author and year cannot print as a reference, unless it is a project document. Describe such a source in the text without a citation. Do not add a source from memory. If a source was read below full text (abstract, metadata or secondary), its access level prints beside the reference.

## 3. What goes in each section

| Section | Content |
|---|---|
| Executive summary | Geo Oracle's executive summary, in ASD-STE100. Keep every conclusion and its strength. Add nothing. |
| Introduction | The objective, the decision the work supports, the scope, and the data the person supplied. |
| Geology of the study area | The regional setting from the literature evidence and from the regional, stratigraphy and sedimentology findings. |
| Data and Methodology | The data (from the overview's `data`), then each specialist's methods statements. |
| Results and Discussion | The claims and evidence by subject, with figures beside the claims they support: at least one figure for each main conclusion that has one, and every bundle figure that supports a claim the report states. Contradictions and alternatives stay in. |
| Conclusion and Limitations | Geo Oracle's conclusions, the limitations, the material uncertainties and the next steps from the ledger. |

**Methods statements.** Each specialist wrote a short statement of what it ran. Carry each statement into Data and Methodology with light edits only. Keep every number, and cite each method it names with its marker. The renderer lists a lost method or number as drift, and Geo Oracle reviews the drift.

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

For Geo Oracle's review, it also warns on:
- a number in a summary, result or conclusion paragraph that is not in the items it is tagged with;
- strong wording ("proven", "confirmed", "commercial", "will produce");
- a ledger hypothesis, contradiction or uncertainty that no paragraph is tagged with;
- an executive summary that departs from Geo Oracle's or changes a negation;
- a figure tagged with claims its specialist did not mark;
- bundle figures the report does not use;
- a Results section with no figure.

Methods drift comes first in the warnings, then these, then the style warnings.

## 6. Render

1. Call `lit_render_report(investigation_id, draft_json, review_round, wait_seconds=240)`. Use the round your brief gives (1 for the first draft). The renderer refuses any other round.
2. **Rejected:** the message lists every error. Correct all of them and call again with the same round.
3. **Running:** collect it with `get_literature_job_result`.
4. **`release_checks: failed`:** the PDF cannot go to the person and has no download link. Correct the problems and render again with the same round.
5. **LaTeX could not compile:** the message gives the first LaTeX error and its line. The renderer escapes a special character in a reference field for you. Correct what the error names.

## 7. Return

Put these in `conclusions`:

- `Report (round <n>): <download_url>` (if the release checks passed; if not, the problems that remain)
- the release checks, the page count, and the number of style and drift warnings;
- for a revision, one line for each of Geo Oracle's review notes: what you changed, or why you did not change it.

Do not put the PDF in a `sandbox_artifacts` block: it is not in your sandbox. Record the render's `prov:` id in `source_reference` as usual.

## 8. Revisions

A revision brief carries Geo Oracle's review notes as evidence `R1`, `R2` and so on. Each note is a contradiction between your draft and the analysis. For each note, read the bundle again and correct the draft. Then render with the round number that the brief gives. There are three rounds at most, so correct every note in one round.

## Methods statements

Write one methods statement for this task, in 1 to 3 sentences. Say that you drafted the report from the frozen bundle and rendered it. Its `method_ids` are `m:lit/render-report` and `m:lit/investigation-report`, and its `provenance_ids` hold the `prov:` id of the render you return. If no render succeeded, there is no `prov:` id. Then leave `method_ids` and `provenance_ids` empty, and say in the text that the render failed. Do not name `lit_read_bundle` in it: a read of the bundle records no computation.

<!-- end of skill: lit-investigation-report -->
