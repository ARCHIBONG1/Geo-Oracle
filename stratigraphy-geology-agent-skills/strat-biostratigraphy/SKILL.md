---
name: strat-biostratigraphy
description: Biostratigraphic events as tie points. It covers sample-type rules, event ordering across wells, graphic correlation, reworking and caving, and where the calibrated ages of datums come from. Load before stg_order_events or stg_graphic_correlation, and before using biostratigraphic events as anchors.
---

# Biostratigraphy

## Events and what they are worth

- A **last occurrence (LO)** is usable from cuttings. Caving carries younger material down, so the highest occurrence of a taxon survives. Reworking can still make its depth too high.
- A **first occurrence (FO)** from cuttings is not a datum: caving places it too high. From core or sidewall samples, it is a datum. `stg_register_table` marks each event `usable_as_datum`.
- Confidence and sample type travel with the event. A datum of low confidence anchors nothing.

## `stg_order_events`

The tool gives the composite order across wells by RASC presorting (Agterberg & Gradstein 1999). An event scores one for every event that it lies above in more wells than below. It scores a half when the two are coeval on average. The tool lists the pairs that reverse between wells, with their crossover frequency. A conflict means that one event is reworked, caved or diachronous between those wells. State it as such in the findings, and demote the event to rank 5 for those wells. No conflicts does not prove synchroneity. A diachronous event can keep its order between other events and still cross a time line. That shows in C2 against the correlated surfaces and in graphic correlation.

## `stg_graphic_correlation`

The tool fits Shaw's line of correlation: the depths of shared events in two wells, fitted by least squares as Shaw did.
- The line is a chain of connected segments whose slopes are zero or positive. A slope is the ratio of sedimentation rates (B per A).
- A straight line with small residuals supports continuous deposition at proportional rates.
- The tool keeps a second or third segment only when the F test accepts it (`loc.selection`). Small changes of slope cannot be detected (Edwards 1984).
- A segment that bridges the gap between two events, and that is flatter than its neighbours (zero: a terrace), is a candidate hiatus in well B. A bridging segment that is steeper than its neighbours is a candidate hiatus in well A (`gaps`).
- Give `fixed_events` for time-equivalent points, such as an ash bed or a dated horizon. The line passes through them.
- The largest residuals are the suspect events: diachronous, reworked, caved or mispicked.
- `order_reversed` means that the events do not keep their order between the wells.

## Ages for datums

A biostratigraphic datum carries no age by itself. Its calibrated age comes from a published zonation. Request it from `literature_review` (`literature_review: calibrated age of the LO of <taxon> in the <scheme> zonation — Ma with error and time scale — anchor the age model`). Never take an age from memory. An uncited age is a hypothesis, and the age model built on it is capped.

## Reporting

- Events: `[observation, project_data]`.
- The composite order and the graphic-correlation line: `[measurement, derived]`, with `prov:` ids.
- A reworking or diachroneity reading: `[interpretation]`.

<!-- end of skill: strat-biostratigraphy -->
