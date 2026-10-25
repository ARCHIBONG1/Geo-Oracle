---
name: geo-investigation-report
description: How Geo Oracle closes an investigation and, when the person asks, gets the formal investigation report written and reviews it. Use at the close of every investigation (after the final synthesis and the closing checkpoint), and whenever the person asks for a report, a formal report, a write-up or a PDF of a closed investigation. Covers the Final status block, the ledger download, the closing sentence, request_report, report_draft, the three review rounds and the delivery.
---

# The close and the investigation report

The person gets the ledger at the close of every investigation. The formal report is optional: the literature specialist writes it only when the person asks. You review the draft against your ledger before the person sees it.

## 1. The close

Do these steps in this order, after the final synthesis (`geo-synthesis-audit`) and the closing checkpoint:

1. **Write the Final status block** at the end of the ledger (format in section 2), then save the ledger.
2. **Give the ledger as a download.** Put this block at the end of your message:

   ````
   ```sandbox_artifacts
   [Investigation ledger](/investigation/ledger.md)
   ```
   ````

   Use the path you saved it at (`/investigation/<investigation_id>/ledger.md` when one chat holds several investigations).
3. **End with one closing sentence**, for example: "If you would like a formal report of this investigation, just say so."

Do not call `ask_user_question` at the close. A question blocks the turn until the person answers. The person may not want a report, and the turn must end.

**After the close, the ledger is frozen.** Do not edit it. The report prints it as its appendix with its SHA-256, so any edit after the close breaks the match between the report and the ledger. If the person asks for more work instead of a report, the investigation opens again: work through the human-on-the-loop rules as usual, add to the ledger, and close again with a new Final status block and a new download.

## 2. The Final status block

```markdown
## Final status
| Field | Content |
|-------|---------|
| Closed at | <time> |
| Decision supported | the decision the person asked about |
| Conclusion | one line, with the H# and claim ids it rests on and their status |
| Hypotheses | each H# with its final status |
| Preserved contradictions | each X# that stays open, or "none" |
| Material uncertainties | each U# that can change the conclusion |
| Not done | analyses that failed or did not run, with the reason |
| Runs used | n of N specialist runs |
```

Copy each status from the ledger. The block summarises the ledger; it adds no new finding.

## 3. When the person asks for a report

Start a report only when the person's latest message asks for one. Do not start one on your own.

Call `request_report` with:

- `investigation_id`;
- `person_words`: the person's request, word for word, copied from their message;
- `executive_summary`: the summary you gave the person at the close, word for word.

The gateway checks that the investigation is closed and that the words are in the person's latest message. It freezes the investigation into a bundle (your ledger from your sandbox, the specialists' results, figures and data) and starts the literature specialist. The report runs do not count against the work ceiling.

**If it refuses**, tell the person why in one or two sentences. An open item (a checkpoint not filed, more work approved, a run still running, evidence set aside and not corrected) must be closed first. Do not work around a refusal.

Then tell the person that the report is in progress, and collect the run with `get_specialist_result(job_id, wait_seconds=...)`. The draft takes some minutes.

## 4. Review the draft

Call `report_draft(investigation_id)`. It gives each paragraph with the ledger ids it rests on (the hidden tags), the release checks, the drift and style warnings, the download link and the rounds left. A long draft comes in pages: call again with `offset=next_offset` until no `next_offset` comes back.

Compare each paragraph with your ledger. Each of these is a contradiction:

- a conclusion stated more strongly or more weakly than its ledger status;
- a claim, number or source that the ledger and the findings do not hold;
- a hypothesis or contradiction the ledger keeps open, shown as settled;
- a limitation or material uncertainty that is lost;
- a methods drift warning that changes what was run;
- release checks that failed.

These are not contradictions: a different word order, shorter sentences, a different paragraph order, or a plainer word. The writer may make light edits for flow.

The text may contain no ids; the hidden tags carry them. Do not ask the writer to add ids to the text.

## 5. Send it back, or deliver it

- **Contradictions found:** call `request_report(investigation_id, review_notes=[...])`, one contradiction for each note. Name the paragraph (section and number), what it says, and what the ledger holds. Then collect the run and review again.
- **No contradictions:** deliver it.
- **Three rounds are the limit.** After the third round, deliver the last report with your remaining objections.

Deliver it in a short message:

1. the download link from `report_draft`, as a normal link (not a `sandbox_artifacts` block: the PDF is not in your sandbox);
2. one line on what the report holds: the title, the page count, and that the ledger is attached and printed as the appendix;
3. your remaining objections, if any, each as one line;
4. a note that the report and its bundle are deleted with this chat.

Do not edit the report yourself, and do not ask a closing question.

<!-- end of skill: geo-investigation-report -->
