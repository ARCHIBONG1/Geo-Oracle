---
name: seismic-structural
description: Structural interpretation from seismic - faults, throw, folds, dip domains and structural closure - using section descriptors, discontinuity, dip and curvature attributes, with uncertainty handling. Lists planned structural tools. Load for questions about faults, traps, deformation or structure maps.
---

# Structural interpretation

## Available now

| Task | Tools |
|---|---|
| Locate candidate faults | `seismic_describe_section`: sharpest_lateral_breaks, low_continuity_tiles. Edge attributes: semblance, eig_complex, chaos. |
| Fault vs non-fault | Break repeated over several z ranges at the same position; coherence lineation on several sections. |
| Dip domains, folds | describe_section apparent_dip grid; gradient_dips and structure tensor; volume_curvature. |
| Locations in X/Y | `seismic_convert_coordinates` |
| Figures for audit | `seismic_render_section`, `seismic_render_slice` |

## Planned, not yet available

Do not call these unless they appear in your tool list: horizon tracking, fault likelihood and fault-surface extraction, fault throw measurement, time-structure and isochron maps, closure and spill-point analysis, and time-to-depth conversion. Without them:
- horizon geometry, closure area and spill point cannot be computed, so report them in `limitations`;
- throw can only be bracketed (see below).

## Fault identification procedure

1. Describe at least three sections crossing the suspected fault. Use perpendicular sections, and extend them beyond the fault zone.
2. A **fault candidate** has all three of the following:
   - a lateral break (low pair correlation) at the same or a smoothly shifting position, across two or more z ranges;
   - continuity recovering on both sides;
   - the position consistent between neighbouring sections.
3. Confirm with coherence on the same window. A fault is a narrow, laterally continuous low-coherence lineament; a chaotic body is a broad zone.
4. Alternatives to consider every time: a channel edge or incised margin; a chaotic facies boundary; a noise or acquisition footprint (regular spacing, aligned with inline or crossline); a migration artefact near steep dip; a velocity pull-up or push-down under an anomaly.
5. Classify:
   - the break and coherence values are **observations and measurements**;
   - "fault at XL 248–250, 300–700 ms" is an **interpretation**;
   - fault extent, and its linkage between sections, is an **inference**.

## Throw

- In time data, throw is in ms. Converting to metres needs a time-depth table or a velocity (`seismic_register_table`), and the source must be stated.
- Current tools give only a bracket. describe_section's lag search spans ±`max_lag` samples. If a break shows no consistent lag, the offset exceeds max_lag × dt, or the reflectors do not match across the fault. You can re-describe with a larger `max_lag`, but a lag larger than half a period is ambiguous (cycle skipping).
- A throw below the tuning thickness cannot be resolved. Report it as below resolution.

## Folds and closure

Dip reversals in the apparent_dip grid across sections suggest a fold crest or trough. Time structure can be a velocity artefact (pull-up under fast layers, sag under gas or salt edges). Any closure claim in time stays **partially_supported** at best until depth conversion or independent structural evidence exists.

## Uncertainty to report

- Fault position uncertainty is at least the lateral resolution and trace spacing.
- In time, dips and throws are apparent.
- Sections chosen: state which ones, and why they are representative.
- Structural claims usually need an independent line (well dips, regional structure from the structural_geology specialist) to be `supported`.
