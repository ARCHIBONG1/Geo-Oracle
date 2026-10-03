---
name: strat-chronostratigraphy
description: Age models from dated samples and datums with cited ages, the seeded envelope, sedimentation rates, missing time, C3 and C7, time-scale discipline, and the age_model product. Load before stg_age_model and before testing C3 or C7.
---

# Chronostratigraphy

## `stg_age_model`

- Inputs per well: dated samples from the `age_dates` table (rank 1) and biostratigraphic datums with cited ages (rank 2), each with its error and source. At least two per well.
- Piecewise-linear age against depth with a seeded Monte Carlo envelope (p10-p90) over the datum errors. The seed is recorded; the same seed reproduces the envelope.
- Reversals (a deeper datum younger than a shallower one beyond their errors) fail C3: reworking, a fault repeat, or a wrong age. Say which, or leave it failing.
- Sedimentation rates between datums are reported; a rate that changes by an order of magnitude between intervals points at a condensed section or a gap.

## Ages of surfaces and missing time

- A correlated surface's age in each well is read from that well's model (the consistency tool does this for C3: the surfaces' ages must not reverse down any well).
- Missing time across a surface is the difference between the model ages just below and just above it, with the range from the envelopes; it is resolvable only when the range excludes zero.
- A well without a datum inside an interval inherits the interpolation between the nearest datums: the Wheeler diagram shows it, and the limitation is stated.

## Time scales

Every age states its time scale; the regional chart's scale is named in its sidecar. Ages on different scales are not compared until converted (`regional_geology: convert ages to GTS2020 — table — compare with the chart`). C7 tests the surfaces' ages against the chart's span and order, given the chart's transfer status.

## Reporting

Dated samples as `[measurement, project_data]`; modelled ages, rates and missing time as `[measurement, derived]` with the seed and `prov:` id; ages assigned to surfaces as `[inference, derived]`; product `age_model` per well.
