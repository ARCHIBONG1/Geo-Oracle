---
name: lit-investigation-report
description: How to read the frozen investigation bundle and render the formal investigation report. Use when Geo Oracle's brief asks you to draft, render or revise the investigation report (a task named R1-lit-report, R2-lit-report or R3-lit-report). It covers lit_read_bundle, the draft JSON with its headings, figures, tables, labels, citations and hidden tags, the narrow rule for conclusions, how to render with lit_render_report, and what to return. The content and style of each section are in lit-report-writing, which you read next.
---

# The investigation report

You write the formal report of an investigation that is closed. Geo Oracle and the specialists did the work. Your job is to tell it as a clear scientific report that the people who decide on the investigation can read from start to end. You do not do new analysis, you do not search for new literature, and you do not change a conclusion. Read `lit-report-writing` after this skill: it says what each section holds and how to write it.

The report reads clean: no ids in the text. Each paragraph carries hidden tags that name the ledger items it rests on. The renderer prints the ledger, unchanged, as the appendix. A reader can then check every paragraph against the ledger.

## 1. Read the bundle

The gateway froze the investigation into a bundle when the person asked for the report. Read it with `lit_read_bundle`, one part at a time:

1. `part="overview"` gives:
   - the objective, the HOTL setting and the executive summary Geo Oracle gave the person;
   - one line per task;
   - the figures, with the agent that made each, its caption and the claims it supports;
   - the data, the methods that ran, and what is not available;
   - `do_not_rely_on`: the superseded tasks and the evidence the gateway set aside.
2. `part="task"` with each `task_id` from the overview: the claims with their status and their recorded alternatives, the evidence, the methods statements, the contradictions, the limitations and the conclusions.
3. `part="literature"`: the literature evidence you may cite.
4. `part="ledger"`: Geo Oracle's ledger. It is the reference for every status and every conclusion.

A long part comes in pages. `part="figures"`, `part="data"` and `part="methods"` hold what the overview names by count. When a reply gives `<list>_next_offset` (or `next_offset` for the ledger), call the same part again with `offset` set to it. Read every page before you draft.

Use only the bundle. Do not use your general knowledge as a finding. One exception: the background of the executive summary may hold one or two sentences of general geoscience context, with no number and no claim about this area (`lit-report-writing`, section 1). Do not rest a result or a conclusion on a superseded task or on set-aside evidence. The overview and each task view mark them. Do not carry their methods statements.

## 2. The draft

The draft is one JSON object, sent to `lit_render_report` as a string in `draft_json`:

```json
{
  "title": "Short title of the investigation",
  "keywords": ["seismic interpretation", "well tie"],
  "executive_summary": [{"part": "background", "text": "...", "tags": ["frame"]},
                        {"part": "results", "text": "...", "tags": ["H1", "T05-struct/C1"]}],
  "sections": {
    "introduction": [{"text": "... as [[tab:data]] lists ...", "tags": []}],
    "geology": [{"text": "... [[lit:T03-lit/E1]].", "tags": ["T03-lit/E1"]}],
    "methodology": [{"heading": "Well products"},
                    {"text": "The petrophysics specialist calculated ... [[m:petro/porosity-permeability-transform]].", "tags": ["T01-wells/E5"]}],
    "results_discussion": [{"heading": "The closure"},
                           {"text": "... as [[fig:closure]] and [[tab:closures]] show.", "tags": ["T05-struct/C1"]},
                           {"figure": "closure_map.png", "caption": "...", "label": "closure", "tags": ["T05-struct/C1"]},
                           {"table": {"caption": "...", "columns": ["Closure", "Area"], "rows": [["A", "2.1 km2"]], "label": "closures"}, "tags": ["T05-struct/C1"]}],
    "conclusion_limitations": [{"text": "...", "tags": ["H1", "U2"]},
                               {"heading": "Record of the investigation"}, {"text": "...", "tags": ["T03-seis"]}]
  },
  "literature_references": [{"evidence_id": "T03-lit/E1"}]
}
```

- **`keywords`**: three to eight short terms for the PDF metadata.
- **`figures_omitted_reason`** (optional): if no bundle figure supports a claim that Results and Discussion states, say why here. Without it, the renderer refuses a Results section that has no figure while the bundle holds figures.
- **A block** is a paragraph `{text, tags}`, a heading `{heading}`, a figure `{figure, caption, label, tags}` or a table `{table: {caption, columns, rows, label}, tags}`.
  - A heading opens a subsection. A heading has no tags and no marker. The executive summary has no headings and no tables.
  - The figure name is one from the overview's `figures`. Tag a figure with the claims that its specialist marked it as supporting (`supports`). Start its caption from the specialist's caption. The renderer adds the source line.
  - A table has a caption, column titles, rows of cells, and at most eight columns. Its tags are the ledger ids its numbers rest on.
  - A **label** is lower-case letters, digits and hyphens, unique in the draft. The text refers to the figure or table as `[[fig:<label>]]` or `[[tab:<label>]]`. The marker prints as "Figure 3" or "Table 2", whole: write "as [[fig:tie]] shows", never "Figure [[fig:tie]]". The data table's label is `data`; it exists when the bundle lists data. The renderer warns of a figure or table the text never refers to.
- **`part`** on an executive summary paragraph names which of its eight parts the paragraph serves (`lit-report-writing`, section 1). The renderer warns when a part is missing; it never refuses for it.
- **Tags** are ledger ids:
  - evidence and claim ids (`T03-seis/E2`, `T05-struct/C1`);
  - task ids;
  - Geo Oracle's `H#`, `X#`, `U#`, `Q#` and `F#` (a figure id in the ledger's Figures table);
  - `frame`, the ledger's Frame section (the objective, the decision, the area), for the background, objective and decision of the executive summary, when the ledger has that section.

  Tags never print. Each paragraph, figure and table in the Executive summary, in Results and Discussion and in Conclusion and Limitations needs at least one tag. The text never prints a tag: the renderer refuses a ledger label written in place of the hypothesis, contradiction or uncertainty it names (`lit-report-writing`, section 7).
- **Citations in the text** are markers only. A method is `[[m:<domain>/<name>]]`. The method must have run in this investigation: it is in the overview's `methods_run`, or it is a connector method (`m:lit/openalex-scholarly-index`, `m:lit/exa-semantic-search`, `m:lit/tavily-web-search`) that a methods statement names. A search keeps no local record, so `methods_run` does not list it. A published source is `[[lit:<evidence id>]]`, with an entry in `literature_references`. The renderer turns markers into author-year citations from the registry and from your entries.
- **No other ids in the text.** No `prov:` ids, no method ids outside a marker, no task, evidence, claim or job ids, no product paths. The renderer refuses a draft that shows one.
- **Complete and printable.** The renderer also refuses a draft that has no title, no executive summary, an empty section or an empty paragraph. It also refuses a draft with a character that no report font can print. Write such a character in words or in plain characters.
- **Literature references** come only from the literature evidence in the bundle, each source once, as `{evidence_id}`. When two evidence items come from one work, list the work once, under one of them. Cite both statements with that marker. The renderer refuses the same work under two entries. Do not cite an item that `part="literature"` marks `do_not_cite`. The renderer prints each reference from what that evidence recorded: author, year, title, venue and DOI. It never prints what you type. A `doi`, `authors`, `year` or `title` that you add must agree with the record. Cite only a resolved source: its `source_reference` has `doi:`, `openalex:`, `url:` or `project:`. Describe an `UNVERIFIED` lead in the text as an unverified lead, without a citation. A source whose record names no author and year cannot print as a reference, unless it is a project document. Describe such a source in the text without a citation. Do not add a source from memory. If a source was read below full text (abstract, metadata or secondary), its access level prints beside the reference.

## 3. What goes in each section

`lit-report-writing` says what each section holds and in what order. In short:

| Section | Content |
|---|---|
| Executive summary | The abstract, in eight parts, from the Final status block. Every conclusion at its ledger status; no conclusion added; numbers and status words added. |
| Introduction | The objective, the decision, the data (with the data table), the scope, the structure of the report. |
| Geology of the study area | The regional setting from the literature evidence and the regional, stratigraphy and sedimentology findings; one paragraph when there are none. |
| Data and Methodology | The data table (the renderer's), then the methods in workflow order, each methods statement once, with its numbers and markers. |
| Results and Discussion | One subsection per subject: the claims with their class and status, the numbers, the figures and tables the text refers to, the alternatives with their tests, the contradictions, the limitations. |
| Conclusion and Limitations | The answer, each hypothesis by content with its status, the limitations, the material uncertainties, what would settle the open items, then the record of the investigation. |

**Methods statements.** Each specialist wrote a short statement of what it ran. Use each statement once, at its stage of the workflow. Keep every number, and cite each method it names with its marker. The renderer lists a lost method or number as drift, and Geo Oracle reviews the drift. The renderer refuses a report that cites no method while the specialists' methods statements name methods.

**Numbers.** Copy each number and unit exactly as the ledger or the findings give it. Do not round, convert or average. A number that only set-aside evidence holds is not available: the renderer refuses it.

## 4. The rule for conclusions

You restate Geo Oracle's conclusions. You never raise, lower or add one.

- Each conclusion has the strength its ledger status gives it. "Weakly constrained" is not "supported". "Unresolved" is not "likely".
- A hypothesis the ledger keeps open stays open. A contradiction the ledger preserves stays in the report.
- A limitation in the ledger or the findings stays with the conclusion it limits.
- If you think the ledger is wrong, do not correct it in the report. Write the concern in your `limitations` for Geo Oracle.

## 5. ASD-STE100

Write all text in ASD-STE100 (Simplified Technical English), with the rules of `lit-report-writing` section 9:

- A sentence has 25 words or fewer. A paragraph has six sentences or fewer. Captions, table cells and headings are not sentences.
- Use the active voice: "The specialist measured the porosity", not "The porosity was measured".
- Use one topic in each sentence.
- Do not use a word ending in -ing as a noun ("the mapping" becomes "the map" or "to map").
- Use simple, common words, with one meaning for each word. Use the technical names of the geology as they are, and the standard symbols for units.

The renderer counts sentence and paragraph length and finds passive voice and -ing nouns. These come back as warnings. Correct them before you return the draft, and before the next round.

For Geo Oracle's review, it also warns on:
- a number in a summary, result or conclusion paragraph that is not in the items it is tagged with (a figure's specialist caption and its record's caption count, and so do a claim's alternatives);
- a caption whose value for a quantity differs from the figure's own record ("contour 10 ms" against "contour step 2 ms"): write the record's value;
- a rare number that is also in set-aside evidence;
- strong wording ("proven", "confirmed", "commercial", "will produce"), unless negated in place;
- a ledger contradiction or uncertainty that no paragraph is tagged with (a hypothesis is refused, not warned);
- a hypothesis the executive summary is not tagged with, and an executive summary part that no paragraph marks;
- a figure tagged with claims its specialist did not mark;
- bundle figures the report does not use, and figures or tables the text does not refer to;
- a Results section with no figure.

Methods drift comes first in the warnings, then these, then the style warnings.

## 6. Render

0. Read your draft against the list in `lit-report-writing` section 10. Correct the draft first.
1. Call `lit_render_report(investigation_id, draft_json, review_round, wait_seconds=240)`. Use the round your brief gives (1 for the first draft). The renderer refuses any other round.
2. **Rejected:** the message lists every error. Correct all of them and call again with the same round.
3. **Running:** collect it with `get_literature_job_result`.
   When it completes, read every warning it returns. Correct the draft and render again with the same round when a warning names a lost number, a lost method, a hypothesis the summary does not rest on, a missing part or a figure the text does not refer to. Style warnings alone do not need a new render in the same round.
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
