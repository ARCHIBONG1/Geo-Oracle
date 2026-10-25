---
name: seismic-stratigraphic
description: Seismic stratigraphy and seismic facies, from the termination and facies tools, section descriptors and attributes. It covers termination detection and key surfaces, facies classification, stratal slices, reflection configuration and character, reflection terminations and sequence boundaries. Load for questions about depositional setting, sequences, facies, channels or reservoir distribution.
---

# Seismic stratigraphy and facies

**Channels and other depositional bodies as volumes:** if a task asks for stratigraphic features, bodies or channels, answer it by running them, not by reading sections. Run `seismic_segment_volume` with the generic `channel` recipe (skill `seismic-attribute-interpretation`, section 1):
- compute envelope, semblance and chaos;
- hang the window on a tracked horizon at the channel level (`interval`; a flat `z_window` only up to 25 samples);
- give the extracted faults as `faults`.

It returns each body's axis, sinuosity, width and thickness, and a labelled volume. Terminations and reflection character describe the frame. They never substitute for the extraction. Say "no channel" only from its result.

## Reading reflection character from describe_section

Describe sections along and across the expected depositional dip. The tile grids translate into standard seismic-facies descriptors:

| Descriptor | From | Reading |
|---|---|---|
| Continuity | continuity grid | ≥0.8 high (continuous); 0.5–0.8 moderate; <0.5 low, discontinuous or chaotic |
| Amplitude | relative_rms grid | >1.5 high; 0.7–1.5 moderate; <0.7 low (relative to the section) |
| Configuration | apparent_dip grid across rows | Parallel: similar dip in all rows. Divergent: dip changes steadily with depth, meaning thickening. Oblique or sigmoid: dipping rows bounded by flatter rows. Chaotic: low continuity with scattered dip. |
| Frequency | instantaneous or dominant frequency attribute, per tile | Relative changes only; affected by tuning and absorption |

These thresholds describe. They do not classify rocks. A facies label ("continuous parallel high amplitude") is an **observation**. The depositional meaning ("shelf, alternating sand and shale") is an **interpretation**. It needs alternatives and, ideally, a well or an upstream stratigraphy/sedimentology finding.

## Terminations and key surfaces

Use `seismic_detect_terminations` on sections along and across depositional dip (at least two orientations).

- Each termination has a projected position (where reflector and surface would meet), the last picked point, its type, the dips and the surface it ends against. Converging reflectors interfere, and the tool stops picking them about half a wavelength early. The projected position corrects for that.
- Types are geometric:
  - **onlap**: reflector less inclined than the surface below;
  - **downlap**: more inclined than the surface below;
  - **truncation_or_toplap**: ends against a surface above.
  Geometry cannot separate toplap from erosional truncation. Name both, unless other evidence (incision, regional truncation) decides it.
- `surfaces` groups terminations by the surface they end against. A surface that collects three or more is a **candidate key surface**:
  - onlap above suggests a sequence boundary or transgressive surface;
  - downlap onto it suggests a downlap (maximum flooding) surface;
  - truncation below suggests an unconformity.
- A candidate sequence boundary needs the same relationship on several sections. One section gives a candidate, not a surface.
- The tool excludes aligned terminations at the same trace as fault-like. Check faults on the same sections before you call a surface.
- Terminations in time data show apparent dips. Later tilting changes onlap/downlap geometry.
- Alternatives: fault, velocity effect, multiple, and pinch-out below resolution.

## Stratal slices and facies maps

- Stratal slice: `seismic_horizon_map` operation `extract` with a horizon and offsets (for example rms from +10 to +30 ms). Proportional slice between two horizons: operation `proportional`. Horizon-parallel slices follow stratigraphy. Time slices do not.
- Facies map: `seismic_classify_facies` on an interval (between two horizons, or a window on one). Start with 3–5 classes.
  - The tool orders classes by RMS amplitude, and does not label them. "class 3 (high amplitude, continuous)" is an **observation**. "channel sands" is an **interpretation** that needs a well tie or other evidence.
  - The tool draws the class map's figure into `result.figures`.
- Keep the window inside one stratigraphic unit: a window that reaches into the next reflector classifies thickness, not facies. Check the class tile means against the isochron.

## Channels and bodies

- A laterally limited zone suggests a channel or body if it has sharp edges on coherence, a distinct amplitude or frequency, and often a concave-up base. Run the generic `channel` recipe over that level (above). Read its QC (`fault_overlap`, `extent`, `stability.merges`) before you decide what the bodies are.
  - The recipe's inputs mark the bodies' edges by design. So a body in the low-semblance tail is not thereby "discontinuity-dominated".
  - A body outside the fault zones is not fault disruption.
- Report its width in traces, converted to metres with the trace spacing from `seismic_convert_coordinates`. Report its thickness in ms, against the tuning thickness.
- Bodies thinner than tuning show amplitude that changes with thickness (tuning). Their amplitude is not a lithology indicator.

## Not available

No tool serves Wheeler (chronostratigraphic) transforms or automatic sequence-stratigraphic interpretation. Use terminations, key surfaces and stratal slices, and say that a Wheeler diagram could not be produced.

## Uncertainty to report

- The vertical resolution limits which sequences and beds can be separated.
- Time-domain thickness is not true thickness.
- Section selection bias.
- Stratigraphic interpretations are one seismic line of evidence. Well ties, biostratigraphy and regional frameworks from the stratigraphy and regional_geology specialists are the independent lines.

<!-- end of skill: seismic-stratigraphic -->
