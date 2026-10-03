---
name: structural-displacement
description: Throw profiles from horizon cutoffs, displacement-length against the global population (V2), growth and expansion indices (V6), relay candidates, and the displacement_analysis product. Load before sg_throw_profile, sg_dl_check or sg_growth_index.
---

# Displacement and growth

## Throw profiles: `sg_throw_profile`

- Needs a depth horizon and the fault from a `fault_set`. The horizon is sampled at the offset and twice the offset on each side of the trace and extrapolated to the trace, which removes the surface's own dip; the maximum comes from a median-filtered profile so one odd sample does not set Dmax.
- Read: `max_throw_m` and where it is, `median_throw_m`, `throw_uncertainty_m` (root-2 times the pick uncertainty), `downthrown_side_consistency` (a side that flips along the trace means a scissor fault, a relay, or picks that do not honour the fault), `relay_candidates` (interior minima below 60 % of the neighbouring maxima).
- Near a tightly curved surface (a crest within a few cells of the fault) the extrapolation over-corrects; use a smaller `offset_m` there and say so.
- A throw finer than the uncertainty band is not resolved: report it as "below resolution".

## Displacement-length: `sg_dl_check` (V2)

Dmax/L for the global population mostly lies between 10^-3 and 10^-1 (Kim & Sanderson 2005). Outside it: under-displaced faults are usually linked segments whose length is over-counted; over-displaced ones usually continue beyond the picks. An outlier is reported with its likely explanation, not hidden; it fails V2 only without one.

## Growth: `sg_growth_index` (V6)

Expansion index = hanging-wall thickness / footwall thickness for an interval (Thorsen 1963). Growth is resolvable only when the thickness difference exceeds twice the pick uncertainty; otherwise V6 is not tested. Cite where the thicknesses come from (horizon pairs or wells).

## Reporting

- `[measurement, derived]`: "F1: maximum throw 32 m (+/-3.5) at 1080 m along the trace, median 30 m, downthrown to the NE consistently; D/L 0.021 within the population (V2 pass); no relay candidates", `prov:` id, product `displacement_analysis`, depends on the horizon and fault_set references.
- Throw profile with `sg_render(kind=throw)`.
