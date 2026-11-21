---
name: strat-chronostratigraphy
description: Age models from dated samples and datums with cited ages. It covers the seeded envelope, sedimentation rates, missing time, C3 and C7, time-scale discipline, and the age_model product. Load before stg_age_model and before testing C3 or C7.
---

# Chronostratigraphy

## `stg_age_model`

- Inputs per well: dated samples from the `age_dates` table (rank 1) and biostratigraphic datums with cited ages (rank 2), each with its error and source. Give at least two per well. Give the registered `age_dates` table as `table`: the tool reads the well's rows itself. A row outside the logged interval is skipped with a warning. Type only the biostratigraphic datums in `datums`; a typed datum that repeats a table row is used once.
- The model is piecewise-linear age against depth, never extrapolated beyond the datums. A seeded Monte Carlo envelope (p10-p90) covers the datum errors. The tool rejects a realisation whose ages reverse, as Blaauw (2010) does, and `rejected_realizations` says how many. Many rejections mean that the datums barely keep their order within their errors. The tool records the seed, and the same seed reproduces the envelope.
- Reversals (a deeper datum younger than a shallower one beyond their errors) fail C3. The cause is reworking, a fault repeat, or a wrong age. Say which, or leave it failing.
- The tool gives sedimentation rates between datums. If a rate changes by an order of magnitude between intervals, it points at a condensed section or a gap.
- The tool draws the well's age-depth figure itself, and returns it in `result.figures`.

## Ages of surfaces and missing time

- A correlated surface's age in each well comes from that well's model. The consistency tool does this for C3: the surfaces' ages must not reverse down any well. A surface outside the model's dated range, or at a hiatus, gets no age: the tool lists it in `untested_surfaces`. With no two aged surfaces in a well, C3 is not tested.
- Missing time at an unconformity: give its depth in `hiatuses_tvdss` of `stg_age_model`.
  - The tool then builds the model separately above and below it (Blaauw 2010's hiatus option), and never interpolates across it.
  - `hiatuses` returns the missing time: the age just below minus the age just above. Each age comes from its own side's nearest datums, extrapolated to the surface (stated), with the p10-p90 range.
  - The missing time is resolvable only when the range excludes zero (its p10 is above zero). The tool keeps negative gaps in the realisations, so the range is not forced above zero. `p_gap_positive` gives the share of positive gaps.
  - With a single datum on one side, only the upper bound (the two nearest datums' age difference) is known.

  Without the hiatus, interpolation across it would smear the gap into the rates.
- A well without a datum inside an interval inherits the interpolation between the nearest datums. The Wheeler diagram shows it. State the limitation.

## Time scales

Every age states its time scale. The regional chart's sidecar names the chart's scale. Do not compare ages on different scales as numbers. No tool converts an age from one scale to another. Compare them by stage. Ask `regional_geology: ages of <stage names> on the chart's time scale — reg_time_scale table — compare with the chart`. Or state the difference of scales as a limitation on C3 and C7. C7 tests the surfaces' ages against the chart's span and order, given the chart's transfer status.

## Reporting

- Dated samples: `[measurement, project_data]`.
- Modelled ages, rates and missing time: `[measurement, derived]`, with the seed and `prov:` id.
- Ages assigned to surfaces: `[inference, derived]`.
- Product `age_model` per well.

<!-- end of skill: strat-chronostratigraphy -->
