---
name: lit-source-verification
description: Rules for verifying every source the literature specialist cites. Covers resolving works to DOIs or OpenAlex IDs, checking metadata and retraction status, labelling how much of each source was actually read (full text, abstract, metadata, secondary), and obtaining legal open-access full text. Use before citing any paper, report or thesis; whenever a search result, snippet or remembered reference is about to become a citation; when a paper is paywalled; and when reference details disagree between sources.
---

# Source verification and access

The most damaging failures of a literature agent are a citation that does not exist, a citation with the wrong details, and a paper reported on as if it had been read when only its abstract was. Verification prevents the first two; access labels prevent the third.

## 1. Resolve every source

For each work you intend to cite:

1. **Get a canonical record:** a DOI (preferred) and the OpenAlex ID. Use `get_work` for a single work (DOI lookups are free), or `resolve_references` for up to 25 references at once. `resolve_references` reports whether each reference exists and how confidently it matched.
2. **Compare the record** with what you believed: title, authors, year, venue. Where they differ, use the record, not your recollection.
3. **Works without a DOI**, such as older papers, survey reports, theses and maps: use the OpenAlex record if there is one. Otherwise use the stable URL of the issuing institution, such as the survey's publication page or a repository handle. Record "no DOI".
4. **Works that cannot be resolved:** do not cite them as verified. Leave them out, or list them as `UNVERIFIED`, with what you know about each and why it matters.
5. **Preprint and published versions:** cite the published version. If only a preprint exists, say so, and check its current status. On arXiv, the latest version can be marked **withdrawn**: a withdrawn preprint is not evidence, so list it as withdrawn.

## 2. Retractions and corrections

Check the retraction flag in the OpenAlex record. If you have Crossref access, also check the work's `update-to` field for retraction or correction notices; Crossref includes the Retraction Watch data there.

- **Retracted work, or withdrawn preprint:** do not use it as evidence. You may mention that it was retracted or withdrawn.
- **Corrected work:** use the corrected version, and note the correction if it matters.

## 3. Quality flags (inputs to appraisal, not reasons to exclude)

Flag each of the following, and weigh it in `lit-source-appraisal`:
- journals of doubtful editorial standards;
- conference abstracts without peer review;
- non-reviewed reports;
- promotional material;
- sources that cite nothing for their key claims.

## 4. Access levels

Label every source with one of these, and carry the label in each evidence item's `source_reference`:

| Level | Meaning | What you may report |
|---|---|---|
| `full-text` | You read the paper (or the relevant sections) | Anything in the text, figures and tables, with its location |
| `partial` | Preview pages or selected sections only | Only what you saw; say which parts |
| `abstract` | The abstract only | Only claims stated in the abstract, marked "per abstract" |
| `metadata` | The bibliographic record only | That the work exists and what it is about; no findings |
| `secondary` | Known only through another work's citation of it | "As cited in …"; treat it as that other work's claim |

Never report values, figures or detailed arguments from a source read at `abstract` or `metadata` level.

## 5. Getting legal full text

Try these in order:
1. the open-access location in the OpenAlex record, which is the same data Unpaywall serves;
2. the publisher's open-access page;
3. a repository copy: institutional repositories, CORE, EarthArXiv or ESSOAr preprints;
4. the author's institutional page.

OpenAlex can also supply cached open-access PDFs. That costs credits, so use it for key papers only.

**Never** use piracy sites, shared credentials or paywall workarounds. If a key paper is paywalled, work at abstract level and record the limitation, e.g. "<Author> et al. <year> (<venue>): paywalled; abstract only; its porosity data were not reviewed."

## 6. Source register

Keep a working register of sources and use it to build every `source_reference` string. For each source record:
- a reference number (R1, R2, …);
- the DOI and OpenAlex ID;
- a short citation;
- the source type and whether it is peer reviewed;
- the access level;
- the retraction check result;
- its geographic relevance.

Format (placeholders shown; always fill them from the resolved record):

```
R4 | doi:<DOI> | openalex:<W-id> | <First author> et al. <year>, <venue> |
     peer-reviewed article | access: full-text | retraction: none | relevance: basin-scale
```
