---
name: strat-unconformities-thickness
description: Isopachs between panel surfaces, and eroded thickness under an unconformity. It covers how to tell erosion from fault cut-outs with the structural fault network, missing time, zonation and the framework product. Load before stg_isopach, stg_eroded_thickness, stg_gap_vs_fault, stg_zonation or stg_framework.
---

# Unconformities, thickness and the framework

## `stg_isopach`

The tool gives the thickness between two panel surfaces per well:
- the isochore (true vertical thickness, TVDSS difference);
- with the bed dip at each well (`dip_deg_by_well`), the isopach (true stratigraphic thickness, TST = TVT cos dip; Tearpock & Bischke 2003).

If every well has a dip, the tool fits the plane to isopachs. Otherwise, it fits isochores, which change with dip as well as thickness. Say which (`thickness_basis`). It posts each well where it crosses the interval, from the trajectory that the panel stores. If the panel has none, rebuild it, or the tool warns and uses the median log position. It gives the residuals and the coefficient of variation. A residual far from the plane is the anomaly that C4 flags. The product `isopach_trends` goes to sedimentology (depositional trends), structural geology (growth) and prospect work. The tool draws the isopach map itself (thickness posted at the wells, with the planar trend), and returns it in `result.figures`. `stg_render` kind `isopach` draws it again from the product.

## `stg_eroded_thickness`

The tool projects the thickness removed at a well where a unit is truncated, from the plane through wells where it is complete (three at least). It is a minimum estimate of the section removed, since younger units may have gone too (Corcoran & Dore 2005). Give dips for isopachs. The projection assumes a linear trend. A depositional edge or growth breaks that assumption. The uncertainty is the plane's RMS residual.

## `stg_gap_vs_fault`

Run it before you call a gap an unconformity.
- The tool searches the structural `fault_network`, or a fault set with throws, within a radius of the well. The network carries each fault's trace, depth, dip and throw, and the tool refuses one published without them.
- It measures to the nearest point on each trace, with the well located at the gap's depth (`gap_tvdss_m`).
- If the fault set carries depth and dip, the tool carries the fault plane down to the well. The plane must cut the well within `depth_tolerance_m` of the gap (the way fault cuts are mapped: Tearpock & Bischke 2003). Without depth and dip, the match rests on throw and distance, and the result says so (`plane_not_tested`).
- At a normal fault, the missing section equals the vertical separation (Tearpock & Bischke 2003). So a fault explains missing section (a cut-out) or repeated section (a repeat) only when its throw matches the anomaly within the tolerance (20 % by default).
- Section missing at the same horizon in other wells (`other_wells_missing`) points to an unconformity instead. So do a change of dip across the surface and a gap that grows up-structure.
- If a fault matches, the gap stays a structural candidate. Ask `structural_geology` to confirm it. If no fault matches, erosion or non-deposition remains.

## Missing time

Run `stg_age_model` with the surface's depth in `hiatuses_tvdss`. The tool builds the model separately above and below the surface, and returns the missing time with its range (see `strat-chronostratigraphy`). A gap resolvable in time and in thickness is an unconformity. A gap resolvable in neither is unresolved. State it as such in the findings.

## `stg_zonation`

Zones bounded by the admitted panel's surfaces, per well, with MD and TVDSS and the weaker tie rank of the two surfaces. The product `zonation` goes to `wells_petrophysics` for zone averages. Zones built on a panel that is not admitted inherit its status.

## `stg_framework`

The `stratigraphic_framework` product holds:
- the admitted panel with its ledger;
- the alternatives with theirs;
- the unconformities;
- the attached products (correlated_tops, age_model, zonation, isopach_trends, correlation_alternatives).

It is the skeleton that the other specialists hang their work on. Its status is the admitted panel's.

## Reporting

- Thicknesses, eroded thickness and missing time: `[measurement, derived]`.
- The erosion-versus-fault reading: `[interpretation]`, with the structural product cited.
- The framework: `[interpretation]`, with its ledger.

<!-- end of skill: strat-unconformities-thickness -->
