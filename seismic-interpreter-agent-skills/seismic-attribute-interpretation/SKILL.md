---
name: seismic-attribute-interpretation
description: Attribute-driven interpretation - faults when fault likelihood looks poor, meandering and submarine channels, paleokarst and any other geobody - extracted from attribute volumes by recipes, with constraints, seeds from the strongest evidence, binary or labelled outputs and a catalogue of every body. Load for channels, karst or geobodies, and whenever the fault-likelihood QC looks poor.
---

# Attribute-driven interpretation

Fault likelihood (`seismic_interpret_faults`, or step by step `seismic_fault_likelihood` -> `seismic_extract_faults`) stays the primary fault method. The tools here are general: one engine, driven by recipes, for faults when likelihood is poor, channels, paleokarst and any body an attribute can outline.

## Tools

| Task | Tool | Key outputs |
|---|---|---|
| See the recipes and what each input needs | `seismic_list_recipes` | per recipe: purpose, the exact attribute call behind each input (`compute_with`), method, required constraints, cleanup, shape filter |
| Extract bodies | `seismic_segment_volume` (job) | labelled, binary or score volume; thresholds and their source; seeds used, rejected or not used; conflicts; every body with size, extents, orientation, volume, channel or karst metrics; `extraction_figure_markdown` |
| Describe or filter any body volume | `seismic_describe_bodies` | the same catalogue for any labelled or binary volume (e.g. from `seismic_binarize_volume`); optional shape filter writes a filtered volume |
| Second opinion | `seismic_compare_extractions` | share of each extraction within a voxel tolerance of the other, F-measure |
| Figure of any body volume | `seismic_render_extraction` | inline, crossline and z slices with each body coloured, map footprints |

## When fault likelihood looks poor (your judgement)

`seismic_interpret_faults` and `seismic_fault_likelihood` return `result.qc`. Read it before trusting the faults:

1. **Dip beyond the lag**: if `qc.suggested_max_dip_samples` is present, reflectors dip more than the likelihood's correlation follows (`reflector_dip.steepest_samples_per_trace_p90` > `max_dip_samples`). Rerun with that `max_dip_samples` first.
2. **Then judge** from the evidence, and say which evidence you used:
   - `measured_fraction` low: much of the interval is too quiet to measure, and likelihood is blind there.
   - `separability.otsu_eta` low (no clear fault tail; values near 1 separate well), or null.
   - the extraction finds no fault, or faults that make no geological sense, although `seismic_describe_section` shows sharp lateral breaks.
   These are judgements, not cut-offs. If likelihood looks poor, run the `fault_attribute` recipe, report both results, and give `seismic_compare_extractions` between the likelihood fault volume (`seismic_interpret_faults` with `output: "binary"`) and the recipe's volume.
3. **An explicit request wins.** If the user names an approach ("use fault likelihood only", "extract faults from coherence"), run exactly that. Report any QC concern, but do not switch methods. "Extract faults from coherence" means the `fault_attribute` recipe, which reads coherence by design; fault likelihood itself needs the seismic and refuses a coherence volume unless you give `override_reason` because the user asked for exactly that.

## Procedure

1. `seismic_list_recipes`; choose the recipe for the question:

   | Recipe | Inputs (sense) | Method | Constraint | Shape filter |
   |---|---|---|---|---|
   | `fault_attribute` | semblance (low) | hysteresis p98 / p93, thinned to sheets | z window suggested | tall (>= 20 % of the window), thin, dip >= 35 degrees |
   | `channel_meandering` | sweetness (high) | hysteresis p95 / p85 | **interval required** | map elongation >= 3 |
   | `channel_submarine` | chaos (high), envelope (high) | hysteresis p93 / p80 | **interval required** | map elongation >= 2.5 |
   | `paleokarst` | chaos (high), semblance (low) | hysteresis p97 / p90 | interval suggested | vertical extent >= 8 samples, round in map view |

2. Compute every input with its `compute_with` call (attribute tools), and pass the output path in `inputs` by the recipe's input names (`'path.h5::component'` for a multi-output attribute).
3. **Constraints** do the geology the attribute cannot:
   - `interval`: top and base horizons (from `seismic_track_horizon`) bound the stratigraphic interval. Channel recipes refuse to run without it.
   - `polygon` / `polygon_xy`: the area of interest.
   - `fault_side`: one fault block (`hanging_wall` or `footwall`, or a reference point for a vertical fault).
   - `z_window`; `body`: inside or outside an earlier extraction.
4. **Seeds from the strongest evidence available:**
   - the user's seeds, exactly as given (never replaced; if none is usable the tool says so);
   - else wells that penetrate the feature, tied to the seismic. Examples: a channel sand, a karst cavity or lost-circulation zone, a fault cut. Give `source: "well"`, the position, and `quality` = the well tie's correlation;
   - else interpreted horizons, for example an amplitude anomaly on a stratal slice (`source: "horizon"`);
   - else none: hysteresis needs no seed, and growing seeds itself at the strongest score peaks.
   The tool uses the strongest group present (`seed_policy: "all"` uses every valid seed).
5. **Time volumes:** give `velocity_model` (as for `seismic_time_to_depth`) so thickness, volume, dip and plunge are in metres. Without it, those are index units, and shape rules that need metres are skipped (listed under `shape_filter.not_applied`).
6. Run `seismic_segment_volume`. Change a threshold only through `overrides` (e.g. `{"method": {"strong": {"percentile": 99}}}`), and say why. The result records the effective recipe.
7. **Always publish `result.extraction_figure_markdown`** for every extraction you report, as for well ties: an extraction is a judgement, and a wrong one is easiest to see. You cannot see it; the user can.
8. For a 3D view, use `seismic_render_3d` with the output volume as a body overlay. To download, use `seismic_export_volume`.

## Reading the result

- **Thresholds** are percentiles of the score inside the constraints. They assume the feature is a minority of the constrained volume, so constrain tightly. Report `thresholds` and their source.
- **Seeds**: report each seed's status. Every entry in `conflicts` is evidence against the attribute's picture (for example a tied well that sees a channel where the score is low: a bed thinner than tuning, or an insensitive attribute). Put each conflict in `contradictions`; never drop one.
- **Bodies**: `shape_units` says whether shape is in metres or index units; never quote a dip or volume in index units. `fault.length_along_strike_m` / `height_along_dip_m`, `channel.sinuosity` (about +/- 1.5 %), `channel.mean_width` (about +/- one bin), `set.clark_evans_r` (karst spacing; no edge correction).
- "No body found with this recipe and these thresholds" is not "no body".
- **Resolution**: a body thinner than the tuning thickness (`seismic_resolution_limits`) has an unreliable thickness; one narrower than the Fresnel zone has an unreliable map width. Say so.
- **Curvature for karst**: most-negative curvature marks sags and collapse. Compute `gradient_dips`, then `volume_curvature` with `darray_il: '<dips>::il_dip'` and `darray_xl: '<dips>::xl_dip'`, and add it to the paleokarst recipe through `overrides.inputs` as `{name: 'curvature', sense: 'low', weight: ...}` with `inputs.curvature = '<curvature>::KMNeg'`. Since `inputs` is a list, an override replaces it whole, so repeat chaos and semblance there.

## Classification

- A body is defined by attribute values, not by rock or fluid. "Candidate channel body", "candidate collapse feature" and "fault from discontinuity" are **interpretations**. Do not call a body sand, gas, a cavity or a fault zone of a given rock type without well or QI evidence.
- Body sizes, orientations and metrics are **measurements** of the extraction, valid for the stated recipe, thresholds and constraints.

## Uncertainty to report

The recipe and version, every override, thresholds and their source, connectivity, constraints, seed evidence and conflicts, units (metres or index), and the threshold sensitivity when it matters. Rerun with stricter and looser thresholds and report how the body count and volume change.
