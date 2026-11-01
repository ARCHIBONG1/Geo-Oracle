---
name: geo-investigation-report
description: How Geo Oracle closes an investigation and, when the person asks, gets the formal investigation report written and reviews it. Use at the close of every investigation (when the closing checkpoint is filed), and whenever the person asks for a report, a formal report, a write-up or a PDF of a closed investigation. Covers the Synthesis section, the Final status block, the synthesis given to the person, the ledger download, the closing sentence, request_report, report_draft, the three review rounds and the delivery.
---

# The close and the investigation report

The person gets the ledger at the close of every investigation. The formal report is optional: the literature specialist writes it only when the person asks. You review the draft against your ledger before the person sees it.

## 1. The close

The close starts when the closing checkpoint is filed: its reply normally gives `ledger_download`. Do not write the Synthesis or the Final status before that. The gateway can send the work on after a checkpoint, and blocks written before the last results describe an earlier state. Do these steps in this order:

1. **Record every task** in the ledger: each task that the investigation ran, by its full task id, with its result.
2. **Write the Synthesis section** (format in section 2) and then **the Final status block** (format in section 3), from the state of the work now, as the last two sections of the ledger. If you wrote either one earlier, replace it: remove the old one. Write nothing after the Final status block. Then save the ledger.
3. **Give the person the synthesis** in your closing message, in plain words, before the ledger download. Give it at every close, also when the person does not ask for a report. Write it from the Synthesis section: how the results combine, what the key figures show, the provenance, the open questions with their competing readings and tests, and what was not in scope. Do not use ledger labels or ids in it, not even as citations in brackets: write each hypothesis and uncertainty by its content. The ledger that the person downloads carries the ids. This rule replaces the citation rule of your prompt for this one message.
4. **Give the ledger as a download.** The closing checkpoint's reply gives `ledger_download`, a `sandbox_artifacts` block with the path on record. Put it, unchanged, near the end of your message, just before the closing sentence (step 5). If the reply has no `ledger_download`, write this block with the path you gave as `ledger_path`. Do not build the path again from `pwd`. Write the path plainly: never escape a slash (`\/`), because an escaped path does not download:

   ````
   ```sandbox_artifacts
   [Investigation ledger](<home>/investigation/ledger.md)
   ```
   ````

   If one chat holds several investigations, use `<home>/investigation/<investigation_id>/ledger.md`. The ledger opens with `# Investigation <investigation_id>`. The gateway takes the ledger into a report only when it names the investigation.
5. **End with one closing sentence that offers the report and shows how to ask for it**, for example: "If you would like a formal report of this investigation, reply 'yes', or ask for it in your own words, for example: 'Write the formal report.'" Offer it only when this investigation filed a closing checkpoint. An investigation with no checkpoint (one specialist call, no human-on-the-loop mode) cannot have a formal report, so end without the offer. Put nothing after the closing sentence. The gateway reads the last 500 characters of your message to take a short 'yes' as the answer to this offer.

The report writer reads the Synthesis section and carries it into the Discussion and the Conclusions. A ledger without it gives a report without your reading of the results.

Do not call `ask_user_question` at the close. A question blocks the turn until the person answers. The person may not want a report, and the turn must end.

**After the close, the ledger is frozen.** Do not edit it, with one exception: when `request_report` refuses because the ledger does not yet describe the end of the work, complete it as the refusal says (record the tasks it names, and put the Synthesis and the Final status at the end), then call again with the same words. The report prints it as its appendix with its SHA-256, so any edit after the close breaks the match between the report and the ledger. If the person asks for more work instead of a report, the investigation opens again. Work through the human-on-the-loop rules as usual, add to the ledger, and close again with a new Final status block and a new download.

## 2. The Synthesis section

```markdown
## Synthesis
### How the results combine
### What the key figures show
### Provenance
### Open questions
### Not in scope
```

Write each part in prose. Put the ledger ids each statement rests on in brackets at its end (H#, claim ids, X#, U#), so the report can rest on them.

- **How the results combine:** how the results of the specialists fit together, and where they disagree. Say which result depends on which.
- **What the key figures show:** for each figure that carries a conclusion, what it shows in words: the structure, the trend, the anomaly, and its scale.
- **Provenance:** which data and records the conclusions rest on, which records were withheld or set aside, and what that changes.
- **Open questions:** for each open hypothesis, contradiction or material uncertainty: the competing readings, the evidence for each, and the test or data that would decide between them.
- **Not in scope:** what the person excluded or the data could not support, and why.

The Synthesis adds no finding. Each statement comes from the ledger and the results, at the status they hold.

## 3. The Final status block

```markdown
## Final status
| Field | Content |
|-------|---------|
| Closed at | <time> |
| Decision supported | the decision the person asked about |
| Conclusion | one line, with the H# and claim ids it rests on and their status |
| Hypotheses | each H# with its final status |
| Answer to the objective | the closing checkpoint's objective_check, with its status |
| Pre-mortem | each reason with what covered it, when the risk specialist ran |
| Preserved contradictions | each X# that stays open, or "none" |
| Material uncertainties | each U# that can change the conclusion |
| Not done | analyses that failed or did not run, with the reason |
| Runs used | n of N specialist runs |
```

Copy each status from the ledger and the Synthesis. The block summarises the ledger and adds no new finding. Its hypotheses, answer and pre-mortem are the ones the closing checkpoint recorded (`hypotheses`, `objective_check`, `pre_mortem`), word for word.

## 4. When the person asks for a report

Start a report only when the person's latest message asks for one. Do not start one on your own.

Call `request_report` with:

- `investigation_id`;
- `person_words`: the person's words, copied exactly from their latest message:
  - a short yes to your offer ("yes", "yes please", "proceed", "go ahead"): their whole reply, as they wrote it. It counts only as the answer to the report offer in your previous message;
  - otherwise their whole sentence that asks for the report, word for word ("Please write the formal report.", "Could you write it up as a PDF?"), not a part of it;
- `executive_summary`: the summary you gave the person at the close, word for word.

The gateway checks that the investigation is closed and that the person's latest message in this chat asks for the report. The request must be a message the person typed, not an answer to `ask_user_question`.

**If the words are refused**, follow the reason the gateway gives:

- *too short, or does not ask for a report* (for example "yes" when your previous message made no offer, a question about the report, or a sentence with a "no" or "not" in it): ask the person in plain text whether they want the formal report. Show them how to ask, for example "Write the formal report." Then wait for their reply.
- *declines* ("no", "not now", "later"): do not start a report.
- *cannot read the chats*: call again in a few seconds.

Never write the request for the person. Never quote words they did not write. The call freezes the investigation into a bundle (your ledger from your sandbox, the specialists' results, figures and data) and starts the literature specialist. The report runs do not count against the work ceiling.

**After more work.** If the person asked for more work after a report and you closed again, call `request_report` again with `person_words` when they ask for a new report. A new report starts, and the gateway keeps the earlier one.

**If it refuses**, tell the person why in one or two sentences. An open item (a checkpoint not filed, more work approved, a run still running, evidence set aside and not corrected) must be closed first. Do not work around a refusal.

**If the ledger was not found**, do not tell the person. Follow the `next_action`: find the ledger file you wrote (never one under `unpacked/`), check that it opens with `# Investigation <investigation_id>`, and call `request_report` again with `ledger_path` set to its absolute path and the same words. The ledger stays in your sandbox until the chat is deleted. Never write a new ledger: the report prints the ledger word for word, so a ledger written again from memory puts a false record in the report. If the reply has a `ledger_note`, tell the person what it says in one sentence.

Then tell the person that the report is in progress, and collect the run with `get_specialist_result(job_id, wait_seconds=...)`. The draft takes some minutes.

## 5. Review the draft

Call `report_draft(investigation_id)`. It gives:

- each paragraph (numbered from 1 within its section), with the ledger ids it rests on (the hidden tags); the headings, and each figure and table with its label;
- the release checks;
- the warnings (the first page, with their total and a count by kind: drift first, then the content warnings, then style; the rest with `part='warnings'` and the `offset` the reply gives);
- the `review_rule`, when there are drift or content warnings;
- the literature references as they print (from what the literature evidence recorded). A source the writer found and verified for the report is marked `found_for_report`;
- the download link (only when the release checks passed);
- the rounds left.

A long draft comes in pages: call again with `offset=next_offset` until no `next_offset` comes back.

If `report_draft` gives a `release_note`, or the drafting run carries the warning `Process audit: skills not opened: lit-investigation-report` or `... lit-report-writing`, the writer drafted without its procedure. Do not deliver it. Send the round back with a review note that asks it to read the skills and redraft from every part of the bundle.

**The warnings are review notes.** Read every page of the warnings. Send each drift and content warning back in your review notes, one subject per note, unless your review of the paragraph shows that it does not apply. When you deliver the report, say why each warning you did not send back does not apply. The style warnings are advice; send back only those that make a paragraph unclear. The process-audit items of a drafting run about findings, figures carried or evidence classes do not apply to a report: do not send them as notes.

**Read the run's `next_action` first.** If the writer's turn ended in an error after the round rendered, the `next_action` says so: the draft is on file. Review it with `report_draft`. Do not ask for the round again.

If the drafting run rendered nothing, the gateway starts the round's next run itself: the run's `next_action` names the new job. Collect it with `get_specialist_result` and do not call `request_report` for it. A round runs at most three times, and a run costs no round. If the `next_action` instead says to call `request_report(investigation_id)` again, do so. Do not tell the person that the report failed while a run is left. Only when the `next_action` says no run is left, or the gateway refuses another run, read the last run's message, and tell the person that the report could not be produced, and why.

Compare each paragraph with your ledger. Each of these is a contradiction:

- a conclusion stated more strongly or more weakly than its ledger status;
- a claim, number or source that the ledger and the findings do not hold, including a reference whose author, year or title is not the source the literature specialist found;
- a hypothesis or contradiction the ledger keeps open, shown as settled;
- a limitation or material uncertainty that is lost;
- a methods drift warning that changes what was run;
- an execution event (a run that was cut off, a dropped connection, a rerun) written as a scientific limitation, in the answer or beside a hypothesis; it belongs in the record paragraph at the end;
- a figure or table the text does not refer to (the warnings name them);
- a claim whose recorded alternative the report does not give (the task views in the bundle hold the alternatives);
- an executive summary part, or a ledger hypothesis, that the warnings name as missing from the summary;
- a measurement given without its uncertainty, or without the sentence that no uncertainty was computed;
- published work, found by the writer for the report, used to change a conclusion; the writer may use it only to compare and to give context;
- a part of your Synthesis that the Discussion or the Conclusions leave out;
- release checks that failed.

These are not contradictions: a different word order, shorter sentences, a different paragraph order, or a plainer word. The writer may make light edits for flow.

The text may contain no ids; the hidden tags carry them. Do not ask the writer to add ids to the text.

## 6. Send it back, or deliver it

- **Contradictions found:** call `request_report(investigation_id, review_notes=[...])`, one contradiction for each note. Name the paragraph (section and number), what it says, and what the ledger holds. Then collect the run and review again.
- **No contradictions, and the release checks passed:** deliver it.
- **Three rounds are the limit.** After the third round, deliver the last report with your remaining objections, but only if it passed its release checks. If it failed them, it has no download link: tell the person that it could not be released, and why.

Deliver it in a short message:

1. the download link from `report_draft`, as a normal link (not a `sandbox_artifacts` block: the PDF is not in your sandbox);
2. one line on what the report holds: the title, the page count, and that the ledger is attached and printed as the appendix;
3. your remaining objections, if any, each as one line;
4. a note that the report and its bundle are deleted with this chat.

Do not edit the report yourself, and do not ask a closing question.

<!-- end of skill: geo-investigation-report -->
