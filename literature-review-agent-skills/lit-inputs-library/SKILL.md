---
name: lit-inputs-library
description: How the literature review specialist serves as librarian of the shared inputs folder. It covers complete listings and how to confirm what files contain. It also covers how to read and search documents with page citations, and how to preview tables. Then it covers how to import chat uploads, and how to route each file to the specialist that opens it. Load before any lit_* tool.
---

# The shared inputs folder

## Who can do what

The routing tables in this skill refer to specialists by their gateway tool names (seismic_interpretation, wells_petrophysics). Geo Oracle uses the same names.

| Agent | In `operations/inputs` it can |
|---|---|
| Geo Oracle | list file names, types and sizes (`list_shared_inputs`); it opens nothing |
| seismic_interpretation | read SEG-Y, arrays and tables it is given by path; list SEG-Y and arrays |
| wells_petrophysics | read LAS and CSV well data it is given by path; list LAS/CSV |
| **you** | list everything, with your chat uploads; read documents; preview tables |

Only you can confirm what a file contains. Be exact: give paths as listed, and infer nothing from names.

## Listing: `lit_list_inputs`

- **Scope**: the whole folder by default. For narrow questions, use `subfolder` (e.g. `wells`) or `types` (`document`, `table`, `seismic`, `well logs`, `array`, `image`, `spreadsheet`, `archive`, `other`).
- **What each file gets**: `path`, `type`, `size_mb`, `modified` and `opened_by`.
- **Totals** per type and subfolder cover everything, even when `truncated` is true. When the listing gives `next_offset`, call it again with `offset` set to that value to see the rest.
- **The listing skips hidden files.** Your own chat uploads appear under `chat_uploads`.

**Routing by `opened_by`:**

| Type | Opened by | Tell Geo Oracle |
|---|---|---|
| document (PDF, Word, text) | you | read on request; cite pages |
| table (CSV/TSV) | you (preview) and the specialist it concerns | e.g. tops, core, pressures → wells_petrophysics; horizon picks, time-depth → seismic_interpretation; point layers with coordinates, temperatures, backstrip columns → regional_geology |
| seismic (SEG-Y), array / volume | seismic_interpretation | pass the path as given |
| well logs (LAS) | wells_petrophysics | pass the path as given |
| well logs (DLIS/LIS) | not supported | ask for LAS |
| GIS vector (GeoJSON, shapefile, KML/GML), GIS layers (GeoPackage), grid (netCDF, GRD, GeoTIFF) | regional_geology | pass the path as given; a shapefile needs its .shx, .dbf and .prj beside it (listed as "shapefile part"); a KMZ needs its .kml extracted |
| JSON | regional_geology when it is GeoJSON; otherwise the specialist whose product it is | say which, from a preview if needed |
| text (unrecognised extension) | you, as text | read it to identify the format; route its data to the specialist once converted to CSV or LAS |
| spreadsheet, image, archive, legacy Office | nobody | say what would make it usable (export to CSV, save as PDF, unpack); a georeferenced TIFF is listed as "grid (GeoTIFF)", not as an image |

## Reading: `lit_read_document`

- **Pages**: `first_page`/`last_page` select pages, and `max_chars` caps a call; continue from `next_page`.
- **Page numbers**: PDF pages are real pages. Word, text, Markdown and HTML have no pages. For them, the tool uses fixed-length pages. Cite them as "page n of the tool's paging".
- **Metadata** (title, author) comes with the first read.
- **Empty text** means a scanned document without a text layer. Say so: OCR is not available.

## Searching: `lit_search_documents`

- **Modes**: `all_words` finds pages that hold every word. `mode: "phrase"` finds an exact phrase. It ignores line breaks and finds a phrase that runs onto the next page.
- **Hits** give the document, the page and a short snippet. Read the page before you rely on it: a snippet is context, not proof.
- **Scope**: search everything first, then read the hits. With no `paths`, the search covers the inputs folder, your chat uploads and the documents retrieved for this investigation (give `investigation_id`). If no readable document is in scope, the search returns no hits and says so. That is a result, not an error: report it as such.

## Tables: `lit_preview_table`

The tool shows the columns, the row count and the first rows. That is enough to describe a table and route it, but never to analyse it. The specialist that the table concerns registers and validates it.

## Uploads: `lit_import_upload`

- **Your own chat**: `sandbox_path` only.
- **Geo Oracle's chat**: `source_agent: "geo-oracle"`, with the path from the task.
- **Afterwards**: read the returned `uploads/...` path.

## Retrieved documents: `lit_fetch_document`

- **This investigation only.** Give `investigation_id` (the Investigation ID line of your task) to `lit_fetch_document` and `lit_list_inputs`. The file then goes into `retrieved/<investigation>/`, and the listing shows only the documents retrieved for this investigation. A document retrieved for another investigation is not evidence for this one. Fetch it again for this investigation if you need it.
- Give the open-access URL and a label (author, year, venue). The URL comes from OpenAlex's `best_oa_location`, Unpaywall, a repository, or a survey's own site. The file lands in `retrieved/` with a sidecar: URL, final URL, date, content type, checksum.
- **A landing page is not the paper.** If the URL claims a PDF and returns something else, the tool refuses the URL. An HTML page comes back with `page_kind`: `document`, `landing_page` (a repository record or an abstract page with a PDF link) or `stub` (a redirect or a script page with no article text). For a landing page, fetch its `pdf_url`. If `flags` holds `redirected_to_other_site`, the page comes from another site: check it before you use it. A source read from a landing page is at most `abstract`, and from a stub at most `metadata`. `lit_read_document` returns this as `access_ceiling`. If you need an HTML page (a survey web page, a report in HTML), the tool fetches it as text.
- Then use `lit_read_document(path='retrieved/…')` and `lit_search_documents(paths=[…])` as for any document. `lit_list_inputs` lists what you have retrieved.
- Never fetch or bypass paywalled content. Report it as a limitation with the citation.

## The basin synthesis: `lit_publish_synthesis`

For a basin or region's framework, give one row per element:

| Column | Content |
|---|---|
| element | `tectonic_phase`, `unit`, `unconformity`, `event` or `setting` |
| name | e.g. "Late Jurassic rifting", "Agbada Formation" |
| age_start_ma, age_end_ma | older first; null when the source gives no age |
| time_scale | the time scale the ages are on (the source's, converted and noted if you changed it) |
| description | one sentence |
| geographic_relevance | study-area, immediate-region, basin-scale, regional-analogue, external-analogue, unknown |
| source | `doi:…`, or `project:<path>` and `p. <n>` separated by a pipe, as in `source_reference` |
| access | full-text, abstract, metadata, secondary |

Rules:
- No row without a source: a row from memory is recall.
- A dated row names its time scale.
- Where sources disagree, give each source its own row, and say so in `notes`.

The product's reference goes in `conclusions`.

## Evidence from the person

Geo Oracle passes what the person said as items of `available_evidence`. An item of kind `expert_input` is a fact with its source. Use it as far as that source allows, and cite it. An item of kind `expert_judgement` is the person's opinion. At most, it can support an interpretation. On its own, it never supports a measurement, a status, a class or a quantity.

## Methodological provenance

Every tool result carries these fields:
- `methods`: the ids of the published methods that the computation implements, such as `m:lit/read-document`;
- `citation_version`: the state of the bibliography that the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:lit/read-document`). Name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user. The method's name in words lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:lit/read-document` says which
method. It never says who published it or when. An author surname may be part of a
method's name, but a year never is. The reference lives in the registry, and
only `lit_method_provenance` returns it.

If you are asked one of these questions, call `lit_method_provenance` and report what
it returns:
- which equation was used;
- what publication supports a method;
- what the method assumes;
- how far it has been verified.

Call it with the `method_id` from the result, or with a tool name for every method that
tool implements.

Do not answer such a question from your own knowledge. An unverified reference presented
as this system's provenance is worse than an admitted gap. It is the one failure that this
mechanism exists to prevent. If the tool reports that no source has been recorded, say
exactly that.

A reference states where a method was published. Status states how far the system's
current tool implementation of this method has been verified against that reference.
Report the two separately. Never let a citation stand in for a demonstration that the
implementation is correct.

## Reporting what you found

Use the citation forms of your system prompt (section 7):
- **Listing**: in `observations` (`classification` `data`, `source_type` `project_data`), with `source_reference` `project:operations/inputs | lit_list_inputs | prov:<id> | m:lit/list-inputs`.
- **Document content**: one evidence item per point, with `source_reference` `project:<path> | <Author or organisation> <year> | title: <document title> | p. <n> | prov:<id> | m:lit/read-document | relevance: … | access: full-text` (give the author, year and title as the document states them, and leave out what it does not state).
  - The readers have two limits. State them when they matter:
    - The reader does not extract a Word document's tables nested inside table cells, or any content inside tracked changes (python-docx documents both exclusions).
    - You may read a `.txt` file as a document with `lit_read_document`. If it is a delimited table, you may instead preview it with `lit_preview_table`. Say which you did.
  - The user's own reports are `project_data`. A published paper found in the folder is `literature`. Resolve it to its DOI as well.
  - A report's conclusion stays that report's `interpretation`, however confident it sounds.
- **Conclusions**: for Geo Oracle, the paths to pass on, grouped by specialist, e.g. "wells_petrophysics: wells/W1.las, wells/W1_tops.csv".
- **Missing data**: what the question needed but the folder lacks, as `data_provider: <item> — <form> — <why>`.

<!-- end of skill: lit-inputs-library -->
