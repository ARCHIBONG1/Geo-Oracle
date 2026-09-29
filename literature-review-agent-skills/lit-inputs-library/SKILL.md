---
name: lit-inputs-library
description: How the literature review specialist serves as librarian of the shared inputs folder - complete listings, confirming what files contain, reading and searching documents with page citations, previewing tables, importing chat uploads, and routing each file to the specialist that opens it. Load before any lit_* tool.
---

# The shared inputs folder

## Who can do what

The routing tables in this skill refer to specialists by their gateway tool names (seismic_interpretation, wells_petrophysics); Geo Oracle uses the same names.

| Agent | In `operations/inputs` it can |
|---|---|
| Geo Oracle | nothing: it asks you |
| seismic_interpretation | read SEG-Y, arrays and tables it is given by path; list SEG-Y and arrays |
| wells_petrophysics | read LAS and CSV well data it is given by path; list LAS/CSV |
| **you** | list **everything**; read documents; preview tables |

So your listing is the only complete view of the folder anyone has. Be exact: paths as listed, nothing inferred from names.

## Listing: `lit_list_inputs`

- **Scope**: the whole folder by default. Use `subfolder` (e.g. `wells`) or `types` (`document`, `table`, `seismic`, `well logs`, `array`, `image`, `spreadsheet`, `archive`, `other`) for narrow questions.
- **What each file gets**: `path`, `type`, `size_mb`, `modified` and `opened_by`.
- **Totals** per type and subfolder cover everything, even when `truncated` is true.
- **Hidden files are skipped.** Your own chat uploads appear under `chat_uploads`.

**Routing by `opened_by`:**

| Type | Opened by | Tell Geo Oracle |
|---|---|---|
| document (PDF, Word, text) | you | read on request; cite pages |
| table (CSV/TSV) | you (preview) and the specialist it concerns | e.g. tops, core, pressures → wells_petrophysics; horizon picks, time-depth → seismic_interpretation |
| seismic (SEG-Y), array / volume | seismic_interpretation | pass the path as given |
| well logs (LAS) | wells_petrophysics | pass the path as given |
| well logs (DLIS/LIS) | not supported | ask for LAS |
| spreadsheet, image, archive, legacy Office | nobody | say what would make it usable (export to CSV, save as PDF, unpack) |

## Reading: `lit_read_document`

- **Pages**: `first_page`/`last_page` select pages, and `max_chars` caps a call; continue from `next_page`.
- **Page numbers**: PDF pages are real pages. Word, text, Markdown and HTML have no pages, so the tool uses fixed-length pages: cite them as "page n of the tool's paging".
- **Metadata** (title, author) comes with the first read.
- **Empty text** means a scanned document without a text layer. Say so: OCR is not available.

## Searching: `lit_search_documents`

- **Modes**: `all_words` finds pages holding every word; `mode: "phrase"` finds an exact phrase.
- **Hits** give the document, the page and a short snippet. Read the page before relying on it: a snippet is context, not proof.
- **Scope**: search everything first, then read the hits.

## Tables: `lit_preview_table`

Columns, row count and first rows. It is enough to describe a table and route it, never to analyse it. The specialist the table concerns registers and validates it.

## Uploads: `lit_import_upload`

- **Your own chat**: `sandbox_path` only.
- **Geo Oracle's chat**: `source_agent: "geo-oracle"`, with the path from the task.
- **Afterwards**: read the returned `uploads/...` path.

## Reporting what you found

Use the citation forms of your system prompt (section 7):
- **Listing**: in `observations` (`classification` `data`, `source_type` `project_data`), with `source_reference` `project:operations/inputs | lit_list_inputs | <time of the call>`.
- **Document content**: one evidence item per point, with `source_reference` `project:<path> | p. <n> | prov:<id> | relevance: … | access: full-text`.
  - The user's own reports are `project_data`. A published paper found in the folder is `literature`, resolved to its DOI as well.
  - A report's conclusion stays that report's `interpretation`, however confident it sounds.
- **Conclusions**: for Geo Oracle, the paths to pass on, grouped by specialist, e.g. "wells_petrophysics: wells/W1.las, wells/W1_tops.csv".
- **Missing data**: what the question needed but the folder lacks, as `data_provider: <item> — <form> — <why>`.
