---
name: seismic-qi
description: Quantitative interpretation and amplitude analysis - prerequisites for trusting amplitudes, DHI screening criteria, tuning, and the strict limits on fluid and lithology claims from post-stack data. Lists planned QI tools (well tie, wavelet, synthetic, AVO, inversion). Load for any question about amplitude anomalies, DHIs, fluids, lithology or reservoir quality.
---

# Amplitude and quantitative interpretation

## Prerequisites: check them before any amplitude statement

1. **Gain.** `seismic_qc` amplitude check: is POSSIBLE_GAIN_OR_AGC or CLIPPING flagged? Does the task or textual header mention AGC? If amplitudes may be gained, relative amplitude statements are unreliable. Say so, and cap every amplitude-based claim at `proposed`.
2. **Polarity.** Stated, or `unknown`? Without polarity, "high-impedance" or "low-impedance" cannot be read from the sign of the amplitude.
3. **Phase.** From `seismic_qc`, modulo 180°. Non-zero phase shifts the peak away from the interface, and mixes peaks and troughs.
4. **Tuning.** From `seismic_resolution_limits`. For beds near or below tuning thickness, amplitude depends on thickness, and may brighten with no change in rock or fluid.
5. **Noise.** The SNR at the target. Below about 2, isolated amplitude anomalies may be noise.

Record the status of all five in `limitations` whenever you discuss amplitude.

## DHI screening with current tools

The criteria below are screening **observations**. Measure each; don't assume it.

| Criterion | How to measure now |
|---|---|
| Anomalous amplitude | envelope or rms; describe_section anomalies relative to the background distribution |
| Conformance to structure | The anomaly's lateral limit follows a constant z or contour. Only approximate without horizon tools; state this. |
| Flat spot | A near-flat event (apparent dip ≈ 0) cutting dipping reflections inside the anomaly. Rule out multiples and processing artefacts. |
| Phase or polarity change at the anomaly edge | instantaneous_phase, or apparent_polarity, across the edge |
| Frequency shadow beneath | cwt_ricker at a low vs a high frequency below the anomaly; instantaneous_frequency drop |
| Velocity sag beneath | apparent_dip pattern of deeper reflectors under the anomaly |

Rules:
- One criterion alone never supports a fluid claim, and several criteria from the same volume are still one line of evidence.
- A statement like "gas-charged sand" is at best a **hypothesis**, `proposed`, until well data or independent QI (AVO, a well tie) supports it.
- Always give the alternatives: a lithology contrast (for example a tight carbonate, coal, volcanic or cemented layer), tuning, a processing or gain artefact, a multiple, or a residual-moveout effect.

## Planned, not yet available

Do not call these unless they appear in your tool list: wavelet extraction, synthetic seismogram and well tie, AVO intercept/gradient and AVO class analysis, fluid factor, coloured or model-based inversion, and 4D difference analysis.

Without them, the following go in `missing_data`:
- well logs: `wells_petrophysics: DT, RHOB and VSHALE logs with checkshot for W1 — LAS — needed for a synthetic tie that fixes polarity and phase`;
- partial stacks: `data_provider: near and far angle stacks over the prospect — SEG-Y — needed for AVO class`.

## Reporting

- Amplitude values are **measurements**, with provenance.
- A DHI criterion being met is an **observation**.
- "Consistent with hydrocarbons" is an **interpretation**, with alternatives.
- A fluid or lithology prediction is a **hypothesis**. Its `limitations` include the prerequisites above and the missing independent data.
