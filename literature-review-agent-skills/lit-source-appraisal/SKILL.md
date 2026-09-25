---
name: lit-source-appraisal
description: How the literature specialist judges the authority, evidence basis, geographic relevance and independence of geological sources. Covers peer-reviewed papers, reviews, geological-survey reports, regional syntheses, conference papers, theses and industry documents. Use when deciding which sources to rely on, when sources disagree, when classifying a finding as study-area, basin-scale or analogue evidence, and when checking whether supporting sources are really independent.
---

# Source appraisal

Search rank, citation count, recency and confident prose are not evidence quality. Appraise each important source for what it can actually support.

## 1. Default priority by source type

Use this order when sources conflict or time is short. A source's data basis (section 2) can move it up or down.

1. Peer-reviewed primary research with original data.
2. Peer-reviewed reviews and syntheses.
3. Geological survey publications and other authoritative institutional reports.
4. Recognised regional syntheses: society memoirs, special publications, basin atlases.
5. Conference papers and extended abstracts that contain original data.
6. Theses and dissertations with clearly documented original data.
7. Industry and technical reports, where the methods and data are documented.
8. Other credible technical sources.

Treat websites, blogs, commercial summaries, marketing material, search snippets and AI-generated pages as leads only, never as evidence.

## 2. Questions for every important source

- **Who produced it,** and could they have an interest in the outcome? Operator promotional material and licensing-round brochures are examples.
- **Type and peer review status.**
- **Data basis:** what data, how much, from where, from when? Is it original, or restated from elsewhere?
- **Coverage:** which area, interval and time period?
- **Methods:** were they appropriate, and are their limits stated? For example, maturity from Tmax alone, porosity from logs without core calibration, or ages from regional correlation rather than biostratigraphy.
- **Consistency:** does it agree with independent sources?
- **Limitations:** what do the authors themselves say limits the work?

## 3. Data-backed or conceptual?

Classify each important interpretation as one of:
- **Data-backed:** it cites specific observations or measurements.
- **Conceptual or model-based:** it rests on a regional model, analogy or reasoning.
- **Restated:** it repeats another source's interpretation.

Restated interpretations inherit the reliability of their origin. Trace them back (see citation chaining in `lit-search-protocol`).

## 4. Geographic relevance

| Class | Use when | Example |
|---|---|---|
| `study-area` | The evidence comes from inside the study area | A well or outcrop inside the licence |
| `immediate-region` | Adjacent areas sharing the same setting | A neighbouring block on the same structural trend |
| `basin-scale` | The same basin, outside the immediate region | A source-rock study from the basin's depocentre |
| `regional-analogue` | A different basin with a comparable setting | A conjugate-margin basin |
| `external-analogue` | A distant or generic analogue | A global type example |
| `unknown` | The location cannot be established | — |

Decide the class from coordinates, well names, maps and figure locations, not from the title. If the evidence is ambiguous, choose the less local class and say why.

## 5. Age, nomenclature and time scales

- **Superseded frameworks:** older work may use superseded formation names, ages or structural frameworks. Map old names to current usage and state the mapping.
- **Stage ages:** stage boundaries and absolute ages have been revised over time. When comparing ages across papers, check which time scale each paper used.
- **Newer is not truer:** a newer paper may reinterpret older data, while an older paper may hold the only direct observations. Judge the evidence, not the date.

## 6. Independence

Two sources count as one line of evidence when:
- the same authors or group analysed the same dataset;
- a review restates a primary paper;
- several reports rely on the same wells, samples or seismic lines;
- a model-based paper uses another paper's interpretation as its input.

Count independent lines, not papers, and state that count when you report support for a claim.

## 7. Reliability grade

For each important source, give a short grade for **this research question**: `high`, `moderate` or `low`, with a one-line reason. For example: "moderate: basin-scale Rock-Eval data (n = 42) but no samples within 50 km of the study area."

Do not give numeric scores. Keep the grades in your working notes. Where they matter, put the reasons into the `limitations` field of the evidence items, and into `limitations` and `claims`.
