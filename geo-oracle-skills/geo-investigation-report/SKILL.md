---
name: geo-investigation-report
description: How Geo Oracle closes an investigation and, when the person asks, gets the formal investigation report written and reviews it. Use at the close of every investigation (after the final synthesis and the closing checkpoint), and whenever the person asks for a report, a formal report, a write-up or a PDF of a closed investigation. Covers the Final status block, the ledger download, the closing sentence, request_report, report_draft, the three review rounds and the delivery.
---

# The close and the investigation report

The person gets the ledger at the close of every investigation. The formal report is optional: the literature specialist writes it only when the person asks. You review the draft against your ledger before the person sees it.

## 1. The close

Do these steps in this order, after the final synthesis (`geo-synthesis-audit`) and the closing checkpoint:

1. **Write the Final status block** at the end of the ledger (format in section 2), then save the ledger.
2. **Give the ledger as a download.** The closing checkpoint's reply gives `ledger_download`, a `sandbox_artifacts` block with the path on record. Put it, unchanged, near the end of your message, just before the closing sentence (step 3). If the reply has no `ledger_download`, write this block with the path you gave as `ledger_path`. Do not build the path again from `pwd`. Write the path plainly: never escape a slash (`\/`), because an escaped path does not download:

   ````
   ```sandbox_artifacts
   [Investigation ledger](<home>/investigation/ledger.md)
   ```
   ````

   If one chat holds several investigations, use `<home>/investigation/<investigation_id>/ledger.md`. The ledger opens with `# Investigation <investigation_id>`. The gateway takes the ledger into a report only when it names the investigation.
3. **End with one closing sentence that offers the report and shows how to ask for it**, for example: "If you would like a formal report of this investigation, reply 'yes', or ask for it in your own words, for example: 'Write the formal report.'" Offer it only when this investigation filed a closing checkpoint. An investigation with no checkpoint (one specialist call, no human-on-the-loop mode) cannot have a formal report, so end without the offer. Put nothing after the closing sentence. The gateway reads the last 500 characters of your message to take a short 'yes' as the answer to this offer.

Do not call `ask_user_question` at the close. A question blocks the turn until the person answers. The person may not want a report, and the turn must end.

**After the close, the ledger is frozen.** Do not edit it. The report prints it as its appendix with its SHA-256, so any edit after the close breaks the match between the report and the ledger. If the person asks for more work instead of a report, the investigation opens again. Work through the human-on-the-loop rules as usual, add to the ledger, and close again with a new Final status block and a new download.

## 2. The Final status block

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

Copy each status from the ledger. The block summarises the ledger and adds no new finding. Its hypotheses, answer and pre-mortem are the ones the closing checkpoint recorded (`hypotheses`, `objective_check`, `pre_mortem`), word for word.

## 3. When the person asks for a report

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

## 4. Review the draft

Call `report_draft(investigation_id)`. It gives:

- each paragraph (numbered from 1 within its section), with the ledger ids it rests on (the hidden tags);
- the release checks;
- the warnings (on the first page, with their total: drift first, then the content warnings, then style);
- the literature references as they print (from what the literature evidence recorded);
- the download link (only when the release checks passed);
- the rounds left.

A long draft comes in pages: call again with `offset=next_offset` until no `next_offset` comes back.

If `report_draft` gives a `release_note`, or the drafting run carries the warning `Process audit: skills not opened: lit-investigation-report`, the writer drafted without its procedure. Do not deliver it. Send the round back with a review note that asks it to read the skill and redraft from every part of the bundle.

If the drafting run failed and rendered nothing, call `request_report(investigation_id)` again. That round runs again and costs no round. A round runs at most three times. After the third failure, read the run's message, and tell the person that the report could not be produced, and why.

Compare each paragraph with your ledger. Each of these is a contradiction:

- a conclusion stated more strongly or more weakly than its ledger status;
- a claim, number or source that the ledger and the findings do not hold, including a reference whose author, year or title is not the source the literature specialist found;
- a hypothesis or contradiction the ledger keeps open, shown as settled;
- a limitation or material uncertainty that is lost;
- a methods drift warning that changes what was run;
- release checks that failed.

These are not contradictions: a different word order, shorter sentences, a different paragraph order, or a plainer word. The writer may make light edits for flow.

The text may contain no ids; the hidden tags carry them. Do not ask the writer to add ids to the text.

## 5. Send it back, or deliver it

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
