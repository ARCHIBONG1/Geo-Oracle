---
name: lit-synthesis-report
description: How the literature specialist synthesises findings across sources. Covers handling conflicting literature, careful consensus language, the standalone literature report for direct conversations, and the final self-check before returning any result. Use when combining evidence from several sources into conclusions, when sources disagree, before returning any result to Geo Oracle, and when a person asks for a literature report directly.
---

# Synthesis and report

A synthesis weighs evidence question by question. It says what the evidence supports, what contradicts it, what is only an analogy, and what remains uncertain.

## 1. Synthesis rules

- **Organise by research question,** not by paper.
- **For each conclusion, state:**
  - the supporting evidence and the number of independent lines;
  - the contradicting evidence;
  - which support is analogy only;
  - the remaining uncertainty;
  - its relevance to the study area.
- **Keep the author's interpretation and your inference separate** in the wording: "Author et al. interpret…" versus "Taken together, the reviewed sources suggest…".
- **Scope conclusions to what was reviewed.** A conclusion is only as broad as the sources behind it.

## 2. Conflicting literature

When sources disagree materially:
1. Identify the competing interpretations and the sources that support each.
2. Identify the observations each interpretation rests on, and its key assumptions.
3. Check whether the evidence on each side is independent.
4. Find later work that supports or challenges the earlier interpretations.
5. State what remains unresolved, and what new data would discriminate between the interpretations.

Record each material disagreement as a `contradiction`. Do not average interpretations into a false consensus, and do not dismiss a minority view without examining its evidence.

## 3. Consensus language

| Use | Only when |
|---|---|
| "The reviewed sources consistently describe…" | All the relevant sources you reviewed agree |
| "Multiple independent studies interpret…" | At least two independent lines of evidence exist |
| "The literature appears broadly consistent regarding…" | Most sources agree and none contradicts materially |
| "The reviewed sources disagree regarding…" | There is a material conflict |
| "The literature does not establish…" | The evidence is absent or insufficient |

Never write "the scientific community agrees" unless the search was broad enough to justify it, which is rare. The number of papers is not a measure of correctness.

## 4. Standalone report (direct conversations only)

When a person asks directly and there is no output contract, write:

1. **Research question(s)**
2. **Search scope:** the geographic and geological scope, the date range, the tools and databases searched (from the search log), the key terms, and what could not be searched or accessed.
3. **Sources reviewed:** a table with source, year, type, access level, relevance and main contribution.
4. **Geological framework:** tectonic, stratigraphic, depositional and structural, as relevant.
5. **Resource-system evidence:** each element separately, e.g. the petroleum-system elements.
6. **Hydrocarbon or resource occurrence evidence**, where relevant.
7. **Major interpretations**, and **conflicting interpretations**.
8. **Evidence gaps and limitations.**
9. **Testable implications** for project data: seismic, wells, tops, logs, samples.
10. **Priority sources** for further work.
11. **Conclusion:** what the literature establishes, what it suggests, and what remains unresolved.

Cite verified sources inline as (Author et al. year, DOI), with the access level wherever it is not full text. Give no prospect rankings, scores, or drilling or commercial recommendations.

## 5. Final self-check (before returning anything)

**Sources and citations**
- Is every cited source resolved to a DOI or OpenAlex ID, or marked `UNVERIFIED`?
- Does every source carry an access level? Does anything reported exceed what that level supports?
- Were retraction checks done for the key sources?

**Findings**
- Is every finding labelled with its evidence class and its geographic relevance? Has any basin-scale or analogue evidence been presented as study-area evidence?
- Do quantitative values have units, method, sample location and a location in the source?
- Is each author's interpretation kept separate from the observation it rests on?

**Search**
- Did I search specifically for disagreement on the important interpretations?
- Are the searches I report exactly those in my search log?
- Are what I could not search or access, and why the search stopped, stated in `limitations`?

**Scope and format**
- Did I avoid prospectivity rankings and recommendations?
- For gateway tasks: is the output a single JSON object that matches the contract, with no extra keys?

Fix every failure before returning. If a failure cannot be fixed, disclose it.
