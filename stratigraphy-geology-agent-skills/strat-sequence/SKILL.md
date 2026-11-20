---
name: strat-sequence
description: Stacking patterns from GR, and candidate maximum flooding and maximum regressive surfaces. It covers how a candidate becomes a key surface on the panel, where a tied sequence boundary comes from, and what stays a hypothesis. Load before stg_stacking_patterns and before naming any sequence-stratigraphic surface.
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

## Where a tied sequence boundary comes from

No tool finds a sequence boundary. You propose it from evidence that other tools and other specialists give. It is tied when it is a pick on the panel at each well, and at least one of these holds:

- **Missing time (rank 1-2).** Biostratigraphic datums or dated horizons show a gap at the surface. Run `stg_age_model` with the surface's depth in `hiatuses_tvdss`, and run `stg_gap_vs_fault` first, with the missing thickness from `stg_isopach` or `stg_eroded_thickness`: a fault cut-out is not a sequence boundary (`strat-unconformities-thickness`).
- **Seismic terminations (rank 3).** The seismic result shows truncation or toplap below the surface and onlap above it, on more than one section, and the horizon is tied at the wells. If you do not have the terminations, name them in `missing_data` for `seismic_interpretation`.
- **Facies (rank 4, a marker once its identity is shown in each well).** The surface carries an abrupt basinward shift of facies (for example fluvial or estuarine valley-fill sandstone directly on marine shelf mudstone; Van Wagoner et al. 1990), or a palaeosol or root traces below it, in the facies product of `sedimentology`. The same relationship must be present in the other wells at a consistent position. One well gives a candidate only.

A boundary that rests on facies alone caps the panel as other rank-4 ties do. A sharp-based sand can also be a channel base inside one sequence, or the base of a shoreface that moved basinward during the fall (a surface of marine erosion, which is not the boundary): name these rivals in `alternatives`.

The boundary is not one kind of surface along its length (Catuneanu et al. 2009):

- **Updip, on the shelf**, it is the subaerial unconformity: erosion or exposure, with missing time. The three kinds of evidence above find this part.
- **Downdip, in the basin**, it is the correlative conformity: no erosion and no missing time that the data can resolve. Missing time will not find it. Carry the surface basinward from the last well or section that shows the unconformity, with a seismic horizon or a biostratigraphic time line. That time line (rank 1-3) is the tie of the downdip wells.

Where the correlative conformity lies depends on the model, so state the model with the pick:

- at the start of the relative sea-level fall (Posamentier and Allen 1999): the deposits of the fall (forced regression) lie above the boundary;
- at the end of the fall (Hunt and Tucker 1992): the deposits of the fall lie below the boundary.

The two models put the downdip boundary at different depths and give different systems tracts for the deposits of the fall. Use the model that the task or the supplied reports use. If nothing states a model, and the panel reaches the basin, pick with one model on the panel, and state the other placement in `assumptions`. Say that the choice is a convention and not a result of the data: no test separates the two, so the other placement is not an entry of `alternatives`. Never join a boundary picked with one model in one well to a boundary picked with the other model in another well.

## Pitfalls

- Blocky logs (sharp-based, uniform sands) give few trends: the tool returns aggradational packages. Say that the method cannot read them.
- A fining-upward package can be a channel fill, not a transgression. The panel and the facies from `sedimentology` decide.
- Lithostratigraphic units cross time lines. A flooding surface does not.

## Reporting

- Packages and candidates: `[observation, derived]`.
- A key surface on the panel: `[interpretation]`, with its tie rank.
- Systems tracts: `[interpretation]` only on an admitted panel, otherwise `[hypothesis]`.
- A sequence boundary: `[interpretation]` with its tie rank, the kind of evidence, the part (unconformity or correlative conformity) at each well, and the model. A boundary from one well, or from GR trends alone, is `[hypothesis]`.

<!-- end of skill: strat-sequence -->
