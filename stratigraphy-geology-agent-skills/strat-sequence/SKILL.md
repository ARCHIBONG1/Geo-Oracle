---
name: strat-sequence
description: Stacking patterns from GR, candidate flooding surfaces and sequence boundaries, how a candidate becomes a key surface on the panel, and what stays a hypothesis. Load before stg_stacking_patterns and before naming any sequence-stratigraphic surface.
---

# Sequence stratigraphy

## `stg_stacking_patterns`

- Windowed slopes of a GR (or Vsh) curve classify packages as coarsening-upward (progradational), fining-upward (retrogradational or channel fill) or aggradational, thicker than a minimum.
- Candidate maximum flooding surfaces: the GR maximum where a fining-upward package below meets a coarsening-upward package above. Candidate sequence boundaries: sharp-based coarsening-upward packages.
- The window sets the scale: 5-10 m for parasequences, 30-50 m for parasequence sets. Run both when the hierarchy matters.

## From candidate to surface

A candidate from one well is a hypothesis. It becomes a key surface on the panel when the same signal appears in the other wells at a consistent position (a rank-4 tie), when it coincides with a biostratigraphic datum or a dated horizon (rank 1-2), or when a seismic termination marks it (rank 3). Systems tracts are assigned only on an admitted panel with a flooding surface and a sequence boundary both tied; from one well they are not assigned at all.

## Pitfalls

- Blocky logs (sharp-based, uniform sands) give few trends: the tool returns aggradational packages; say that the method cannot read them.
- A fining-upward package can be a channel fill, not a transgression: the panel and the facies from `sedimentology` decide.
- Lithostratigraphic units cross time lines; a flooding surface does not.

## Reporting

Packages and candidates as `[observation, derived]`; a key surface on the panel as `[interpretation]` with its tie rank; systems tracts as `[interpretation]` only on an admitted panel, otherwise `[hypothesis]`.
