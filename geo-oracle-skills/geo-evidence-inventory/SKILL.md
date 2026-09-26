---
name: geo-evidence-inventory
description: How Geo Oracle characterises the evidence available to a geological investigation and packages it for specialist agents. Use at the start of every investigation; whenever the user supplies or mentions data (files, well logs, LAS, seismic, core, tops, maps, reports, tables); before briefing any specialist with project evidence; and whenever you are unsure whether a dataset or measurement actually exists.
---

# Evidence inventory and packaging

Specialists analyse only what you hand them, and a conclusion is never better than the evidence under it. Before anything is interpreted, establish what actually exists, what is merely named, and what is missing.

## 1. Build the inventory

For each item the user supplies or mentions, record the following:

- **ID**: short and stable, e.g. `W1-logs`, `3D-North`, `Rpt-2019-regional`. Reuse it in briefs and citations.
- **Type, source, format.**
- **Coverage**: area, wells, depth or time interval, acquisition date.
- **Content actually present**: e.g. "curves GR, RHOB, NPHI, DT; no resistivity". Never write just "logs".
- **Reference frames and units**:
  - depth reference (MD, TVD or TVDSS, and the datum);
  - time or depth domain;
  - coordinate reference system;
  - units.
- **Processing, calibration and QC status**, as documented.
- **Known limitations.**
- **Availability class**, from the table below.

| Class | Meaning | Can support observations? |
|---|---|---|
| verified | You opened it and confirmed its content | Yes |
| partial | Available, but key characteristics are undocumented, e.g. seismic with no processing report | Yes, with those gaps stated |
| named | Mentioned but not supplied | No |
| unavailable | Known not to exist or not accessible | No |
| unknown | Cannot tell | No; treat as unknown |

## 2. A label is not a measurement

- "Well logs" does not establish that gamma ray, resistivity, density, neutron, sonic or image logs exist.
- "3D seismic" does not establish bandwidth, signal-to-noise ratio, phase, amplitude preservation, velocity control, depth conversion or adequate resolution.
- "Core" does not establish core descriptions, plug measurements or depth-shifted core.
- "Geological report" does not establish that its interpretations are current, local or reliable. A report's interpretation is a published interpretation, not project data.

## 3. What you may do with files yourself

In your sandbox you may:
- open files;
- read headers and metadata;
- list curves, columns and extents;
- extract the rows or excerpts a question needs;
- convert formats so a specialist can use them.

You may not interpret. Computing properties, picking horizons, correlating wells and interpreting facies all belong to the specialists.

## 4. Ask for what matters, once

If a planned question depends on evidence that is named, unknown or missing, ask the user in one grouped message. Say which question each item would unblock, for example "The density log for W2 would let the petrophysics specialist estimate porosity in the target interval."

If the user cannot provide the item, proceed with what exists. Record the gap in the ledger and pass it to specialists in `known_unknowns`.

## 5. Package evidence for a specialist

- **Send what the question needs, not everything.** Irrelevant evidence dilutes attention and costs tokens.
- **Small evidence goes verbatim in `available_evidence`:**
  - `id`: the inventory ID.
  - `kind`: e.g. `well_tops`, `core_description`, `seismic_section`, `report_excerpt`.
  - `description`: the source, reference frames, units, and QC caveats.
  - `excerpt`: the content itself.
- **Tables** keep their header row with units. If you send a subset, say so, e.g. "rows for W1–W3 only; the full table has 40 wells".
- **Bulk data** (full-resolution curves, seismic volumes, grids) goes by the identifier the specialists' own data tools can resolve, such as a project data-store URI or dataset ID, plus a short summary. Never retype large numeric arrays: a single transcription error silently corrupts an analysis.
- **Seismic data** goes by its path relative to the shared seismic data folder, e.g. `{"id": "S1", "kind": "seismic_volume", "description": "survey_a/pstm_full.sgy", "excerpt": "3D PSTM, time, 2 ms; polarity and CRS not stated"}`. Only `seismic_interpretation` can open that folder. A seismic file the user uploaded to this chat is in your sandbox, not in the folder: pass it as `{"id": "S1", "kind": "seismic_volume", "description": "uploaded to Geo Oracle chat: /opt/tf/uploads/seismic.npy", "excerpt": "3D volume, axes T,XL,IL, 4 ms"}` using its real sandbox path, and the seismic specialist imports it. Include an array's axis order when the user states one other than the default IL,XL,T (inline, crossline, time). If no file name is known, ask `seismic_interpretation` to list the available seismic data.
- **Never pass a sandbox path.** Specialists run in their own environments and cannot read your sandbox.
- **State what is uncertain about each item** in its `description`, and list missing items in `known_unknowns`.

Example entries:

```json
{
  "available_evidence": [
    {
      "id": "W1-tops",
      "kind": "well_tops",
      "description": "Operator tops, W1. Depths m MD below RT (RT = 25 m above MSL). No deviation survey supplied, so TVDSS is unknown.",
      "excerpt": "Formation | Top (m MD)\nUpper Shale | 1840\nMain Sand | 2112\nLower Shale | 2178"
    },
    {
      "id": "W1-logs",
      "kind": "well_log",
      "description": "LAS 2.0, 1800-2300 m MD, 0.1524 m step. Curves: GR (API), RHOB (g/cc), NPHI (v/v, limestone matrix). No resistivity. Readable by the petrophysics data tool as dataset W1-logs-v1.",
      "excerpt": "Dataset reference: W1-logs-v1. Summary: GR 20-140 API; RHOB 2.18-2.62 g/cc in 2112-2178 m."
    }
  ],
  "known_unknowns": [
    "Deviation survey for W1 (TVDSS conversion)",
    "Resistivity logs for W1"
  ]
}
```

## 6. Keep the inventory current

Write the inventory into the ledger's *Evidence inventory* section. When new data arrive:
1. Update the inventory.
2. Re-check tasks that were blocked by missing data.
3. Flag conclusions that the new data could change.
