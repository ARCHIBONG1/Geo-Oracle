---
name: seismic-stratigraphic
description: Seismic stratigraphy and seismic facies - termination detection and key surfaces, facies classification, stratal slices, reflection configuration and character, reflection terminations and sequence boundaries, from the termination and facies tools, section descriptors and attributes. Load for questions about depositional setting, sequences, facies, channels or reservoir distribution.
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

## Terminations and key surfaces

Use `seismic_detect_terminations` on sections along and across depositional dip (at least two orientations).

- Each termination has a projected position (where reflector and surface would meet), the last picked point, its type, the dips and the surface it ends against. Converging reflectors interfere and stop being picked about half a wavelength early; the projected position corrects for that.
- Types are geometric: **onlap** (reflector less inclined than the surface below), **downlap** (more inclined than the surface below), **truncation_or_toplap** (ends against a surface above). Geometry cannot separate toplap from erosional truncation: name both unless other evidence (incision, regional truncation) decides it.
- `surfaces` groups terminations by the surface they end against. A surface collecting three or more is a **candidate key surface**: onlap above suggests a sequence boundary or transgressive surface; downlap onto it suggests a downlap (maximum flooding) surface; truncation below suggests an unconformity.
- A candidate sequence boundary needs the same relationship on several sections. One section gives a candidate, not a surface.
- Aligned terminations at the same trace are excluded as fault-like; check faults on the same sections before calling a surface.
- Terminations in time data show apparent dips; later tilting changes onlap/downlap geometry.
- Alternatives: fault, velocity effect, multiple, and pinch-out below resolution.

## Stratal slices and facies maps

- Stratal slice: `seismic_horizon_map` operation `extract` with a horizon and offsets (for example rms from +10 to +30 ms). Proportional slice between two horizons: operation `proportional`. Horizon-parallel slices follow stratigraphy; time slices do not.
- Facies map: `seismic_classify_facies` on an interval (between two horizons, or a window on one). Start with 3–5 classes. Classes are ordered by RMS amplitude and unlabelled: "class 3 (high amplitude, continuous)" is an **observation**; "channel sands" is an **interpretation** needing a well tie or other evidence.
- Keep the window inside one stratigraphic unit: a window reaching into the next reflector classifies thickness, not facies. Check the class tile means against the isochron.

## Channels and bodies

- A laterally limited zone with sharp edges on coherence, a distinct amplitude or frequency, and often a concave-up base suggests a channel or body.
- Report its width in traces, converted to metres with the trace spacing from `seismic_convert_coordinates`, and its thickness in ms against the tuning thickness.
- Bodies thinner than tuning show amplitude that changes with thickness (tuning). Their amplitude is not a lithology indicator.

## Not available

Wheeler (chronostratigraphic) transforms and automatic sequence-stratigraphic interpretation have no tool. Use terminations, key surfaces and stratal slices, and say that a Wheeler diagram could not be produced.

## Uncertainty to report

- The vertical resolution limits which sequences and beds can be separated.
- Time-domain thickness is not true thickness.
- Section selection bias.
- Stratigraphic interpretations are one seismic line of evidence. Well ties, biostratigraphy and regional frameworks from the stratigraphy and regional_geology specialists are the independent lines.
