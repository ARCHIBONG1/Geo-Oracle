---
name: seismic-qi
description: Quantitative interpretation and amplitude analysis - prerequisites for trusting amplitudes, wavelet, synthetic and well tie, AVO classes, coloured inversion, the DHI criteria screen and 4D differences, tuning, and the strict limits on fluid and lithology claims from post-stack data. Load for any question about amplitude anomalies, DHIs, fluids, lithology or reservoir quality.
---

# Amplitude and quantitative interpretation

## Prerequisites: check them before any amplitude statement

1. **Gain.** `seismic_qc` amplitude check: is POSSIBLE_GAIN_OR_AGC or CLIPPING flagged? Does the task or textual header mention AGC? If amplitudes may be gained, relative amplitude statements are unreliable. Say so, and cap every amplitude-based claim at `proposed`.
2. **Polarity.** Stated, or `unknown`? Without polarity, "high-impedance" or "low-impedance" cannot be read from the sign of the amplitude.
3. **Phase.** From `seismic_qc`, modulo 180°. Non-zero phase shifts the peak away from the interface, and mixes peaks and troughs.
4. **Tuning.** From `seismic_resolution_limits`. For beds near or below tuning thickness, amplitude depends on thickness, and may brighten with no change in rock or fluid.
5. **Noise.** The SNR at the target. Below about 2, isolated amplitude anomalies may be noise.

Record the status of all five in `limitations` whenever you discuss amplitude.

## Inputs from other specialists

| Tool | Needs | Usually from |
|---|---|---|
| `seismic_synthetic` | well logs (depth_m, rhob_g_cc, vp_m_s or dt_us_ft) and a time-depth table for the well: the petrophysics products `elastic_logs` and `time_depth` (references `@petrophysics-agent/products/...`) | wells_petrophysics |
| `seismic_well_tie` | the synthetic, the well location (inline/crossline via `seismic_convert_coordinates`) | as above |
| `seismic_avo`, AVO part of `seismic_dhi_screen` | near and far stacks on the same grid and scaling, with their representative angles | data provider |
| `seismic_4d_difference` | base and monitor surveys on the same grid | data provider |

Register supplied tables (`seismic_register_table`, kinds well_logs and time_depth) or read CSV files from the shared folder. Never assume angles, logs or a time-depth relation; if missing, write the `missing_data` entry (examples below).

## Procedures

1. **Wavelet and tie.** `seismic_extract_wavelet` (statistical; phase assumed) → `seismic_synthetic` → `seismic_well_tie`.
   - **Tie procedure with petrophysics products**, in this order:
     1. Locate each well: `seismic_convert_coordinates(well_header=<ref>)`.
     2. Seabed check: `seismic_trace_events(water_depth_m=<from the well header or the task>)` at each well. `offset_ms` near 0 means the seismic datum is sea level; a consistent offset at both wells is the datum correction.
     3. Datum of the well relation: if the time-depth product gives a non-zero time at depth 0 (the petrophysics result says so), the relation is referenced to another datum. Ask Geo Oracle to have wells_petrophysics re-reference it (`td_time_shift_ms`), or proceed with `max_shift_ms` of 100 so the tie can find the offset, and report it.
     4. Synthetic and tie. Read `shift_at_search_edge` and `alternative_shifts`: a shift at the edge means widen and rerun; two alternatives within a few hundredths of correlation mean the shift is ambiguous.
     5. Markers against events: `seismic_trace_events(markers_table=<tops_time ref>)` and report which events sit at the markers.
     6. Depth conversion from the calibration well: `seismic_time_to_depth(method="table", table=<its time_depth ref>)`. The result's `velocity_model.datum_shift_applied_ms` states any re-referencing it did; report it.
     7. Cross-well check: `seismic_depth_function_check(time_depth=<calibration well's ref>, markers=<check well's tops_time ref>)`: residuals per marker and their summary. Residuals sharing a sign point at a datum offset between the wells before any velocity conclusion.
   - **Reporting a tie**: correlation, phase, shift, the window, and the seabed check, each as a measurement with its provenance; the cross-well residuals likewise. A correlation below 0.6 is reported as a weak tie with the reasons tried (datum, window, phase), not as no tie.
   - **With petrophysics products**: pass the `elastic_logs` and `time_depth` references as they are. Both give depth below the same seismic reference datum, as their sidecars state: check that datum against the survey's.
   - **Well position**: `seismic_convert_coordinates(well_header=<the @petrophysics-agent/products/…well_header….json reference>)` reads the product's X/Y and CRS and returns the inline/crossline. Both wells must land inside the survey: if not, the well and survey CRS differ, so stop and report it. A well header without X/Y is a `missing_data` item for wells_petrophysics.
   - **Textual header**: `seismic_scan_segy` decodes it (EBCDIC or ASCII, reported as `textual_header_encoding`); read it for datum, polarity and CRS statements before assuming any.
   - **Datum check (marine) before the tie**: pick the seabed at each well.
     - If the seismic datum is sea level, the seabed is at about 2 × water depth / 1,480 m/s (e.g. 118 m of water gives about 159 ms).
     - The difference between the seabed pick and the well time-depth's seabed time is a datum offset. Report it as a measurement so that Geo Oracle can have wells_petrophysics correct the time-depth (`td_time_shift_ms`) before the tie.
   - **A bulk shift of more than a sample or two** after a checkshot-calibrated time-depth points at a datum mismatch or wrong checkshot times: raise it with wells_petrophysics, rather than shifting silently.
   - **No `elastic_logs` product in the task?** A tie cannot be made from markers and events by eye. Do the seabed check and `seismic_trace_events` at the well (marker times from the `tops_time` product), publish nothing as a tie, and put `wells_petrophysics: elastic_logs product (vs_source none is enough) — needed for the synthetic` in `missing_data`.
   - **`seismic_trace_events`** gives the first strong event (the seabed) and the strongest events at the well, and matches marker times to their nearest event: use it for the datum check and to report which events sit at the markers. It is context for the tie, not a substitute for the synthetic.
   - The tie measures the bulk shift and constant phase. Phase near 0 means the data are consistent with the synthetic's polarity convention (positive reflection coefficient = peak); near 180 means reversed; near ±90 means the data are not zero-phase.
   - When `next_best_correlation_60deg_away` is close to the best, phase and shift trade off (a time shift looks like a phase rotation). Report both and constrain the shift with checkshots before relying on the phase.
   - A tie below 0.6 correlation fixes nothing.
2. **AVO.** `seismic_avo` with a horizon at the reflector gives intercept (A), gradient (B) and class maps. The class rules and near-zero limit are in the result; quote them. Classes depend on stack scaling: if near and far are not balanced together, class II vs III is unreliable.
3. **Coloured inversion.** `seismic_coloured_inversion` gives relative impedance: compare values laterally (low relative impedance = softer than the surroundings). The default exponent assumes white reflectivity; say so, or use a value derived from supplied impedance logs. Input must be zero-phase (tie first).
4. **DHI screen.** `seismic_dhi_screen` on the candidate top horizon, with near/far stacks when available. It reports for the strongest anomaly: amplitude strength, conformance to structure (edge level), flat spot (z, fraction of the anomaly, agreement with the edge level), frequency shadow and AVO class, each `met`, `not_met` or `indeterminate` with the measurement.
   - A flat spot matching the conformance edge level is the strongest seismic criterion; still check for multiples and processing flats.
   - `not_met` is informative too: report it, don't drop it.
5. **4D.** `seismic_4d_difference`: first read the background NRMS (repeatability). Only changes well above it are interpretable. The change regions are measurements; "gas-water contact rose" is an interpretation needing production data.

## Rules for amplitude claims

- The screen's criteria are **measurements** and **observations**. One criterion alone never supports a fluid claim, and all criteria from the same volume are one line of evidence.
- A statement like "gas-charged sand" is at best a **hypothesis**, `proposed`, until well data or independent QI (a tie plus AVO consistent with rock physics from wells_petrophysics) supports it.
- Always give the alternatives: a lithology contrast (tight carbonate, coal, volcanic or cemented layer), tuning, a processing or gain artefact, a multiple, or a residual-moveout effect.

## Not available

Fluid factor and rock-physics templates, three-term or prestack AVO (gathers), model-based or absolute inversion, and attenuation (Q) estimation have no tool. Without the inputs above, write `missing_data` entries such as:
- `wells_petrophysics: DT, RHOB and VSHALE logs with checkshot for W1 — CSV depth_m,vp_m_s,rhob_g_cc and twt_ms,depth_m — needed for a synthetic tie that fixes polarity and phase`;
- `data_provider: near and far angle stacks over the prospect with their angle ranges — SEG-Y — needed for AVO class`.

## Reporting

- Amplitude values are **measurements**, with provenance.
- A DHI criterion being met is an **observation**.
- "Consistent with hydrocarbons" is an **interpretation**, with alternatives.
- A fluid or lithology prediction is a **hypothesis**. Its `limitations` include the prerequisites above and the missing independent data.
