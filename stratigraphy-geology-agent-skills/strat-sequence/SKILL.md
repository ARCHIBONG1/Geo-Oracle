---
name: strat-sequence
description: Stacking patterns from GR, and candidate flooding surfaces and sequence boundaries. It covers how a candidate becomes a key surface on the panel, and what stays a hypothesis. Load before stg_stacking_patterns and before naming any sequence-stratigraphic surface.
---

# Sequence stratigraphy

## `stg_stacking_patterns`

- Windowed slopes of a GR (or Vsh) curve classify packages, thicker than a minimum, as coarsening-upward (progradational), fining-upward (retrogradational or channel fill) or aggradational.
- Candidate surfaces come from the grading trends (Catuneanu et al. 2009):
  - maximum flooding surfaces at the top of fining-upward trends: the GR maximum where a fining-upward package below meets a coarsening-upward package above (Galloway 1989);
  - maximum regressive surfaces at the top of coarsening-upward trends: the GR minimum where a coarsening-upward package below meets a fining-upward package above.
- Do not infer a sequence boundary from GR trends alone.
- Surfaces from grain size alone are uncertain. Confirm them across the wells or with biostratigraphy.
- The window sets the scale: 5-10 m for parasequences, 30-50 m for parasequence sets. If the hierarchy matters, run both.

## From candidate to surface

A candidate from one well is a hypothesis. It becomes a key surface on the panel in these cases:
- the same signal appears in the other wells at a consistent position (a rank-4 tie);
- it coincides with a biostratigraphic datum or a dated horizon (rank 1-2);
- a seismic termination marks it (rank 3).

Assign systems tracts only on an admitted panel, with a flooding surface and a sequence boundary both tied. From one well, do not assign them at all.

## Pitfalls

- Blocky logs (sharp-based, uniform sands) give few trends: the tool returns aggradational packages. Say that the method cannot read them.
- A fining-upward package can be a channel fill, not a transgression. The panel and the facies from `sedimentology` decide.
- Lithostratigraphic units cross time lines. A flooding surface does not.

## Reporting

- Packages and candidates: `[observation, derived]`.
- A key surface on the panel: `[interpretation]`, with its tie rank.
- Systems tracts: `[interpretation]` only on an admitted panel, otherwise `[hypothesis]`.

<!-- end of skill: strat-sequence -->
