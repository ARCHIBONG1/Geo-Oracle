---
name: lit-investigation-report
description: How to read the frozen investigation bundle and render the formal investigation report. Use when Geo Oracle's brief asks you to draft, render or revise the investigation report (a task named R1-lit-report, R2-lit-report or R3-lit-report). It covers lit_read_bundle and the parts you must read to the end, the draft JSON with its sections, headings, figures, tables, labels, citations, derived values and hidden tags, the published work you may add, the narrow rule for conclusions, how to render with lit_render_report, and what to return. The content and register of each section are in lit-report-writing, which you read next.
---

# The investigation report

You write the formal report of an investigation that is closed. Geo Oracle and the specialists did the work; your task is to present it as a scientific report that the people who decide on the investigation can read from start to end, with the rigour of a peer-reviewed geoscience paper. You do not do new analysis and you do not change a conclusion. You may search for and verify published work to set the study in context and to compare its results (section 5). Read `lit-report-writing` after this skill: it says what each section holds and how to write it.

The report reads clean: no ids in the text. Each block carries hidden tags that name the ledger items it rests on, and the renderer prints the ledger, unchanged, as the appendix, so a reader can check every paragraph against it.

## 1. Read the bundle

The gateway froze the investigation into a bundle when the person asked for the report. Read it with `lit_read_bundle`, one part at a time. **The renderer refuses to render until you have read the parts the overview's `read_before_render` names, each to its last page**: `overview`, `digest`, `ledger`, `supervision` and, when the bundle holds figures, `figures`. A long part comes in pages: when a reply gives `items_next_offset`, `tasks_next_offset` or `next_offset`, call the same part again with `offset` set to it, until a reply gives none. Every page you read is recorded.

| Part | What it gives |
|---|---|
| `overview` | The objective, the HOTL setting, the executive summary Geo Oracle gave the person, one line per task, the figures (agent, caption, the claims each supports), the data, the methods that ran, what is not available, `do_not_rely_on` (superseded tasks, set-aside evidence), and whether the ledger has a Synthesis section. |
| `digest` | Every task's reply, item by item: the specialist's summary word for word; each claim with its class, status and alternatives; each evidence statement with its limitations; each methods statement; the contradictions, limitations, assumptions, unknowns, missing data, conclusions and follow-ups; the figures; and the prose of the reply outside its findings JSON. This is your main source. Read all of it. |
| `ledger` | Geo Oracle's ledger: the reference for every status and every conclusion. Its Synthesis section, when present, is Geo Oracle's reading of the results together. |
| `supervision` | Geo Oracle's supervision record: the request (objective, deliverables, constraints, the decision chain), each checkpoint (why the work stopped or went on, its contribution, the deliverables, the hypotheses, the items and the answers to them) and the requester's interventions. It tells you what was asked, what was decided and why. |
| `parameters` | Each recorded computation: tool, methods, parameters, and the tasks that cite it. The source of the parameter table in Methods. |
| `task` (with `task_id`) | One task's findings in one view, when you need to check a claim against its evidence. |
| `literature` | The literature evidence of the investigation that you may cite. |
| `figures` | Every figure: file name, agent, caption and the claims it supports. Required reading when the bundle holds figures. |
| `data`, `methods` | The lists the overview names by count only. |

Use only the bundle for results and conclusions, and do not use your general knowledge as a finding. Do not rest a result or a conclusion on a superseded task or on a set-aside claim or evidence item; the overview lists them, and the digest and each task view mark them (`superseded`, `set_aside`). Do not tag them, and do not carry their methods statements.

## 2. The draft

The draft has the parts below. **Store each part with `lit_draft_section(investigation_id, section, blocks_json, review_round)`**: first `title` (one object with the top-level fields: title, keywords, setting_title, figures_omitted_reason, literature_references, literature_sources), then each section's block list (`executive_summary`, `introduction`, `setting`, `data`, `methods`, `results`, `discussion`, `conclusions`). Each reply lists that part's `errors`: correct them and store the part again until it has none. Store a long section in pieces with `append: true`; a part stored on its own cannot regress when another is corrected. In a revision round the store starts from the previous round's draft: store only the parts you change. The stored parts assemble to the draft below:

```json
{
  "title": "Short title of the investigation",
  "keywords": ["seismic interpretation", "well tie"],
  "executive_summary": [{"part": "background", "text": "...", "tags": ["frame"]},
                        {"part": "results", "text": "...", "tags": ["H1", "T05-struct/C1"]}],
  "sections": {
    "introduction": [{"text": "... earlier work on the basin [[lit:L1]] ...", "tags": []}],
    "setting": [{"text": "... [[lit:T03-lit/E1]].", "tags": ["T03-lit/E1"]}],
    "data": [{"text": "[[tab:data]] lists the data. ...", "tags": []}],
    "methods": [{"heading": "Well products"},
                {"text": "The petrophysics specialist calculated ... [[m:petro/porosity-permeability-transform]].", "tags": ["T01-wells/E5"]},
                {"table": {"caption": "...", "columns": ["Step", "Method", "Parameters"], "rows": [["...", "...", "..."]], "label": "parameters"}, "tags": []}],
    "results": [{"heading": "The closure"},
                {"text": "... as [[fig:closure]] and [[tab:closures]] show.", "tags": ["T05-struct/C1"]},
                {"figure": "closure_map.png", "caption": "...", "label": "closure", "tags": ["T05-struct/C1"]},
                {"table": {"caption": "...", "columns": ["Closure", "Area"], "rows": [["A", "2.1 km2"]], "label": "closures"}, "tags": ["T05-struct/C1"]}],
    "discussion": [{"text": "... a difference of 12 ms[[d:1]] ...", "tags": ["H2", "synthesis"],
                    "derived": [{"value": "12", "op": "difference", "inputs": ["1520", "1508"]}]},
                   {"table": {"caption": "...", "columns": ["Reading", "Evidence for", "Evidence against", "Deciding test"], "rows": [["..."]], "label": "alternatives"}, "tags": ["H2", "X1"]}],
    "conclusions": [{"text": "...", "tags": ["H1", "U2"]},
                    {"heading": "Record of the investigation"}, {"text": "...", "tags": ["T03-seis"]}]
  },
  "literature_references": [{"evidence_id": "T03-lit/E1"}],
  "literature_sources": [{"source_id": "L1", "source_reference": "Author et al. 2015, Journal | title: ... | doi: 10.xxxx/... | access: abstract"}]
}
```

- **`keywords`**: three to eight short terms for the PDF metadata.
- **`setting_title`** (optional): `Geological setting` (the default) or `Description of the synthetic model`, for a study of a synthetic volume.
- **`figures_omitted_reason`**: only when no bundle figure supports a claim the report states; the renderer checks this and refuses the reason otherwise. Place each figure beside the paragraph that states a claim it supports.
- **A block** is a paragraph `{text, tags}`, a heading `{heading}`, a figure `{figure, caption, label, tags}` or a table `{table: {caption, columns, rows, label}, tags}`.
  - A heading opens a subsection; it has no tags and no marker. The executive summary has no headings, figures or tables.
  - The figure name is one from the overview's `figures`. Tag a figure with the claims its specialist marked it as supporting (`supports`). Start its caption from the specialist's caption; the renderer adds the source line (the specialist's discipline).
  - A table has a caption, column titles, rows of cells, and at most eight columns. Its tags are the ledger ids its numbers rest on.
  - **Every figure and table needs a label** (lower-case letters, digits and hyphens, unique), and **the text must refer to each one** as `[[fig:<label>]]` or `[[tab:<label>]]`; the renderer refuses either gap. The marker prints the word and the number ("Figure 3", "Table 2"): write "as [[fig:tie]] shows", never "Figure [[fig:tie]]" or "Tables [[tab:a]]", which the renderer refuses. The data table's label is `data`; it exists when the bundle lists data, and the renderer prints it at the top of the Data section.
- **`part`** on an executive summary paragraph names which of its eight parts the paragraph serves (`lit-report-writing`, section 1).
- **Tags** are ledger ids: evidence and claim ids (`T03-seis/E2`, `T05-struct/C1`); task ids; Geo Oracle's `H#`, `X#`, `U#`, `Q#` and `F#`; `frame` (the ledger's Frame section) and `synthesis` (the ledger's Synthesis section), when the ledger has them. Tags never print. Each block of the executive summary, Results, Discussion and Conclusions needs at least one. The text never prints a tag: the renderer refuses a ledger label written in place of the hypothesis, contradiction or uncertainty it names (`lit-report-writing`, section 9).
- **Citations in the text** are markers only. A method is `[[m:<domain>/<name>]]`; it must have run in this investigation (it is in the overview's `methods_run`, or it is a connector method that a methods statement names, since a search keeps no local record). A source of the investigation is `[[lit:<evidence id>]]`, with an entry in `literature_references`; a source you found for the report is `[[lit:L<n>]]`, with an entry in `literature_sources` (section 5). Put the marker in the clause that names the step or the statement it supports, not at the end of a list.
- **Derived values** (`derived`, on a paragraph of the executive summary, Results, Discussion or Conclusions): you may do arithmetic on recorded numbers, a sum, difference, product, ratio, mean or percent, when the result does not contradict the investigation. Give each as `{value, op, inputs}`. Every input must be in the items the paragraph is tagged with, and the value must be the result at the decimals you write. Mark the number in the text with `[[d:<n>]]` right after it (n counts from 1 in the paragraph); it prints a dagger with the footnote "Calculated for this report from recorded values." The renderer checks the arithmetic.
- **No other ids in the text**: no `prov:` ids, no method ids outside a marker, no task, evidence, claim or job ids, no product paths. The renderer refuses a draft that shows one.
- **Complete and printable.** The renderer refuses a draft with no title, no executive summary, an empty section or paragraph, an unknown section, or a character no report font can print.
- **Literature references** come from the literature evidence in the bundle, each work once, as `{evidence_id}`. When two evidence items come from one work, list the work once and cite both statements with that marker. Do not cite an item that `part="literature"` marks `do_not_cite`. The renderer prints each reference from what the evidence recorded, never from what you type; a `doi`, `authors`, `year` or `title` you add must agree with the record. Cite only a resolved source (`doi:`, `openalex:`, `url:` or `project:`); describe an `UNVERIFIED` lead in the text without a citation. A source read below full text prints its access level.

## 3. The sections

`lit-report-writing` says what each section holds and in what order. In short:

| Section | Content |
|---|---|
| Executive summary | The abstract for the decision makers, in eight parts, from the Final status block and the Synthesis. |
| Introduction | The problem, the prior work, the question, and what this investigation contributes. |
| Geological setting | The regional and local geology; or, for a synthetic volume, a description of the model. |
| Data | The data table (the renderer's), then the data, their quality and their gaps. |
| Methods | The methods by stage of the workflow, each methods statement kept whole, and the parameter table. |
| Results | One subsection per subject: each claim with its numbers, uncertainty, class and status, and its figures and tables. |
| Discussion | How the results combine, the alternatives table, comparison with published work, then the limitations. |
| Conclusions | The answer, each hypothesis by content with its status, what would settle the open items, the record of the investigation. |

## 4. What the renderer refuses and what it warns on

It **refuses** (correct every one and render again):
- a ledger hypothesis that no block is tagged with, or that the executive summary is not tagged with;
- a claim that is supported, partially supported, unresolved or contradicted and that no Results or Discussion block is tagged with;
- a methods statement that lost a method, or two or more of its numbers and a quarter of them, in Methods;
- no parameter table in Methods, when the bundle records computations; no alternatives table in Discussion, when the investigation has hypotheses or alternatives;
- a ledger Synthesis section that no Discussion or Conclusions paragraph is tagged `synthesis` with;
- a figure or table with no label, or one the text never refers to;
- a derived value whose inputs are not in its tags or whose arithmetic is wrong;
- a source you found for the report cited outside the Introduction, the Geological setting and the Discussion, or one that is not resolved;
- ids, bare ledger labels, numbers that only set-aside evidence holds, and markers it cannot read.

It **warns** (correct them before you return the draft):
- a methods statement that lost fewer than two of its numbers, or fewer than a quarter of them (drift), or a method whose record the bundle lacks (describe it in words; it cannot be cited);
- a number not in the items a paragraph is tagged with; a caption value that differs from the figure's record;
- strong wording ("proven", "confirmed", "commercial", "will produce") not negated in its clause;
- a Results paragraph that states measurements with no uncertainty;
- a ledger contradiction or uncertainty that no block is tagged with; an executive summary part not marked;
- bundle figures the report does not use; an alternatives table without columns for the evidence against and the deciding test;
- a sentence longer than about 40 words (style advice, marked `style:`; split it where that helps).

## 5. Published work you add

In a report task you may search for and verify published work (skills `lit-search-protocol` and `lit-source-verification`) for three purposes: the prior work in the Introduction, the regional context in the Geological setting, and the comparison of the results with published studies in the Discussion. List each work in `literature_sources` as `{source_id: "L1", source_reference}`, with the `source_reference` in the format of `lit-evidence-extraction`: author, year and venue, title, a DOI or an OpenAlex work id, and the access level you read. Cite it as `[[lit:L1]]`. The renderer refuses it elsewhere, and refuses a source without a DOI or OpenAlex id, an author, a year and a title. Never use published work to change, strengthen or weaken a conclusion of the investigation: it gives context and comparison only, and you say so where the comparison differs from the result. Say in `limitations` which works you added and why.

## 6. The rule for conclusions

You restate Geo Oracle's conclusions. You never raise, lower or add one.
- Each conclusion has the strength its ledger status gives it. "Weakly constrained" is not "supported". "Unresolved" is not "likely".
- A hypothesis the ledger keeps open stays open. A contradiction the ledger preserves stays in the report.
- A limitation in the ledger or the findings stays with the conclusion it limits.
- If you find the ledger inconsistent with itself or with the findings (a status that differs between two places, a task listed as running at the close, a number that differs from its record), do not correct it in the report. Report each inconsistency to Geo Oracle in your `limitations`, with where you found it.

## 7. Render

1. Read your stored parts against the checklist in `lit-report-writing` section 12, and store the corrected ones.
2. Call `lit_render_report(investigation_id, review_round, wait_seconds=240)` with the round your brief gives and no `draft_json`.
3. **Rejected:** the message lists every error that remains (or the bundle parts still unread). Correct them in the parts they name (store those parts again) and call again with the same round. **Keep correcting and rendering until the draft renders**: a round has no limit on renders, and a refused draft is no report. A corrected draft is a changed call, so the rule to stop after two refusals does not apply to `lit_render_report`. Stop only for an error the draft cannot correct (a missing bundle part, a server failure) or when you cannot finish the task (see Unfinished work), and name the remaining errors in `limitations`.
4. **Running:** collect it with `get_literature_job_result`. Read every warning in `style_warnings`. Correct the draft and render again with the same round for each drift, content, uncertainty and `layout:` warning; style advice alone (`style:`) needs no new render.
5. **`release_checks: failed`:** the PDF has no download link. Correct the problems and render again.
6. **LaTeX could not compile:** the message gives the first LaTeX error and its line; correct what it names.

## 8. Return

Put these in `conclusions`:
- `Report (round <n>): <download_url>` (if the release checks passed; if not, the problems that remain);
- the release checks, the page count, and the number of drift, content and style warnings left;
- for a revision, one line for each of Geo Oracle's review notes: what you changed, or why you did not.

Put the ledger inconsistencies you found, and the published works you added, in `limitations`. Do not put the PDF in a `sandbox_artifacts` block: it is not in your sandbox.

## 9. Revisions

A revision brief carries Geo Oracle's review notes as evidence `R1`, `R2` and so on. Each note is a contradiction between your draft and the analysis. For each note, read the bundle again and correct the draft; correct the warnings of the last render too. Then render with the round the brief gives. There are three rounds at most, so correct every note in one round. The bundle parts you read in round 1 stay read.

## Methods statements

Write one methods statement for this task, in 1 to 3 sentences: that you drafted the report from the frozen bundle and rendered it, and, if you did, that you searched for and verified published work for the Introduction, Setting and Discussion. Its `method_ids` are `m:lit/render-report` and `m:lit/investigation-report` (and the connector methods of any search you ran), and its `provenance_ids` hold the `prov:` id of the render you return. If no render succeeded, there is no `prov:` id: leave `method_ids` and `provenance_ids` empty and say in the text that the render failed. Do not name `lit_read_bundle` in it: a read of the bundle records no computation.

<!-- end of skill: lit-investigation-report -->
