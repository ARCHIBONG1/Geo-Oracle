---
name: seismic-attributes
description: Choosing, parameterising, running and reading seismic attributes with the six attribute tools (complex trace, edge detection, dip/azimuth, frequency, noise reduction, signal processing). Covers the question-to-attribute decision matrix, parameter rules, chaining and pitfalls. Load before any seismic_*_attribute call.
---

# Seismic attributes

Six tools, one per attribute family. Each takes `attribute`, `input_paths` (usually `{"darray": <volume path>}`, an ingested `.h5` volume, always in IL,XL,T order; raw `.npy` or plain `.h5` files are refused, so load them with `seismic_ingest_array` first, stating their axis order if it is not IL,XL,T), `params`, and optionally `wait_seconds`. The method names below come from the d2geo-derived library. **Confirm exact names, parameters and defaults with `list_seismic_attributes(category=...)`** before your first call in a task, because the installed fork may differ.

Every result gives you:
- `provenance_id`, which you cite;
- `effective_params`: the values actually used, defaults included;
- `outputs`: statistics per output component;
- `geometry`: the output keeps the input's inline, crossline and z axes;
- `output_path`: a new volume you can describe, render, window or feed to another attribute.

## Workflow

1. Window first: use `seismic_extract_window` around the target, with at least 100 ms above and below and a few dozen traces beyond the area of interest, to limit edge effects.
2. If `seismic_qc` gave `LOW_SNR` or strong random noise, condition the data first (Subset E), then compute the attribute on the conditioned output.
3. To run several independent attributes, start each with `wait_seconds: 0`, then collect them with `get_attribute_job_result`.
4. Read the attribute volume with `seismic_describe_section` on the same sections you described in amplitude, and compare the two.
5. Report the attribute, its effective parameters and the provenance id in `source_reference`.
6. If the user wants the attribute volume itself, export it with `seismic_export_volume` and give the `download_url`.

## Decision matrix

| Question | First choice | Support | Subset |
|---|---|---|---|
| Where are faults or fractures? | semblance, eig_complex (coherence) | chaos; volume_curvature (Kmax/Kmin) for flexures | B |
| Fault vs chaotic facies vs noise? | coherence + chaos | describe_section breaks repeated over several z ranges | B |
| Structural dip, strike, azimuth? | gradient_structure_tensor, gradient_dips | describe_section apparent dip | C |
| Folds, flexures, subtle drape? | volume_curvature (needs inline and crossline dip) | gradient_dips first | B, C |
| Bright or dim amplitude, reflection strength? | envelope | rms, reflection_intensity, sweetness | A, F |
| Lateral lithology or thickness change, thin beds? | instantaneous_frequency, dominant_frequency | cwt_ricker spectral components | A, D |
| Channel edges, stratigraphic detail? | coherence on conditioned data | spectral components; instantaneous_phase for continuity | B, D, A |
| Frequency shadow below a bright spot? | cwt_ricker at low vs high frequency | instantaneous_frequency | D, A |
| Weak reflector continuity? | instantaneous_phase or cosine_instantaneous_phase | — | A |
| Phase or polarity display? | phase_rotation (for display only) | apparent_polarity | F, A |

## Subsets

**A. Complex trace** (`seismic_complex_trace_attribute`)
- Methods: envelope, instantaneous_phase, cosine_instantaneous_phase, instantaneous_frequency, instantaneous_bandwidth, dominant_frequency, frequency_change, sweetness, quality_factor, relative_amplitude_change, amplitude_acceleration, response_phase, response_frequency, response_amplitude, apparent_polarity.
- Envelope is reflection strength, independent of phase: the right amplitude measure for bright or dim events.
- Instantaneous phase shows continuity independent of amplitude, which is good for weak reflectors.
- Instantaneous frequency is noisy by nature. Read its tile statistics, not single samples. Anomalously low values under bright events may be absorption or tuning.
- Sweetness is envelope divided by the square root of instantaneous frequency. It highlights sand-prone bright, low-frequency packages in clastic settings, and is a screening tool only.

**B. Edge detection** (`seismic_edge_detection_attribute`)
- Methods: semblance, eig_complex, chaos, gradient_structure_tensor, volume_curvature.
- Default kernel (3,3,9) means inline, crossline and samples, matching the volume's IL,XL,T order whatever the order of the user's original array. The vertical size should span about half to one dominant period (period in samples = 1000 / (f_dom × dt_ms)). Increase the lateral size for noisy data.
- Low semblance means discontinuity: faults, channel edges, chaotic bodies or noise.
- `volume_curvature` needs two inputs: `{"darray_il": <inline dip volume>, "darray_xl": <crossline dip volume>}`. Compute the dips first (Subset C).
- It returns six components: H (mean), K (Gaussian), Kmax, Kmin, KMPos and KMNeg. Kmax and Kmin are the usual fault and flexure indicators.

**C. Dip and azimuth** (`seismic_dip_azimuth_attribute`)
- Methods: gradient_dips (dip_factor, kernel), gradient_structure_tensor (kernel required), plus 2D/3D dip and azimuth methods if the fork has them.
- The structure tensor returns six components (gi2, gj2, gk2, gigj, gigk, gjgk).
- In time data, dips are apparent, in ms per trace. Convert to degrees only with a velocity and the trace spacing, and state both.

**D. Frequency** (`seismic_frequency_attribute`)
- Methods: lowpass_filter, highpass_filter, bandpass_filter (freq_lp, freq_hp), cwt_ricker (freq), cwt_ormsby (freqs).
- `sample_rate` is filled from the volume automatically (see the tool warning). Never pass a value that differs from the volume's sample interval.
- Choose frequencies inside `band_minus20db_hz` from `seismic_qc`. Components outside the usable band are noise.
- Spectral decomposition for thickness uses components at about 0.5×, 1× and 1.5× the dominant frequency.

**E. Noise reduction** (`seismic_noise_reduction_attribute`)
- Methods: gaussian (sigmas), median (kernel), convolution (kernel).
- Median (3,3,3) removes spikes and preserves edges better than gaussian, so it is better before coherence.
- Heavy smoothing hides small faults. Report the conditioning used whenever you cite a coherence result.

**F. Signal processing** (`seismic_signal_processing_attribute`)
- Methods: rms (kernel), trace_agc, time_gain, reflection_intensity, first_derivative, second_derivative, gradient_magnitude, histogram_equalization, rescale_amplitude_range, phase_rotation (rotation).
- `trace_agc` and `histogram_equalization` destroy relative amplitude. Use their outputs for structural or stratigraphic viewing only, never for amplitude claims.
- `rms` with a vertical window about one period long is a stable amplitude-strength map attribute.

## Reading attribute output

Read the statistics in `outputs` (min, max, mean, std) for scale: an anomaly means something only relative to that distribution. Then run `seismic_describe_section` on the attribute volume. For unsigned attributes such as envelope, coherence and frequency, `relative_rms` per tile reads as "attribute level relative to the section".

An attribute anomaly is an **observation** or **measurement**. The geological meaning (a fault, a channel, gas) is an **interpretation**, and it needs the alternatives from the matrix above.

## Pitfalls

- Attributes from one volume are one line of evidence. See the independence rule in `seismic-core`.
- Edge artefacts sit within half a kernel of window boundaries, so ignore those tiles.
- Unknown params are ignored with a warning. Check `effective_params` to confirm what was used.
- Time slices through dipping strata mix stratigraphic levels. Prefer sections, or a window around the target interval.
