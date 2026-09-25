---
name: seismic-stratigraphic
description: Seismic stratigraphy and seismic facies - reflection configuration, continuity, amplitude and frequency character, reflection terminations and sequence boundaries - read from section descriptors and attributes. Lists planned stratigraphic tools. Load for questions about depositional setting, sequences, facies, channels or reservoir distribution.
---

# Seismic stratigraphy and facies

## Reading reflection character from describe_section

Describe sections along and across the expected depositional dip. The tile grids translate into standard seismic-facies descriptors:

| Descriptor | From | Reading |
|---|---|---|
| Continuity | continuity grid | ≥0.8 high (continuous); 0.5–0.8 moderate; <0.5 low, discontinuous or chaotic |
| Amplitude | relative_rms grid | >1.5 high; 0.7–1.5 moderate; <0.7 low (relative to the section) |
| Configuration | apparent_dip grid across rows | Parallel: similar dip in all rows. Divergent: dip changes steadily with depth, meaning thickening. Oblique or sigmoid: dipping rows bounded by flatter rows. Chaotic: low continuity with scattered dip. |
| Frequency | instantaneous or dominant frequency attribute, per tile | Relative changes only; affected by tuning and absorption |

These thresholds describe; they don't classify rocks. A facies label ("continuous parallel high amplitude") is an **observation**. The depositional meaning ("shelf, alternating sand and shale") is an **interpretation** that needs alternatives and, ideally, a well or an upstream stratigraphy/sedimentology finding.

## Terminations and boundaries

Termination types (onlap, downlap, toplap, truncation) show as a dip change between vertically adjacent tiles at the same lateral position. Reflections dipping in one row meet a flatter row above or below.

- Record the position and z of each change as an observation.
- A candidate **sequence boundary** or unconformity needs the same angular relationship on several sections, together with truncation below or onlap above.
- At tile resolution, terminations are suggested, not proven. Say so, and request dedicated tools or data in `recommended_followups`.
- Alternatives: a fault, a velocity effect or a multiple.

## Channels and bodies

- A laterally limited zone with sharp edges on coherence, a distinct amplitude or frequency, and often a concave-up base suggests a channel or body.
- Report its width in traces, converted to metres with the trace spacing from `seismic_convert_coordinates`, and its thickness in ms against the tuning thickness.
- Bodies thinner than tuning show amplitude that changes with thickness (tuning). Their amplitude is not a lithology indicator.

## Planned, not yet available

Do not call these unless they appear in your tool list: horizon-based stratal and proportional slicing, automatic termination detection, seed-based seismic facies classification, and Wheeler or chronostratigraphic transforms. Without them:
- say that facies maps and termination maps could not be produced;
- use sections and windowed attributes instead.

## Uncertainty to report

- The vertical resolution limits which sequences and beds can be separated.
- Time-domain thickness is not true thickness.
- Section selection bias.
- Stratigraphic interpretations are one seismic line of evidence. Well ties, biostratigraphy and regional frameworks from the stratigraphy and regional_geology specialists are the independent lines.
