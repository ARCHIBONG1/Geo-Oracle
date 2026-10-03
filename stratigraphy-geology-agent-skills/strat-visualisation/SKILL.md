---
name: strat-visualisation
description: Correlation panels - the rendering standards the tool enforces, the two-datum check, and what a panel may and may not support. Load before stg_render.
---

# Stratigraphic visualisation

## `stg_render` (kind panel)

Inputs: the panel file, the wells' `correlation_logs` in panel order, the curve, the datum. The tool draws the unflattened panel (TVDSS) and the panel hung on the datum side by side.

## Standards (enforced)

- Wells spaced by their distance apart, not evenly; one curve track per well with identical scale.
- Depth axis in TVDSS; the hung panel's axis is depth relative to the datum.
- Reported tops dashed, interpreted picks solid, with a key; tie ranks printed beside the correlation lines.
- The caption names the panel, its status and its ledger.

## Protocol

1. **Describe, then interpret**: "the sand's top is 15 m below FS3 in W1 and 12 m below FS2 in W4" before any meaning.
2. **Check two datums.** View the key correlation hung on two different time lines; a conclusion that holds on only one is flagged.
3. **Panels give hypotheses, tools give correlations.** A match seen on a panel enters the findings only with a DTW result and a ledger behind it.
4. **Depths, thin beds and small offsets come from tools**, not from reading the panel.

## Reporting

`Figure: <caption> — <url>` in `conclusions`, with the datum and the panel's status named.
