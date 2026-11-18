---
name: lit-report-writing
description: How to write the formal investigation report so that a person who was not in the investigation can read it and act on it. It covers how to work, a report whose main question stays open, what each section does, paragraphs, terms, numbers and status words, figures and captions, and the renderer's notes. Read it after lit-investigation-report and again before you render. Worked examples are in lit-report-examples.
---

# Writing the investigation report

A report is an argument. It answers a question for a reader who must decide something and who was not there when the work was done. The investigation gave you the evidence and the conclusions. Your work is to make a reader understand them: what was asked, what the evidence shows, how far it goes, and what would change it.

A report is not a list of all that the investigation produced: the appendix is.

`lit-investigation-report` gives the mechanics. Open `lit-report-examples` when you draft a section.

## 1. How to work

1. **Read the whole bundle first.** Read every required part to its last page before you write a sentence of the report.
2. **Write the answer before the report.** In your own notes, write one short paragraph: the question the investigation sought to address, the answer it reached, the two or three results that carry that answer, and what stays open.
3. **Outline the argument.** List the subsections of Results and Discussion, and under each one the point it makes in one sentence. Then give every item a home in the outline before you draft:
   - every claim that is supported, partially supported, unresolved or contradicted;
   - every methods statement;
   - every hypothesis, contradiction and material uncertainty of the ledger;
   - every figure that shows something a conclusion rests on.
   An item with no home shows a gap in the outline: change the outline.
4. **Draft section by section, from the outline.** Write prose first, for the reader, without thinking about ids.
5. **Reverse-outline each section.** When a section is drafted, write down the first sentence of each paragraph, in order. Read only those sentences, as Geo Oracle will before anything else. They must tell the section's story with no gap and no repeat. Then check each paragraph against its first sentence: the rest of the paragraph is the evidence for it and nothing else. Join, cut or rewrite a paragraph that does not fit. Keep the item it carried by moving it to the paragraph where it belongs.
6. **Read the whole draft as the reader.** Read from the title to the last line as a geoscientist who does not know this investigation. Mark each place where you would stop.
7. **Tag last.** When the prose is right, add the tags to each block from the items it rests on.
8. **Render, and treat what comes back as a proofreader's notes** (section 8).

## 2. What a good report does

It gives the answer early. It shows the reader where things are before it says what was found there. It says once what it does not try to do. It gives a reason for each choice of method. It explains its terms and its figures. Its discussion argues to a picture. It says nothing twice. A rule of this skill that you satisfy with a sentence written only for the rule has failed.

## 3. When the main question stays open

Such a report is not a list of what went wrong. It tells the reader three things:

- **What the investigation sought to establish:** the question, and what answer would have supported the decision.
- **What it established:** the findings that stand, with their evidence. A thing shown to be absent, or not a factor, is a finding.
- **What it did not establish:** each item with why it could not be established from the supplied data, the candidate explanations that remain, and the data or test that would decide between them.

The executive summary carries this in full (its `delivered`, `results` and `open_items` parts). The body does not repeat it: Results give the evidence for each finding, and the Conclusions give only the answer and what would settle the open items.

## 4. The sections

### 4.1 Executive summary

The executive summary is the abstract of the report, written for the people who decide. It is in plain language, with the same rigour as the body. Write it last, from the finished body, in eight parts, in this order, and mark each paragraph with its `part`:

| part | What the reader gets | Tags |
|---|---|---|
| `background` | Where and what: the area, the data, the setting of the question. One or two sentences of general context are allowed here and nowhere else, with no number and no claim about this area. | `frame` |
| `objective` | What the investigation sought to address. | `frame`, the task ids |
| `decision` | The decision that the reader faces, then the answer, said plainly ("The evidence does not yet support a well"). Never "the work supports the decision whether ...": it reads as a yes. | `frame` |
| `delivered` | What the investigation sought to establish and what it delivered. | the claim ids and hypotheses |
| `results` | What was established: the principal results, each with its key number, its uncertainty and its status in words. | the claim ids, hypotheses and the evidence ids that hold the numbers |
| `open_items` | What was not established, and why each matters to the decision. | the contradictions, uncertainties and open hypotheses |
| `confidence` | How far the answer can be trusted, and the limitations that matter most. | the uncertainties and the claims they limit |
| `next_steps` | What would settle the open items. | the hypotheses and uncertainties they would settle |

- Write 250 to 500 words. Explain each technical term in a few words. Give no number without its meaning.
- The eight parts are one account, not eight answers. State the decision once. Write the results as sentences that a person who decides can read, not as a row of statuses.
- `frame` is a tag when the ledger has a Frame section; when it has none, tag these parts with the task ids.
- The conclusions are those of the ledger's Final status block, at the status each holds there (without one, the Hypotheses table and the closing summary). Keep each conclusion and each negation of Geo Oracle's closing summary. Every ledger hypothesis is in the summary, by its content. Add no conclusion; numbers and status words may be added.

### 4.2 Introduction

Four movements, each one or more paragraphs:
1. **The problem.** The geological question, why it matters to the decision, and the area.
2. **What was known or claimed before.** What published work and the project's own records say about the area and the question, cited. When neither the bundle nor your search gives any, say so in one sentence.
3. **The question.** The objective, and the decision that it serves.
4. **Purpose and scope.** What this investigation set out to do, the scale it works at, and what it does not try to do, each with the reason.

### 4.3 Geological setting

The regional and local geology that the reader needs for the argument, and no more: the basin, the structural and stratigraphic framework, and the units the study names, with the formations the well tops give and their depths or times. Take it from the literature evidence, the findings and published work you verified, each cited in the clause it supports. Write no result here. The setting is never empty: when no regional work ran, build it from what the investigation holds (the units in their order with the depths that the well tops give, the ages that the supplied data give, what the core shows the reservoir to be), and say in one sentence what is not known. When the data are synthetic, set `setting_title` to `Description of the synthetic model` and describe what the model represents, its dimensions and its known properties.

### 4.4 Data

Refer to the data table as `[[tab:data]]`. Then describe the data by kind, not file by file: what each data set covers, its quality as the quality control found it, its gaps, and what each gap prevents.

### 4.5 Methods

Write the methods as one workflow that a geoscientist could repeat, in the order the science needs (data preparation, well products, seismic quality control, well ties, horizons, faults, depth conversion), not the order in which the tasks ran. Open each stage with a heading block.

Say what was done at each stage and, when the findings say why, why the method suits these data. Cite the source of a method in the clause that names the step: the source of the method that ran, not a general text for a common operation. When a method was tried and set aside, say so in a sentence, with the reason.

Let the parameter table carry the settings. End the section with a table of the computations (step, method, key parameters and values, from `lit_read_bundle(part="parameters")`), grouped by stage, with the values copied as recorded. The prose then explains the choices and does not recite the values.

Every methods statement of the specialists must be carried whole: each method it names is cited at its stage, and each number it states is written: a setting in Methods (prose or table), an outcome (a value measured, a shift found, a count) in Results. Plan this in the outline (section 1, step 3). When many statements are lost at a render, the outline is at fault, not the wording: go back to the stages, give each lost statement its stage, and write that stage again as one procedure with its numbers in the table. Do not add a paragraph for each lost statement at the end of the section.

Methods describe procedures. A measured value or an outcome ("the panel failed", "the median throw is ...") is a result and belongs in Results. A method with no published source prints a star; write nothing else for it. If a methods statement was set aside, say in one sentence which results depend on it and that the record of the method is withheld.

### 4.6 Results

**Orient the reader first.** Before the first result, say where things are, as far as the record shows: the wells, the features that the results name, and how each mapped level relates to the level the decision is about. Use the map that shows them together when the record has one. Where the record does not show a position or a relation, say so there: it is a limit of the result (`lit-report-examples`, section 12).

Results say what was found, one subject to a subsection, under a heading block. Open each subsection with its point: what was found about the rock, not what happened to a table or a tool. Then give the evidence with its numbers, and read the figure in the sentence that uses it (section 7). Where the supplied material claimed or implied something, set the finding against it: what was believed, what was found, and the one reason for the difference. When two supplied records disagree, the disagreement is the finding: write neither of them as shown. Give a limitation where one changes how the result can be used, with what it changes; a result with no such limit needs no sentence about limits.

A measurement carries its uncertainty, range or spread ("1250 ± 6 ms"); when none was computed, say so. Status words belong to interpretations and hypotheses, not to a measurement.

A claim that the argument needs has its home in a subsection. A claim that it does not need goes in one labelled table at the end of Results (finding, status), tagged with those claims and introduced by one sentence. Never close Results with a run of one-sentence paragraphs, one for each claim left over.

Every claim that is supported, partially supported, unresolved or contradicted is in a subsection or in that table, with its status in words; a superseded claim is written as superseded, with the claim that replaced it. Nothing about how the report was made belongs here (which figures were left out, what was not supplied to you). Diagnostic products that were not released go under one heading, "Diagnostic products not released", with the reason they are shown in its first sentence.

### 4.7 Discussion

Write the discussion as an argument, in this order:
1. **What the results mean together.** Which result depends on which, where they agree, where they disagree, and what follows for the question. Geo Oracle's Synthesis section is the source: write its reading into this argument, and tag those paragraphs `synthesis`. Do not go through the results again one by one: end on the picture that the evidence favours, at its status, and on what the reader cannot yet be told.
2. **The rival readings.** For each hypothesis and each claim with a recorded alternative: the competing readings, the evidence for and against each, the test that would decide, and whether it ran. Give them in a table block with the columns reading, evidence for, evidence against and deciding test. In the text, discuss those that matter to the decision, and say which the evidence favours.
3. **The contradictions** the ledger preserves, each with what would resolve it, weighed in prose: which of them the decision turns on, and why. Sentences of one form in a row are a list.
4. **Comparison with published work,** cited. It gives context and never changes a conclusion. Where it differs from a result, say so and say what would explain it. With no published work, leave this part out.
5. **Limitations.** What the data and methods could not do, each with the conclusion it can change and the conclusions it leaves standing.

### 4.8 Conclusions

Short, and in this order:
1. The answer to the objective, at the status the Final status block gives it, with the real reason for it. If the question is not established, give the reason that matters most, not the easiest to state.
2. What each hypothesis came to, as a finding ("the closure on the mapped reflector is not a validated trap"), never its own words with a status after them.
3. What would settle each open item, the most decisive first.
4. A heading block "Record of the investigation", then one paragraph: the work that did not run or ran again, and the records that were withheld, a sentence each. Tag the paragraph with the ids the ledger's Tasks table holds for those runs.

Say each thing once.

An execution event (work cut off, a dropped connection, work run again) is not a scientific limitation: it goes in the record paragraph only. A withheld record is a fact about the provenance of a result: say it once, where the method or the result is, without saying why.

## 5. Paragraphs and sentences

- **One point for each paragraph, stated in its first sentence.** The rest is its evidence and what it means.
- Keep sentences under 30 words.
- **Say a thing once.** State a limitation where it first matters; after that, refer to it in a few words or not at all. A limit that applies to every result (an unconfirmed datum, synthetic data) is stated once, in the setting or the data, with what it covers; the results then do not repeat it.

## 6. Terms, numbers and status words

- **Define each term that is not standard geoscience usage at first use, in a clause,** and use one term for one thing. The names that the investigation coined need this most (a label for a reflector with no formation name, a name for a test): the reader meets them nowhere else. Also give them one place: end the Introduction with a labelled table, "Terms used in this report" (term, meaning), for each coined name, abbreviation and test name that the report uses, and refer to it there. Expand each abbreviation at first use. A code that a specialist's tool gives a test (a letter and a number) means nothing to the reader: write what the test checks ("the check that both sides of each fault cut the horizon"), and use the code after that only if you must.
- **Ledger labels are tags, never text.** Write a hypothesis by its content and status (never "H2 is contradicted", nor "(H2)" after the sentence), a contradiction by its content, an uncertainty by its cause and effect.
- **Horizons and markers keep their names,** with the full name at first mention (the label, the formation name and the word horizon), then the short name.
- **Numbers:** copy each number and unit as the ledger, the findings or the records give it. When one quantity has two values in the report (a depth at the well and the median of its map), say what each is where the second appears. Do not round, convert or average, except as a derived value (`lit-investigation-report`, section 2).
- **Status words.** A ledger hypothesis is untested, weakly constrained, partially constrained, supported, strongly supported, contradicted, unresolved or superseded. A claim is proposed, supported, partially supported, contradicted, unresolved or superseded. The objective is answered, partly answered or not answerable. Use the word the ledger or the findings give. Certainty comes only from these words: use no intensifier ("clearly", "robustly", "strongly suggests") that raises a status.
- **No platform words.** Write of the investigation, not of a person or a request ("the investigation sought to address", not "the requester asked" or "the request asked"; "the investigation ended before ...", not "the requester stopped ..."). Write "the investigation record", not "the bundle"; "withheld", not "set aside". Do not write "gateway", "HOTL", "specialist call", "iteration", "job", "turn", "session", "ledger" or "the tool". Name the specialists by discipline. What happened to the software is not geology: a table that did not register, a refused run and a product "admitted" or not are written by their effect on what is known, and the event itself goes in the record paragraph.

## 7. Figures, tables and captions

- **A figure is part of a sentence.** Name it and say what the reader should see: "As [[fig:thickness]] shows, the sand thickens to the north-east, from 18 m at the first well to 31 m at the second."
- **Choose figures.** Use a figure only when a conclusion rests on what it shows, and place it after the paragraph that first uses it. When two show nearly the same thing, use one. Most reports need about a dozen or fewer. A sentence that only names figures ("Figures 8 and 9 show two more sections") reads nothing: when you cannot say what the reader should see in a figure, leave the figure out. The unused-figure warning is no reason to place one.
- **A caption has three parts:** what is shown, with its units and its slice, horizon or line; what the reader should see, in one sentence; and what it rests on. Start from the specialist's caption, and check its numbers against the findings.

## 8. The renderer's notes

The renderer answers with errors and warnings. Read them as a proofreader's notes: each points at a place where the report fails a reader or the record. It is not a list of things to add.

- **Find the cause first,** from all the notes of a render together. Most come from the outline: a claim with no home, a method at the wrong stage.
- **Revise the paragraph the note points at,** so that the item is part of the argument. Do not add a sentence or a paragraph whose only purpose is to make the note go away: a report made of such additions passes every check and cannot be read.
- **A note may be wrong for your draft.** When your reading of the paragraph shows that a warning does not apply, leave the text and say why in your `limitations`.

## 9. Before you render

Read the stored draft once more as its reader (section 1, steps 5 and 6) and revise it. Then check it against the record and against what the renderer refuses (`lit-investigation-report`, section 4): each item at its home, each measurement with its uncertainty, and no label, id, platform word or misplaced execution event in the text.

<!-- end of skill: lit-report-writing -->
