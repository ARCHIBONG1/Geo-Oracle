---
name: lit-report-writing
description: How to write each section of the formal investigation report as a scientific report for the people who decide on the investigation. Covers the executive summary and its eight parts, the introduction, the geological setting, the data, the methods with their parameter table, the results by subject with uncertainties and figures, the discussion with its alternatives table, the conclusions and the record of the investigation, hypotheses written by content, naming and terms, captions, the scientific report register, and the checks before a render. Read it after lit-investigation-report, before you write any section, and again before you render.
---

# Writing the investigation report

The readers are the people who decide on the investigation: stakeholders and investors, with geoscientists among them. Write the body as you would a paper for a peer-reviewed geoscience journal: the full evidence, the reasoning that connects it, and the limits of what it shows. Write the executive summary for the decision makers, in plain language, with the same rigour.

`lit-investigation-report` gives the mechanics: the bundle, the draft JSON, tags, markers, labels and the render. This skill gives the content and the register of each section. Write from the whole bundle: the digest holds every claim, alternative, limitation and methods statement of every task, and the report must carry them.

## 1. The executive summary

The executive summary is the abstract of the report. Write it in eight parts, in this order, one or more paragraphs each, and mark each paragraph with its `part`:

| part | Content | Tags |
|---|---|---|
| `background` | From the broad to the particular: the geological and business context of the area, then the area and its data, then the task. One or two sentences of general geoscience context are allowed here and nowhere else, with no number and no claim about this area. Give context; do not define methods. | `frame` |
| `objective` | The primary objective, as the requester asked for it. | `frame`, the task ids |
| `decision` | The decision the work supports, and what was asked. | `frame` |
| `delivered` | What was delivered and what was not, in one paragraph. | the claim ids and hypotheses |
| `results` | The principal results, each with its key numbers, its uncertainty and its ledger status in words. | the claim ids and hypotheses, and the evidence ids that hold the numbers |
| `open_items` | The open items, and why each matters to the decision. | the contradictions, uncertainties and open hypotheses |
| `confidence` | The confidence in the answer, and the main limitations. | the uncertainties and the claims they limit |
| `next_steps` | What would settle the open items. | the hypotheses and uncertainties they would settle |

Rules:
- `frame` is a tag when the ledger has a Frame section; when it has none, tag these parts with the task ids.
- The Final status block of the ledger is the source of the conclusions, at the status each holds there; without one, take them from the Hypotheses table and the closing summary. Add no conclusion. Numbers and status words may be added.
- Tag every ledger hypothesis in the summary, written by its content with its status; the renderer refuses a summary that misses one.
- Write 250 to 500 words. Write numbers with their units.
- Geo Oracle's closing summary is the anchor: keep each of its conclusions and each of its negations. The Synthesis section of the ledger, when present, gives how the results combine: use it in `results`, `open_items` and `confidence`.

## 2. Introduction

Four movements, in this order, each one or more paragraphs:
1. **The problem**: the geological question, why it matters for the decision, and the area.
2. **Prior work**: what published studies and the project's own reports establish about the area and the question, cited. You may search for and verify published work for this (`lit-investigation-report`, section 5). When neither the bundle nor your search finds any, say so in one sentence.
3. **The question**: the objective and the decision, in the form "The decision this work supports is ...".
4. **The contribution**: what this investigation adds, its scope and its exclusions, and one sentence on the structure of the report.

## 3. Geological setting

The regional and local geology: the basin, the structural and stratigraphic framework, and the units the study names, from the literature evidence, from the regional, stratigraphy and sedimentology findings, and from published work you verified. Cite each source in the clause it supports. Name the formations the well tops give, with their depths or times.

When the data are synthetic (the objective or the data say so), set `setting_title` to `Description of the synthetic model` and describe the model: what it represents, its dimensions and its known properties, from the bundle. Write no result in this section.

## 4. Data

The renderer prints the data table at the top of the section. Refer to it as `[[tab:data]]`. Then describe the data as the findings report them: each data set, its coverage, its quality (the quality control results), its gaps and what they prevent, and the products that later stages used. When the bundle lists no data, name the inputs in the text.

## 5. Methods

Write the methods by stage of the workflow, not task by task: data preparation, well products, seismic quality control, well tie, horizons, faults, bodies, attributes, depth conversion, follow-up work. Open each stage with a heading block.

- **Keep each methods statement whole.** Every method it names and every number it states stays in Methods, at its stage. Re-order and connect the statements so that each stage reads as one procedure, but merge nothing away: two statements that ran the same method with different settings are two statements. The renderer refuses a statement that lost a method or a fair part of its numbers.
- **When many statements are lost**, do not shorten them again. Put the statements under their stage headings first, then copy each one's method names and numbers into the text as the statement gives them, and connect the sentences around them. Summarise nothing.
- **The role of each method in words.** Say what each method does in this workflow and why it was chosen, when the findings state why: "The horizons were tracked with a waveform-correlation tracker [[m:...]], seeded at the well ties, because ...". Cite the method in the clause that names the step.
- Name the specialist by discipline once per stage, not in each sentence.
- A method with no published source prints a star; write nothing else for it.
- If a methods statement was set aside, write one sentence: which results depend on it, and that the record of the method is withheld.
- **The parameter table.** End the section with a table block of the computations: step, method, the key parameters and their values, from `lit_read_bundle(part="parameters")`. Group the rows by stage. Copy the values as recorded.

## 6. Results

One subsection per subject, with a heading block: the input data, the well products, the well ties, the horizons, the faults, the bodies, the attributes, the depth conversion, and so on. Within a subsection:

1. **Every claim** of the subject that is supported, partially supported, unresolved or contradicted, written with its evidence and its status in words, and tagged with its claim id. The renderer refuses a report that leaves one out.
2. **Measurements carry their uncertainty**: write "1250 ± 6 ms", or the range or spread the findings give. When no uncertainty was computed, say so: "no uncertainty was computed for the throw". Status words (supported, unresolved) belong to interpretations and hypotheses, not to a measurement.
3. **Figures and tables**: refer to each figure or table in the sentence that uses it, and say what it shows: the trend, the structure, the anomaly, and its scale. Put numbers that compare three or more items in a table block: well ties, horizon statistics, fault throws and distances, depth ranges and their uncertainties.
4. **The specialists' limitations** of each result, with the result.
5. **Products not released** (a diagnostic map, a failed track) go under one heading, "Diagnostic products not released"; say in the first sentence why they are shown.

Results report; they do not argue. The argument belongs in the Discussion.

## 7. Discussion

1. **How the results combine.** The investigation's reading of its results together: what depends on what, where the results agree, and where they disagree. Carry Geo Oracle's Synthesis section here and tag those paragraphs `synthesis`; the renderer refuses a report that leaves it out when the ledger has one.
2. **The alternatives.** For each hypothesis and each claim with a recorded alternative: the competing readings, the evidence for and against each, and the test that would decide between them, and whether it ran. Give them in a table block with the columns reading, evidence for, evidence against and deciding test, and discuss the ones that matter to the decision in the text.
3. **The contradictions** the ledger preserves, with their discriminating evidence.
4. **Comparison with published work**: how the results compare with published studies of the area or of analogues, cited. A comparison gives context; it never changes a conclusion. Where published work differs from a result, say so and say what would explain the difference.
5. **Limitations**: what the data and the methods could not do, and the material uncertainties, each with the conclusion it can change.

## 8. Conclusions

1. The answer to the objective, at the status the Final status block gives it.
2. Each hypothesis, by its content, with its status in words.
3. What would settle each open item.
4. A heading block "Record of the investigation", then one paragraph: the tasks that did not run or ran again, and the records that were withheld, in a sentence each, without platform words. Name a task by its content, not its id. Tag the paragraph with the ids the ledger's Tasks table holds for those runs, and with the task that replaced a superseded one.

An execution event (a task cut off, a dropped connection, a task run again or split) is not a scientific limitation; it goes in the record paragraph only. A withheld record is a fact about the provenance of a result: say it once, where the method or the result is ("The record of the fault orientations was withheld, so the dips rest on the specialist's statement only"), without saying why.

## 9. Naming, terms and labels

- **Ledger labels are tags, never text.** Write a hypothesis by its content and status ("the hypothesis that the closure holds oil, which remains weakly constrained"), a contradiction by its content, an uncertainty by its cause and effect. The renderer refuses a label that stands for a hypothesis.
- **Horizons and markers keep their names**, with the full name at first mention: the label, the formation name and the word horizon (for example "the H7 Lower Marl horizon"), then the short name. A horizon label alone, next to the hypotheses, reads as a hypothesis.
- **Terms**: define each technical term that is not standard geoscience usage at its first use, in a clause. Use one term for one thing throughout. Expand each abbreviation at its first use.
- Name the specialists by discipline: "the petrophysics specialist", "the seismic specialist".
- **Status vocabulary.** A ledger hypothesis is untested, weakly constrained, partially constrained, supported, strongly supported, contradicted, unresolved or superseded. A claim is proposed, supported, partially supported, contradicted, unresolved or superseded. The objective is answered, partly answered or not answerable. Use the status word the ledger or the findings give. "Supported" is not "proven", "weakly constrained" is not "supported", and "unresolved" is not "likely". A superseded claim is written as superseded, with the claim that replaced it.

## 10. Register

- **Paragraphs**: one claim, or one closely related group of claims, per paragraph. Open with the point, then the evidence, then what it means, with the connecting words that show the reasoning ("because", "so", "however", "which implies"). No list of disconnected sentences.
- **Sentences**: complete and varied, active or passive as the sense needs. Keep most below about 30 words; the renderer flags one above about 40.
- **Certainty only through the status words** of the ledger and the findings. No intensifiers ("clearly", "robustly", "strongly suggests") that raise a status.
- **Numbers**: copy each number and unit as the ledger, the findings or the records give it. Do not round, convert or average, except as a derived value (`lit-investigation-report`, section 2), which the renderer checks and marks. Use the standard symbols: ±, %, °, m, ms, m/s, km.
- **No platform words**: write "the requester", not "the person"; "the investigation record", not "the bundle"; "withheld", not "set aside". Do not write "gateway", "HOTL", "specialist call", "iteration", "job" or "turn".
- **The executive summary** is in plain language: short paragraphs, the decision first, each technical term explained in a few words, and no number without its meaning.

## 11. Captions

A caption has three parts: what is shown, with its units and its slice, horizon or line; what the reader should see, in one sentence; and what it rests on. Explain every symbol: blank cells, bands, dots, lines, colours. Start from the specialist's caption, check each number against the findings and the figure's record, and write the record's value. Place each figure after the paragraph that refers to it.

## 12. Before you render

Read your draft against this list, correct it, then render.
1. Every part of the digest, ledger and supervision record was read to its last page.
2. Each section has the content of sections 1 to 8; the summary has its eight parts and every hypothesis.
3. Every supported, partially supported, unresolved or contradicted claim is in Results or Discussion, tagged.
4. Every measurement has its uncertainty, or the sentence that none was computed.
5. Every methods statement is whole in Methods, and the parameter table is there.
6. The Discussion carries the Synthesis, the alternatives table and the limitations.
7. Every figure and table has a label, and the text refers to each one.
8. No label, id or platform word in the text; no execution event outside the record paragraph.
9. Published work you added is cited only in the Introduction, the Setting and the Discussion, and changes no conclusion.
10. The ledger inconsistencies you found are in your `limitations`.

After the render, read every warning and correct the draft before you return it.

<!-- end of skill: lit-report-writing -->
