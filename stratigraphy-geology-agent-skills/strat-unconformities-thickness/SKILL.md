---
name: strat-unconformities-thickness
description: Isopachs between panel surfaces, eroded thickness under an unconformity, telling erosion from fault cut-outs with the structural fault network, missing time, zonation and the framework product. Load before stg_isopach, stg_eroded_thickness, stg_gap_vs_fault, stg_zonation or stg_framework.
---

# Unconformities, thickness and the framework

## `stg_isopach`

Thickness between two panel surfaces per well: the isochore (true vertical thickness, TVDSS difference) and, with the bed dip at each well (`dip_deg_by_well`), the isopach (true stratigraphic thickness, TST = TVT cos dip; Tearpock & Bischke 2003). The plane is fitted to isopachs when every well has a dip, otherwise to isochores, which change with dip as well as thickness: say which (`thickness_basis`). Each well is posted where it crosses the interval, from the trajectory the panel stores (a panel built before 2026-10-18 lacks it: rebuild it, or the tool warns and uses the median log position). Residuals and the coefficient of variation are reported. A residual far from the plane is the anomaly that C4 flags; the product `isopach_trends` goes to sedimentology (depositional trends), structural geology (growth) and prospect work.

## `stg_eroded_thickness`

The thickness removed at a well where a unit is truncated, projected from the plane through wells where it is complete (three at least): a minimum estimate of the section removed, since younger units may have gone too (Corcoran & Dore 2005). Give dips for isopachs. The projection assumes a linear trend; a depositional edge or growth breaks that assumption, and the uncertainty is the plane's RMS residual.

## `stg_gap_vs_fault`

Before a gap is called an unconformity: the structural `fault_network` (which carries each fault's trace, depth, dip and throw; one published without them is refused) or a fault set with throws is searched within a radius of the well, measured to the nearest point on each trace, with the well located at the gap's depth (`gap_tvdss_m`). Where the fault set carries depth and dip, the fault plane is carried down to the well and must cut it within `depth_tolerance_m` of the gap (the way fault cuts are mapped: Tearpock & Bischke 2003); without them the match rests on throw and distance and says so (`plane_not_tested`). At a normal fault the missing section equals the vertical separation (Tearpock & Bischke 2003), so a fault explains missing section (a cut-out) or repeated section (a repeat) only when its throw matches the anomaly within the tolerance (20 % by default). Section missing at the same horizon in other wells (`other_wells_missing`) points to an unconformity instead; so do a change of dip across the surface and a gap that grows up-structure. If a fault matches, the gap stays a structural candidate and `structural_geology` is asked to confirm; if none does, erosion or non-deposition remains.

## Missing time

From `stg_age_model` with the surface's depth in `hiatuses_tvdss`: the model is built separately above and below the surface and the missing time returned with its range (see `strat-chronostratigraphy`). A gap resolvable in time and in thickness is an unconformity; one resolvable in neither is reported as unresolved.

## `stg_zonation`

Zones bounded by the admitted panel's surfaces, per well with MD and TVDSS and the weaker tie rank of the two surfaces. The product `zonation` goes to `wells_petrophysics` for zone averages; zones built on a panel that is not admitted inherit its status.

## `stg_framework`

The `stratigraphic_framework` product: the admitted panel with its ledger, the alternatives with theirs, the unconformities, and the attached products (correlated_tops, age_model, zonation, isopach_trends, correlation_alternatives). It is the skeleton the other specialists hang their work on; its status is the admitted panel's.

## Reporting

Thicknesses, eroded thickness and missing time as `[measurement, derived]`; the erosion-versus-fault reading as `[interpretation]` with the structural product cited; the framework as `[interpretation]` with its ledger.
