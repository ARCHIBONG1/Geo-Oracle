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
| Cross-check any structure against discontinuity attributes | `seismic_structure_qc` | per body its share in the attribute's anomalous tail; the share of the tail the structure captures; tall anomalies no body explains |
| Hand bodies to other specialists | `seismic_publish_bodies` | a `body_set` product (footprints in X/Y, sizes, orientation, channel/karst/fault metrics, stability, resolution flags) for prospect_target, structural_geology and sedimentology |

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

1. `seismic_list_recipes` lists every recipe with its inputs (sense and weight), the exact attribute call behind each input (`compute_with`), its method and thresholds, whether horizons are required (`horizons`), its cleanup and shape filter. Read it rather than remembering a table. Choose by the question: `fault_attribute` when fault likelihood looks poor; `channel` for every channel, body or stratigraphic-feature question; `paleokarst`, `salt`, `mtd`, `gas_chimney` by the feature named. The typed channel recipes (`channel_meandering`, `channel_submarine`) need a `type_basis` (`user` or `well`) and an interval; they never run before the generic one otherwise.

   **The generic `channel` recipe is the channel answer.** It does not assume bright or dim fill, and the catalogue reads the type from the body: `channel.planform` (straight < 1.05, sinuous, meandering >= 1.5 by sinuosity) and `input_percentiles` (envelope near 1 = bright fill, near 0 = dim fill). A type the user names that has no recipe (tidal, braided, incised valley, distributary) is still the generic recipe with `type_basis: {source: 'user', detail: '<their words>'}`; report the measured planform and fill. A valley is a wide channel body: report its width and planform, never a type it was not measured to have. Run it with a `z_window` (or the interval between tracked horizons) and **`faults: {volume: <the seismic_interpret_faults volume>}` whenever faults have been extracted**: the recipe's inputs (low semblance, high chaos) are also a fault's, and without the faults the bodies grow into the fault zones and merge. "No channels" is said only from its result, never from sections or terminations.

   **Read the body QC before the bodies** (`result.qc`; when the reply says it was abridged, `seismic_get_record` with the provenance id returns it whole):
   - `selection`: how much of the window the bodies fill and how many rounds the cap took; report the final percentiles (`thresholds`), not the recipe's defaults. `final_exceeds_cap` means the bodies are still merged with their surroundings: give the faults, narrow the window or seed.
   - `stability.merges` / `merged_bodies`: bodies that become one a setting away are parts of one feature or a threshold effect; never report them as separate channels without saying so.
   - `extent`: a body flagged `window_share`, `level_share` or `branching` is a sheet or a merger, never one channel; its width, planform and sinuosity describe the merger. A channel does not have the survey's extents.
   - `fault_overlap`: a body mostly in the fault zones is fault-parallel; report it with the fault it follows, not as fill. Bodies outside the fault zones are not fault disruption.
   - `cross_check`: an attribute the recipe scored is marked `circular`; the bodies lie in its tail by construction, which says nothing about what they are. Only an attribute the recipe did not use, or the faults, checks that.
   - Stability `9/9` says the body persists; it says nothing about what the body is. A merger of channels, sheets and fault zones is perfectly stable.
   - A body that passes these checks is a channel interpretation at its claim status (`partially_supported` from seismic alone). The status is the answer; it is never a reason to withhold the volume or to call the bodies something else.

2. Compute every input with its `compute_with` call (attribute tools), and pass the output path in `inputs` by the recipe's input names (`'path.h5::component'` for a multi-output attribute).
3. **Constraints** do the geology the attribute cannot:
   - `interval`: top and base horizons (from `seismic_track_horizon`) bound the stratigraphic interval. The typed recipes (`channel_meandering`, `channel_submarine`, `mtd`) refuse to run without it. The generic `channel` recipe does not: with no horizons, give it a `z_window` around the level of interest (or a `polygon`), or run it on the whole volume.
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

**`result.qc` first, as for faults.** Four questions, each with a reading line:
- *measured*: `valid_fraction`, the share of the volume the constraints and finite inputs left.
- *signal*: `separability` per input (Otsu's eta; a constant input is flagged and carries nothing).
- *stability*: the extraction repeated at the nine neighbouring threshold settings; each body's `found_at`, `stable` (two thirds) and `voxels_p10/p50/p90`. A body found at under two thirds is setting-dependent: report it as such, never retune to keep or lose it. Skipped above 60 million voxels (crop first).
- *resolution*: `thickness_below_tuning`, `width_below_fresnel_diameter` per body, from the dominant frequency (measured on the source seismic for time volumes, or given in `resolution`) and the velocity (given, or a constant `velocity_model`). A flagged body is detected, not measured: never quote its thickness or width as resolved.
- *truncation*: the share of a body on a constraint or data boundary; over a quarter means its size is a minimum.
- *cross_check* (when `cross_check` names discontinuity volumes): each body's share in the attribute's anomalous tail and the tall anomalies no body explains. Same seismic, so consistency, not confirmation.


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
