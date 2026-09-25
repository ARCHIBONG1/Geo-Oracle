---
name: lit-evidence-extraction
description: How the literature specialist extracts geological evidence from sources and maps it onto the Geo Oracle gateway's JSON output contract. Covers separating observations, measurements and author interpretations; recording quantitative values with units, method, sample and location; labelling geographic relevance and access level; and the source_reference format. Use whenever you turn what you read into findings, and always before writing the final JSON for a task that came with an OUTPUT CONTRACT.
---

# Evidence extraction and output mapping

The output is a geological evidence base organised by research question, not a set of paper summaries. Every finding must be traceable to where it came from and labelled with how local and how direct it is.

## 1. What to record for each finding

- **Statement:** one clear sentence.
- **Evidence class:** `observation`, `measurement`, `interpretation`, `inference` or `hypothesis`.
- **Geographic relevance:** `study-area`, `immediate-region`, `basin-scale`, `regional-analogue`, `external-analogue` or `unknown`.
- **Source:** its DOI or OpenAlex ID, a short citation, its access level, and the location in the source (page, figure or table).
- **Agreement:** whether other sources support or contradict it, with their ids.
- **Assumptions and limitations** that matter.

**Split observation from interpretation, even within one paper.** For example:
- "Well A-1 penetrated 25 m of gas-bearing sandstone" is an observation.
- "The sandstone is interpreted as a basin-floor fan" is an interpretation that depends on it.

## 2. Quantitative data

For every value, record:
- the value, its units and the method, e.g. Rock-Eval, core plug, log-derived or vitrinite reflectance;
- the sample type and location: well or outcrop, and depth or interval;
- the number of samples and the range, where given;
- where in the source the value appears, e.g. "Table 2, p. 14".

Rules for values:
- **No merging.** Do not average values across sources or methods; report each source's range separately.
- **No silent conversion.** If you convert units, give both the original and the converted value.
- **Figures.** Mark values read from a figure as "read from Fig. X; approximate".
- **Access.** Values need `full-text` or `partial` access to the part that contains them. Values quoted in an abstract are marked "per abstract".

## 3. Mapping to the gateway JSON

Use this mapping when the task includes an OUTPUT CONTRACT:

| What you found | JSON section | `classification` | `source_type` |
|---|---|---|---|
| Something a source documents from data | `observations` | `observation` | `literature` |
| A reported value | `measurements` | `measurement` | `literature` |
| The authors' interpretation | `interpretations` | `interpretation` | `literature` |
| Your own conclusion across sources | `inferences` | `inference` | `derived` (put the supporting E-ids in `supporting_evidence_ids`) |
| A hypothesis raised in the literature | `hypotheses` | `hypothesis` | `literature`, or `derived` if it is your own |
| A material conclusion for Geo Oracle | `claims` | Its nature | — (support and contradiction through evidence ids; `status` from the claim-status list) |
| A disagreement between sources | `contradictions` | — | Evidence ids, possible explanations, discriminating evidence |
| A question the literature cannot answer | `unknowns` | — | — |
| Data that would answer it | `missing_data` | — | — |
| Access and coverage limits, and the search coverage line | `limitations` | — | — |
| Implications the project data can test | `recommended_followups` | — | — |

**`source_reference` format** (pipe-separated, one source per item):

```
doi:<DOI> | openalex:<W-id> | <First author> et al. <year>, <venue> | relevance: <class> | access: <level> | loc: <page/figure/table>
```

An unresolved source starts with `UNVERIFIED |`. A source without a DOI uses `url:<stable institutional URL>` in its place.

**Statements.** When the evidence is not from the study area, begin the statement with its relevance, e.g. "Basin-scale: …" or "Regional analogue: …". The qualifier then survives even when the statement is quoted without its metadata.

**Ids.** Use the local ids E1…, C1… and X1…; the gateway namespaces them. Reference upstream findings by the full ids you were given.

## 4. Example

Placeholders are shown in angle brackets. Real items carry resolved identifiers.

```json
{
  "scientific_status": "partially_completed",
  "summary": "Basin-scale literature documents an organic-rich Lower Shale with oil-prone kerogen; no published maturity data exist within the study area.",
  "measurements": [
    {
      "evidence_id": "E1",
      "statement": "Basin-scale: Lower Shale TOC 1.8–6.2 wt% (n = 38 cuttings samples, wells B-2 and B-5).",
      "classification": "measurement",
      "source_type": "literature",
      "source_reference": "doi:<DOI> | openalex:<W-id> | <Author> et al. <year>, <venue> | relevance: basin-scale | access: full-text | loc: Table 2",
      "limitations": ["Wells B-2 and B-5 lie about 60 km from the study area"]
    }
  ],
  "interpretations": [
    {
      "evidence_id": "E2",
      "statement": "Basin-scale: the authors interpret the Lower Shale as a Type II marine source rock.",
      "classification": "interpretation",
      "source_type": "literature",
      "source_reference": "doi:<DOI> | openalex:<W-id> | <Author> et al. <year>, <venue> | relevance: basin-scale | access: full-text | loc: p. 9",
      "supporting_evidence_ids": ["E1"]
    }
  ],
  "claims": [
    {
      "claim_id": "C1",
      "statement": "An organic-rich Lower Shale is documented at basin scale; its presence and maturity in the study area are not established by the literature.",
      "classification": "inference",
      "supporting_evidence_ids": ["E1", "E2"],
      "status": "partially_supported"
    }
  ],
  "unknowns": ["Thermal maturity of the Lower Shale within the study area"],
  "missing_data": ["Rock-Eval and vitrinite reflectance data from study-area wells"],
  "limitations": ["Searched OpenAlex (12 queries), Exa publication search (5), Tavily (2) on <date>. Not searched: GeoRef, AAPG Datapages (no access)."],
  "recommended_followups": ["Test Lower Shale TOC and maturity in study-area wells (Rock-Eval, Ro) to check the basin-scale data"]
}
```

## 5. Size and priority

Return the most material findings: typically 15–30 evidence items and 5–10 claims, unless `required_outputs` asks for more. The gateway trims long results, and Geo Oracle can page through them, but a focused evidence base is worth more than an exhaustive one. Prioritise evidence that:
1. bears directly on the research questions;
2. is more local;
3. is data-backed;
4. is disputed.

## 6. Choosing `scientific_status`

| Status | Use when |
|---|---|
| `completed` | All the research questions were addressed to saturation |
| `partially_completed` | Some questions remain open, or the budget or access limits ended the search |
| `insufficient_data` | The literature is too thin to answer the main question |
| `cannot_determine` | The sources conflict and cannot be discriminated |
| `requires_clarification` | The request is ambiguous, e.g. two basins share the name, and you could not resolve it |
