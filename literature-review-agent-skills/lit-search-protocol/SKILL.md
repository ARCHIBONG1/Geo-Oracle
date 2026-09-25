---
name: lit-search-protocol
description: Search planning and execution for the geological literature specialist. Covers term expansion (formation, historical and local names, translations), which tool to use for what (OpenAlex, Exa publication search, Tavily), iterative narrowing, citation chaining, challenge searches, author disambiguation, the search log, and when to stop. Use at the start of every literature task, when searching for papers by a named author, when results are thin, noisy or all from one group, and when deciding whether to keep searching.
---

# Search protocol

A literature search is only as good as its coverage, and coverage fails quietly. The usual causes are a wrong formation name, a sub-basin nobody searched, a disagreement published as a "Discussion", or a key regional report that no journal index holds. This protocol guards against each of these.

## 1. Plan before searching

For each research question, write down the following before running any search:

- **Concepts and their variants:**
  - current, historical and local names;
  - formation, group and member names;
  - sub-basins and structural provinces;
  - well names;
  - the local-language names of key terms.
- **Constraints:** the date range, if relevant, and the resource type.
- **Tools:** which tools you will use (section 2), and the evidence you expect each to find.

Term expansion for common geological concepts:

| Concept | Also search |
|---|---|
| Source rock | organic-rich shale / mudstone, TOC, Rock-Eval, hydrogen index, kerogen type, vitrinite reflectance, biomarkers, oil–source correlation |
| Reservoir quality | porosity, permeability, diagenesis, cementation, petrography, core analysis |
| Seal | caprock, top seal, seal capacity, mercury injection, evaporite, fault seal, shale gouge |
| Charge and timing | burial history, thermal history, basin modelling, fluid inclusions, apatite fission track, generation window |
| Trap | closure, anticline, inversion structure, pinch-out, stratigraphic trap, fault-dependent trap |
| Structural evolution | rifting, inversion, transpression, transtension, kinematics, restoration, fault reactivation |
| Stratigraphy | lithostratigraphy, biostratigraphy, sequence stratigraphy, chronostratigraphy, well correlation, type section |
| Hydrocarbon occurrence | discovery, oil show, gas show, seep, DST, well test |

Regional literature is often published in French, Portuguese, Spanish, Russian, Chinese, Arabic or another national language. Search translated terms for the key concepts and names.

## 2. Which tool for which job

| Need | Tool | Notes |
|---|---|---|
| Keyword or Boolean search with filters (year, type, open access, venue) | OpenAlex `search_works` | Use `preview: true` to size a query before pulling results. Record the canonical OQL query it returns. |
| Finding papers from a description of a concept or result | Exa (`category: "publication"`); OpenAlex `search_works` in semantic mode | Resolve every hit in OpenAlex before using it |
| Metadata for a known work; checking references | OpenAlex `get_work` (free); `resolve_references` (up to 25 per call) | See `lit-source-verification` |
| Citation chaining (references, citing works, related works) | OpenAlex `list_citations` | Section 4 |
| Authors, institutions, journals | OpenAlex `search_entities`, `get_entity` | Section 6 |
| Grey literature: geological surveys, government and operator reports, society pages, theses | Tavily, or Exa without a category | Restrict by domain where possible, e.g. the national survey's site |
| Legal full text | The open-access location in the OpenAlex record, then the publisher's open-access page or a repository copy | Then fetch or extract the page |

OpenAlex search calls cost credits against the daily budget; record lookups by DOI or ID are free. Use `preview` to size a query before pulling many pages.

## 3. Iterate: broad, then narrow

1. **Broad round:** establish the terminology, the major papers, the main authors and the competing interpretations.
2. **Narrow rounds:** use the formations, sub-basins, authors and disputes you have learned about, e.g. "Basin X Formation Y maturity" or "Basin X fault reactivation Cretaceous".
3. **After each round, note what changed:** new terms, authors or interpretations. Base the next queries on that. Do not re-run near-identical queries.

## 4. Citation chaining

For each key paper, meaning one that a conclusion depends on:
- **Backward:** scan its references for the primary data sources behind its interpretations.
- **Forward:** scan the works that cite it, especially later ones, which may refine or challenge it.
- **Stop** when chaining mostly returns works you already have.

## 5. Challenge searches

For each important interpretation, search specifically for disagreement:
- "Discussion" and "Reply" articles, comments and reinterpretations;
- contrary terms, such as "immature", "not a source", "alternative model", "reinterpretation";
- later papers on the same wells, outcrops or seismic lines.

Published discussions are among the most useful sources: they set out the disagreement and the evidence on each side.

## 6. Author requests

An author's publication list is judged by its completeness and its accuracy, and author profiles are often split or merged.

1. **Identify the author.** Use `search_entities` and examine each candidate's affiliations, topics, works and ORCID. If several people share the name, say how you told them apart: co-authors, affiliation, topic or ORCID.
2. **Look for split profiles.** Search works by name variants ("G. Archibong", with and without middle initials) without an author-ID filter, and across earlier affiliations such as previous universities. Add any works that belong to the same person but sit outside the main profile, and say so.
3. **Cross-check** the list against the author's ORCID record, where one exists and is readable.
4. **Present a table:** year, title, all authors (with this author's position), venue, type, DOI, status.
   - **Type:** journal article, conference paper or extended abstract, preprint, thesis, book chapter, or other.
   - **Status:** published, preprint, withdrawn, retracted or corrected (see `lit-source-verification`).
   - Merge a preprint into its published version. Flag records that look like duplicates, such as the same title under two DOIs, rather than listing them twice.
5. **State the coverage:** the total number of works, the sources checked, and whether the list is complete as far as those sources show. Never call a list "all papers" unless the checks above support it; say "works found in <sources>".

## 7. Keep a search log

Record, for every search:
- the tool;
- the query or OQL and the filters;
- the date;
- the number of hits, and the number of relevant hits kept.

Report the coverage in `limitations`, for example: "Searched OpenAlex (14 queries), Exa publication search (6), Tavily (3) on 2026-09-25. Not searched: GeoRef, AAPG Datapages, Lyell Collection, OnePetro (no access)."

Report only searches in the log.

## 8. When to stop

Stop when any of the following holds:
- **Saturation:** two consecutive rounds, including chaining, add no new relevant sources or claims for any research question.
- **Budget:** the budget is reached. The default is about 40 tool calls; about 10 for a narrow lookup; up to about 80 for `critical` priority.
- **Access:** the remaining gaps need sources you cannot access.

State which condition ended the search. A search that ended on budget with gaps remaining should report `partially_completed`, not `completed`.
