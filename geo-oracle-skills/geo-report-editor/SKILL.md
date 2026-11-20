---
name: geo-report-editor
description: How Geo Oracle acts as the editor in chief of the formal investigation report. Load when a formal report is requested, before request_report, and each time a draft or a reading comes back, after geo-investigation-report. It covers the commission to the writer, the reader's questions and your private answer key, starting the reader, how to read the reader's report and the writer's outline, the editorial reading of the draft, the record check against the ledger, how to write few ranked notes that ask for revision and not addition, what the writer may and may not see of the reader's report, the verdict, and the later rounds.
---

# Editing the investigation report

You ran the investigation, so you know better than anyone what it found. That is why you cannot judge its report alone: you will read into the text what you already know. An editor in chief does two things you can do well. You say what the report must achieve before it is written. And you judge the draft by what it does to a reader who was not there, using a reader who was not there.

The report is good when a person who must decide, and who did not see the work, can read it once and say what was asked, what was found, how far it goes and what would change it. Passing the renderer's checks is necessary. It is not that.

`geo-investigation-report` gives the mechanics: the close, `request_report`, `report_draft`, the rounds and the delivery. This skill gives the editing.

## 1. The commission

Before the writer starts, write the commission. It gives the writer a spine, so that it does not have to build one from every item of the record. Write it from your Synthesis and Final status. It adds no finding.

1. **The reader:** who will read the report and what they must decide.
2. **The question** the investigation sought to address.
3. **The answer,** in one short paragraph, at the status the ledger gives it: what the investigation sought to establish, what it established, and what it did not.
4. **The storyline:** the few results that carry the answer, in the order a reader needs them, and for each the figure or table that shows it best.
5. **What was believed before,** when the supplied material claimed or implied something that the investigation tested: the report sets its results against it.
6. **What to leave short:** work that the answer does not rest on.

Give it as `commission` in your first `request_report` call. The writer gets it with every round, as a guide to order and weight and never as a source: put no result in it that the ledger does not hold.

## 2. The reader's questions and your answer key

A reader who was not in the investigation reads the draft and answers your questions from it alone (section 3). Write the questions before the first draft comes back, so that the draft cannot shape them.

- **Start from these four, and change them freely.** They are suggestions, not a form:
  1. What did the investigation establish?
  2. What is interpreted, and what stays open?
  3. Is the decision that the investigation was to support, supported? Why, or why not?
  4. What would change the answer?
- **Add the questions this investigation needs.** You know where a reader is most likely to go wrong: the result that looks stronger than it is, the two things that are easy to confuse, the limitation that changes less than it seems. Ask about those.
- **Ask open questions.** "What does the report conclude about the seal, and on what evidence?" tests the report. "Is the seal unproven because the pressure data are missing?" gives the reader the answer. Never put the answer, or a hint of it, in a question.
- **Keep them few:** six to ten.

Then write **your answer key**: for each question, the answer that a reader should reach, in two or three sentences, from your ledger. Keep the questions and the key in `report_key.md` in the ledger's folder (`<home>/investigation/`, or `<home>/investigation/<investigation_id>/`), so that they survive a compaction. Do not put them in the ledger: it is frozen. Do not send the key to anyone.

## 3. The reader

When a draft has rendered and passed its release checks, start the reader: a fresh literature session that gets the report and your questions, and nothing else. It does not get the ledger, the record, your key, the commission or the writer's outline. Call `request_report(investigation_id, reader_questions=[...])`, then collect the reading with `get_specialist_result`. A reading costs no review round. The report does not depend on it: if the reading fails, review the draft without it, and say when you deliver that no outside reading was made.

One reader serves all the rounds of one report. In a later round it gets the revised report, the same questions, and one or two new questions that it has not seen. It remembers the earlier version, so the new questions are the part of its reading that is still a first reading.

## 4. Read what comes back

Read in this order.

**1. The outline.** `report_draft` gives `outline`: the first sentence of each paragraph, section by section (a long one continues with `part='outline'`, as `outline_next` says). Read only that, first. Does it tell the story of your commission? Is the answer on its first page? Does a section repeat another? Is a result missing from the storyline? Problems you can see here are problems of structure. No note about a sentence will repair them.

**2. The reader's report, against your key.** For each question, compare the reader's answer with your key:
- **The same, and found clearly:** the report works there.
- **The same, but inferred from several places:** the report holds it and does not say it. Where should it be said?
- **Different:** the report misleads. Find the passage that produced the wrong answer.
- **Not found:** the report leaves it out, or hides it.

Then read what the reader took from the report in its own five sentences. If that is not your answer, the report has failed at its main task, whatever else is right. Then read where it stopped, the terms it was not given, what it did not find and what was repeated.

**3. The draft itself,** with `report_draft`, as an editor. These are questions to read with, not a form to fill in: note only what a reader would lose by.
1. Is the answer on the first page, at its true strength?
2. Does the report say what it does not try to do, once?
3. Is there a reason for each choice of method, and is anything that was tried and dropped reported?
4. Is each term and code explained where it first appears?
5. Does each figure have a sentence that reads it? Are there figures that no argument needs, or results with no figure that need one?
6. Are the numbers there, with their uncertainties?
7. Does the Discussion argue: results set against each other, rival readings weighed, what would decide?
8. Does the conclusion give the real reason, or an easier one?
9. Is anything said twice?
10. Is the weight right: most space on what the answer rests on?

**4. The record check.** The editorial reading does not replace the check against your ledger. Each of these is a contradiction, and each goes back (the list in `geo-investigation-report`, section 5, is the full one): a conclusion stronger or weaker than its ledger status; a claim, number or source that the record does not hold; an open hypothesis or contradiction shown as settled; a lost limitation or uncertainty; a method described differently from what ran; an execution event written as a scientific limitation; published work used to change a conclusion; failed release checks.

**5. The renderer's warnings.** Read every page. A warning about the record (a number not in its items, a methods statement that lost its numbers, strong wording) goes back, unless your reading shows that it does not apply. A warning about form (a figure not used, an item not tagged, a long sentence) is a question for your judgement: does a reader lose something? If yes, it becomes part of a note about the passage. If no, leave it, and say why when you deliver.

## 5. The notes

A note is an instruction to revise a passage for a reader. It never asks for a sentence appended so that a check passes: what is missing goes where it belongs.

- **Every contradiction with the record goes back,** each as its own `record` note. They are not optional (`request_report` takes 20 notes in all). A record note may ask for what is missing: say where it belongs in the argument.
- **Editorial notes are few and ranked.** Send at most five, the most serious first. Structure before sections, sections before paragraphs, paragraphs before sentences. If the outline is wrong, the first note says so, and notes about sentences can wait.
- **Each note names:** the place (section and paragraph); what a reader gets from it now; what the reader should get; and its kind: `revise` (the writing), `record` (it contradicts the ledger), or `cut`.
- **Ask for cuts.** Say what to remove, join or shorten. A report gets better by losing what the reader does not need.
- **Write the note about the report, not about the question.** Not "the reader could not answer question 3". Write: "A reader finishes section 5 believing that the sand is continuous across the whole area. The report must make plain, where continuity is first described, that it is shown only north of the fault."
- **Give the writer the reader's own words** where they show the problem: what the reader took from the report, where it stopped, the terms it was not given, what it did not find, what was repeated. Copy them as the reader wrote them, and leave out any line that restates one of your questions.
- **Never give the writer the questions, the reader's answers as answers, or your key.** A writer who has the questions writes to them, and the next reading then tests nothing.
- Do not ask the writer to add ids to the text. Do not write the passage for the writer.

Send the notes with `request_report(investigation_id, review_notes=[...])`, one note for each entry. Each note is one string that starts with its kind. The three calls of a report, with invented values (`commission` is one string; `reader_questions` and `review_notes` are lists of strings, never in the same call):

```
request_report(investigation_id="INV-<slug>-YYYYMMDDTHHMM", person_words="<their request, word for word>",
  executive_summary="<your closing summary, word for word>",
  commission="Reader: the board that decides on a well. Question: ... Answer: ... (partly answered). Storyline: ... Believed before: ... Leave short: ...")
request_report(investigation_id="INV-<slug>-YYYYMMDDTHHMM", reader_questions=[
  "What did the investigation establish?", "What does the report conclude about the seal, and on what evidence?"])
request_report(investigation_id="INV-<slug>-YYYYMMDDTHHMM", review_notes=[
  "record: Results, paragraph 3 calls the closure a trap. The ledger keeps the trap hypothesis unresolved. Write it at that status.",
  "revise: Discussion, paragraph 2. A reader gets two readings of the southern reflector but not the test that decides. Give the test where the readings are weighed."])
```

## 6. The verdict

After each round, decide:

- **Accept:** your record check finds no contradiction, the release checks passed, the reader's answers agree with your key, in substance and not in wording, on the questions that bear on the decision, and what the reader took from the report is your answer in substance. Deliver it.
- **Revise:** a contradiction remains, or the reader misunderstood something that bears on the decision, while a round is left. Do not spend a round on a difference a reader would not act on.
- **A revision that reads worse:** compare each revised round with the rounds before it. Read each draft with `report_draft` (`review_round` gives an earlier round). A revision is worse in these cases:
  - your record check finds a contradiction, an unsupported statement or a lost result that the earlier round did not have;
  - on a question that the reader has not seen before, the reader does not reach your key on a point that the earlier round stated clearly.

  Do not judge this from the questions that the reader answered in an earlier round. The reader remembers the earlier version, so those answers do not show what a new reader gets. If a round is left, send a note that names what the revision lost: a `record` note for a lost result or a contradiction, a `revise` note for a point that a reader no longer reaches. If no round is left, deliver the best round that passed its release checks: `report_draft` gives its link. If only the last round passed, deliver it with your objections. Tell the person which round you delivered, and why.
- **After the last round:** deliver the last report that passed its release checks, unless an earlier round reads better (see above), with your remaining objections, each in one line. If none passed, tell the person that the report could not be released, and why.

Do not hold a report back for style when the reader understood it. Do not accept a report that passed every check and that the reader misunderstood.

When you deliver, add one line on the reading: that an outside reader read the report, and whether its understanding matched the investigation's answer. If you deliver an earlier round, say which round the reader's last reading was of.

<!-- end of skill: geo-report-editor -->
