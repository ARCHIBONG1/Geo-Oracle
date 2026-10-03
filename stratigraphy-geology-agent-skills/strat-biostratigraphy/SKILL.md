---
name: strat-biostratigraphy
description: Biostratigraphic events as tie points - sample-type rules, event ordering across wells, graphic correlation, reworking and caving, and where the calibrated ages of datums come from. Load before stg_order_events or stg_graphic_correlation, and before using biostratigraphic events as anchors.
---

# Biostratigraphy

## Events and what they are worth

- A **last occurrence (LO)** is usable from cuttings: caving carries younger material down, so the highest occurrence of a taxon survives; its depth may still be too high through reworking.
- A **first occurrence (FO)** from cuttings is not a datum: caving places it too high. From core or sidewall samples it is. `stg_register_table` marks each event `usable_as_datum`.
- Confidence and sample type travel with the event; a datum of low confidence anchors nothing.

## `stg_order_events`

The composite order across wells and the pairs that reverse between wells. A conflict means one event is reworked, caved or diachronous between those wells; it is reported as such and the event is demoted to rank 5 for those wells. No conflicts does not prove synchroneity: a diachronous event can keep its order between other events and still cross a time line; that shows in C2 against the correlated surfaces and in graphic correlation.

## `stg_graphic_correlation`

Shaw's line of correlation: the depths of shared events in two wells. The slope is the ratio of sedimentation rates; a straight line with small residuals supports continuous deposition at proportional rates; a breakpoint with a step is a candidate gap (positive step: missing section in well B) or rate change. The largest residuals are the suspect events: diachronous, reworked or mispicked.

## Ages for datums

A biostratigraphic datum carries no age by itself. Its calibrated age comes from a published zonation, requested from `literature_review` (`literature_review: calibrated age of the LO of <taxon> in the <scheme> zonation — Ma with error and time scale — anchor the age model`). Never an age from memory: an uncited age is a hypothesis, and the age model built on it is capped.

## Reporting

Events as `[observation, project_data]`; the composite order and the graphic-correlation line as `[measurement, derived]` with `prov:` ids; a reworking or diachroneity reading as `[interpretation]`.
