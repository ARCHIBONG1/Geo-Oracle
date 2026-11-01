---
name: lit-report-writing
description: How to write each section of the formal investigation report for the people who decide on it, with full scientific rigour. Covers the executive summary and its eight parts, the introduction, the geology, the methods as a workflow, the results by subject with figures, tables, alternatives and status words, the conclusions and the record of the investigation, hypotheses written by content, captions, the register and the words to avoid, and the checks before a render. Read it after lit-investigation-report, before you write any section, and again before you render.
---

# Writing the investigation report

The readers are the people who decide on the investigation: stakeholders and investors. They are not the specialists. Keep the full rigour of the findings. Explain each technical term once. Lead each section with what matters to the decision.

`lit-investigation-report` gives the mechanics: the bundle, the draft JSON, tags, markers, labels and the render. This skill gives the content and the style of each section.

## 1. The executive summary

The executive summary is the abstract of the report. Write it in eight parts, in this order, one or more paragraphs each. Mark each paragraph with its `part`:

| part | Content | Tags |
|---|---|---|
| `background` | Broad to specific: the geological context, then the area and the data, then the task. One or two sentences of general geoscience context are allowed here and nowhere else; they hold no number and no claim about this area. | `frame` |
| `objective` | The primary objective, as the requester asked for it. | `frame`, the task ids |
| `decision` | The decision the work supports, and what was asked. | `frame` |
| `delivered` | What was delivered and what was not, in one paragraph. | the claim ids and hypotheses |
| `results` | The principal results, each with its key numbers and its ledger status in words. | the claim ids and hypotheses, and the evidence ids that hold the numbers |
| `open_items` | The open items, and why each matters to the decision. | the contradictions, uncertainties and open hypotheses |
| `confidence` | The confidence in the answer, and the main limitations. | the uncertainties and the claims they limit |
| `next_steps` | What would settle the open items. | the hypotheses and uncertainties they would settle |

Rules:
- `frame` is a tag when the ledger has a Frame section. When it has none, tag these parts with the task ids.
- If the ledger has no Final status block, take the conclusions from the Hypotheses table and the closing summary.
- The Final status block of the ledger is the source. Every conclusion comes from it, at the status it holds there. Add no conclusion. Numbers and status words may be added.
- Tag every ledger hypothesis in the summary. The renderer warns when one is missing.
- Write 250 to 500 words. Write numbers with their units. A part may hold more than one paragraph; two short parts may share one paragraph when it is marked with the first part.
- Geo Oracle's closing summary is the anchor: keep each of its conclusions and each of its negations.

## 2. Introduction

Write it in this order:
1. The objective, in one sentence.
2. The decision, in the form "The decision this work supports is ...".
3. The data the requester supplied. When the bundle lists data, the renderer prints the data table: refer to it as `[[tab:data]]`, which prints as "Table 1". When the bundle lists no data (the overview's `data` is empty), name the inputs in the text instead.
4. The scope and its exclusions.
5. The structure of the report, in one sentence.

If the bundle holds no literature evidence, write: "No published work on the study area was part of this investigation. The published sources of the methods are listed in References." Do not write that the report cites no source: the method references are sources.

## 3. Geology of the study area

Write the regional setting from the literature evidence and from the regional, stratigraphy and sedimentology findings. Cite each source with its marker.

If the bundle holds none of these, write one paragraph only. Say that the geological frame comes from the well tops and the seismic data. Name the formations the tops give, with their depths or times. Say that no regional context was part of this investigation. Write no result in this section. A result belongs in Results and Discussion.

## 4. Data and Methodology

The renderer prints the data table when the bundle lists data. Then write the methods in the order of the workflow, not in the order of the tasks: data preparation, well products, seismic quality control, well tie, horizons, faults, bodies, attributes, depth conversion, follow-up work. Open each stage with a heading block.

- Use each methods statement once, at its stage. Keep every number and every method marker of the statement.
- Give the reason for each key choice when the findings state one.
- Name the specialist by discipline once per stage, not in each sentence. Write "The seismic specialist tracked ..." once, then "The tracker ...".
- Cite a method once, in the sentence that first names it, at the end of that sentence. A method with no published source prints a star; write nothing else for it.
- If a methods statement was set aside, write one sentence: which results depend on it, and that the record of the method is withheld.

## 5. Results and Discussion

Give one subsection to each subject, with a heading block: the input data, the well products, the well ties, the horizons, the faults, the bodies, the attributes, the depth conversion, the horizons not delivered, and so on. Within a subsection:

1. State each claim with its class and its status in words: "The tie is a measurement, supported: ...". Use the status vocabulary of section 7.
2. Give the numbers, with units, as the findings give them.
3. Refer to the figure or table that shows the claim, in the sentence that uses it: `[[fig:<label>]]`, `[[tab:<label>]]`.
4. For each claim whose findings record an alternative, write the alternative, the test that would decide it, and whether the test ran. The task view gives them under `alternatives`.
5. Write the contradictions of the task, with their discriminating evidence.
6. Write the limitations that the evidence items carry.

Put numbers that compare three or more items in a table block: well ties, horizon statistics, fault throws and distances, depth ranges and uncertainties. Give each table a caption and a label.

Products that the investigation did not release (a diagnostic map, a failed track) go under one heading, "Diagnostic products not released". Say in the first sentence why they are shown.

## 6. Conclusion and Limitations

Write it in this order:
1. The answer to the objective, at the status the Final status block gives it.
2. Each hypothesis, by its content, with its status in words.
3. The scientific limitations: what the data and the methods could not do.
4. The material uncertainties, and which conclusion each can change.
5. What would settle each open item.
6. A heading block "Record of the investigation", then one paragraph: the tasks that did not run or ran again, and the records that were withheld, in one sentence each, without platform words. Name a task by its content, not its id: "The first seismic quality-control task failed on a dropped connection and ran again." Tag the paragraph with the ids the ledger's Tasks table holds for those runs, and with the task that replaced a superseded one (a superseded task id and set-aside evidence cannot be tags). If the paragraph needs more than six sentences, split it into two.

An execution event is not a scientific limitation. A task that was cut off, a connection that dropped, a task that ran again, a task that was split: these go in the record paragraph only. Never in the answer, the hypotheses or the limitations. A withheld record is different: it is a fact about the provenance of a result. Say it once, in one sentence, where the method or the result is: "The record of the fault orientations was withheld, so the dips rest on the specialist's claim only." Do not say why it was withheld.

## 7. Hypotheses, contradictions and uncertainties by content

The ledger names its hypotheses, contradictions and uncertainties with labels. The labels are tags. The text never prints them.

- Write a hypothesis by its content and its status: "the hypothesis that the closure holds oil (weakly constrained)".
- Write a contradiction by its content: "the report gives one porosity for the sand, the log another".
- Write an uncertainty by its content: "no core was available, so the permeability rests on the log transform".
- Horizon, well and marker names stay as the investigation names them. A horizon named with a letter and a digit keeps that name. The renderer refuses a label that stands for a hypothesis.

Status vocabulary. A ledger hypothesis has one of these statuses: untested, weakly constrained, partially constrained, supported, strongly supported, contradicted, unresolved, superseded. A claim has one of these: proposed, supported, partially supported, contradicted, unresolved, superseded. The objective is answered, partly answered or not answerable. Use the status word the ledger or the findings give, and no other. "Supported" is not "proven". "Weakly constrained" is not "supported". "Unresolved" is not "likely". A superseded claim is written as superseded, with the claim that replaced it.

## 8. Figures and captions

- Label every figure and table, and refer to each one in the text.
- A caption has three parts: what is shown, with the units and the slice or horizon; what the reader should see, in one sentence; and what it rests on. The renderer adds the source line.
- Explain every symbol: blank cells, bands, dots, lines, colours.
- Start from the specialist's caption, then check each of its numbers against the findings and the figure's record. Write the number the record gives.
- Place each figure in the subsection that discusses it, after the paragraph that refers to it.

## 9. Register

Write in ASD-STE100, with these rules for a scientific report:
- A sentence has 25 words or fewer. A paragraph has six sentences or fewer. Use the active voice. One topic in each sentence.
- Use the standard symbols: ±, %, °, m, ms, m/s, km. Write the symbol, not the words.
- Captions, table cells and headings are not sentences. The length rules do not apply to them.
- Do not use platform words. Write "the requester", not "the person". Write "the investigation record", not "the bundle". Write "withheld", not "set aside". Do not write "gateway", "HOTL", "specialist call", "iteration", "job" or "turn".
- Define each technical term that is not standard geoscience usage the first time you use it.
- Copy each number and unit as the ledger or the findings give it. Do not round, convert or average.
- Name the specialists by discipline: "the petrophysics specialist", "the seismic specialist".

## 10. Before you render

Read your draft once against this list. Correct the draft, then render.
1. Each section has the content that sections 1 to 6 name. The executive summary has its eight parts.
2. Every figure and table has a label, and the text refers to it.
3. Every ledger hypothesis, contradiction and uncertainty is written by its content, and tagged.
4. No label and no platform word is in sections 1 to 5. No execution event is in them outside the record paragraph; a withheld record is stated in one sentence where its result is.
5. The Introduction and the References agree.
6. Every number in the summary, the results and the conclusions is in a tagged item.
7. Every claim with a recorded alternative has its alternative sentence.
8. The methods are in workflow order, with each methods statement used once.

After the render, read every warning. Correct the draft before you return it.

<!-- end of skill: lit-report-writing -->
