---
name: structural-displacement
description: Throw profiles from horizon cutoffs, displacement-length against the global population (V2), growth and expansion indices (V6), relay candidates, and the displacement_analysis product. Load before sg_throw_profile, sg_dl_check or sg_growth_index.
---

# Displacement and growth

## Throw profiles: `sg_throw_profile`

- Needs a depth horizon and the fault from a `fault_set`.
- The tool samples the horizon at the offset and at twice the offset on each side of the trace, and extrapolates to the trace. This removes the surface's own dip.
- The maximum comes from a median-filtered profile, so one odd sample does not set Dmax.
- Read:
  - `max_throw_m` and where it is;
  - `median_throw_m`;
  - `throw_uncertainty_m` (root-2 times the pick uncertainty);
  - `downthrown_side_consistency`: a side that flips along the trace means a scissor fault, a relay, or picks that do not honour the fault;
  - `relay_candidates` (interior minima below 60 % of the neighbouring maxima).
- **Junctions.**
  - Another fault of the set can abut or cross this one from the side (`junctions` of kind `interior`). There, the horizon offset across this trace, along the abutting fault's hanging wall, is the sum of the two throws. So the profile steps there.
  - A junction of kind `tip` is where this fault itself ends against another. Its offset is not summed, the profile is not split, and its own maximum is the whole profile's.
  - `segments_between_junctions` gives the median and maximum per segment.
  - `max_throw_own_m` (the maximum of the segment away from the junctions) is this fault's own displacement. The D/L check uses it.
  - Quote `max_throw_m` as the composite offset at the junction, never as the fault's throw.
  - The tool does not sample a side whose sample line crosses another trace (`masked_by_other_faults`).
- Near a tightly curved surface (a crest within a few cells of the fault), the extrapolation over-corrects. Use a smaller `offset_m` there, and say so.
- A throw finer than the uncertainty band is not resolved: report it as "below resolution".

## Displacement-length: `sg_dl_check` (V2)

Dmax/L for the global population mostly lies between 10^-3 and 10^-1 (Kim & Sanderson 2005). Outside it:
- under-displaced faults are usually linked segments whose length is over-counted;
- over-displaced faults usually continue beyond the picks.

Report an outlier with its likely explanation, and do not hide it. It fails V2 only without an explanation.

## Growth: `sg_growth_index` (V6)

Expansion index = hanging-wall thickness / footwall thickness for an interval (Thorsen 1963; as defined by Eskens et al. 2025 and Lathrop et al. 2021). EI > 1 is across-fault thickening: deposition while the fault moved.
- Growth is resolvable only when the hanging wall is thicker by more than twice the pick uncertainty. Only then does V6 pass.
- EI below 1 beyond the uncertainty is unusual. It is typically a thickness misread on one side (Eskens et al. 2025). It is not growth, and V6 is not tested. Check the picks on both sides.
- For several intervals across one fault, give `intervals` (youngest first) instead of the two thicknesses. Each interval is read by the same rule. V6 passes when at least one interval shows resolvable growth, and the result names those intervals.
- With two or more intervals, the tool draws the expansion-index chart itself (`result.figures`): one bar per interval from EI = 1, over the band that the pick uncertainty cannot resolve.

Cite where the thicknesses come from (horizon pairs or wells).

## Reporting

- `[measurement, derived]`: "F1: maximum throw 32 m (+/-3.5) at 1080 m along the trace, median 30 m, downthrown to the NE consistently; D/L 0.021 within the population (V2 pass); no relay candidates", `prov:` id, product `displacement_analysis`, depends on the horizon and fault_set references.
- `sg_throw_profile` draws the throw profile and the displacement-length plot itself (`result.figures`).
- For several faults on one displacement-length plot, use `sg_render(kind=dl, products=[...])` with their displacement_analysis products.

<!-- end of skill: structural-displacement -->
