---
name: strat-unconformities-thickness
description: Isopachs between panel surfaces, eroded thickness under an unconformity, telling erosion from fault cut-outs with the structural fault network, missing time, zonation and the framework product. Load before stg_isopach, stg_eroded_thickness, stg_gap_vs_fault, stg_zonation or stg_framework.
---

# Unconformities, thickness and the framework

## `stg_isopach`

Thickness between two panel surfaces per well (TVDSS), a plane fitted across the wells, residuals and the coefficient of variation. A residual far from the plane is the anomaly that C4 flags; the product `isopach_trends` goes to sedimentology (depositional trends), structural geology (growth) and prospect work.

## `stg_eroded_thickness`

The thickness removed at a well where a unit is truncated, projected from the plane through wells where it is complete (three at least). The projection assumes a linear trend; a depositional edge or growth breaks that assumption, and the uncertainty is the plane's RMS residual.

## `stg_gap_vs_fault`

Before a gap is called an unconformity: the structural `fault_network` (or a fault set with throws) is searched within a radius of the well; a fault with throw of at least half the anomaly can explain missing section (a cut-out) or repeated section (a repeat). If one exists, the gap stays a structural candidate and `structural_geology` is asked to confirm; if none does, erosion or non-deposition remains.

## Missing time

From the age model: the ages just above and below the surface, with the range from the envelopes (see `strat-chronostratigraphy`). A gap resolvable in time and in thickness is an unconformity; one resolvable in neither is reported as unresolved.

## `stg_zonation`

Zones bounded by the admitted panel's surfaces, per well with MD and TVDSS and the weaker tie rank of the two surfaces. The product `zonation` goes to `wells_petrophysics` for zone averages; zones built on a panel that is not admitted inherit its status.

## `stg_framework`

The `stratigraphic_framework` product: the admitted panel with its ledger, the alternatives with theirs, the unconformities, and the attached products (correlated_tops, age_model, zonation, isopach_trends, correlation_alternatives). It is the skeleton the other specialists hang their work on; its status is the admitted panel's.

## Reporting

Thicknesses, eroded thickness and missing time as `[measurement, derived]`; the erosion-versus-fault reading as `[interpretation]` with the structural product cited; the framework as `[interpretation]` with its ledger.
