---
name: strat-unconformities-thickness
description: Isopachs between panel surfaces, and eroded thickness under an unconformity. It covers how to tell erosion from fault cut-outs with the structural fault network, missing time, zonation and the framework product. Load before stg_isopach, stg_eroded_thickness, stg_gap_vs_fault, stg_zonation or stg_framework.
---

# Unconformities, thickness and the framework

## `stg_isopach`

The tool gives the thickness between two panel surfaces per well:
- the isochore (true vertical thickness, TVT). In a vertical well, TVT is the TVDSS difference;
- with the bed dip at each well (`dip_deg_by_well`), the isopach (true stratigraphic thickness, TST = TVT cos dip; Tearpock & Bischke 2003).

In a deviated well, the TVDSS difference is the true vertical depth thickness (TVDT). Where beds dip, TVDT is not TVT (Tearpock & Bischke 2003, pp. 96-101). Give the bed dip and the dip azimuth (`dip_azimuth_deg_by_well`) for each deviated well. The tool then computes TVT and TST from the well path between the picks (Eq. 4-6). Without them, the value stays TVDT: the tool lists the well in `tvdt_wells`, warns and sets a `tvdt_not_tvt` quality flag. `stg_eroded_thickness` takes the same `dip_azimuth_deg_by_well`. A well within 3 degrees of vertical between the picks counts as vertical (the error is then at most about 3 % at a 30 degree dip). Each well states its own `thickness_basis`. If a well's path crosses the bed against the pick order (it climbs up-section, or runs nearly parallel to the bedding), it gets no thickness: the tool names it in `inconsistent_wells`, leaves it out of the fit, warns and sets an `inconsistent_geometry` flag. Check its picks and its dip.

If every well has a dip, the tool fits the plane to isopachs. Otherwise, it fits isochores, which change with dip as well as thickness. Say which (`thickness_basis`). It posts each well where it crosses the interval, from the trajectory that the panel stores. If the panel has none, rebuild it, or the tool warns and uses the median log position. It gives the residuals and the coefficient of variation. A residual far from the plane is the anomaly that C4 flags. The tool reads the picks from the panel you give it, and nowhere else. `picks_from` names that panel (its file and the `provenance_id` of `stg_build_panel`), in the result and in the product. If another panel beside it picks the same surfaces at other depths in the same wells, the tool names it in `other_panels_with_different_picks` and warns. Build the isopach on the admitted panel, and say which panel it rests on. The product `isopach_trends` goes to sedimentology (depositional trends), structural geology (growth) and prospect work. The tool draws the isopach map itself (thickness posted at the wells, with the planar trend), and returns it in `result.figures`. `stg_render` kind `isopach` draws it again from the product.

The tool reads the panel's status and its comparisons, which `stg_compare_panels` keeps beside the panel. A crossing found by a comparison made before either panel changed still counts, worded as possibly out of date; compare again. On a panel that is `lithostratigraphic_only` or declared lithostratigraphic, the thickness is rock-unit thickness, not a time-interval trend, and a limitation says so. On a contradicted panel the tool still computes, with a limitation and a `thickness_on_contradicted_panel` flag. Below `partially_supported`, a limitation names the status.

## `stg_eroded_thickness`

The tool projects the thickness removed at a well where a unit is truncated, from the plane through wells where it is complete (three at least). It is a minimum estimate of the section removed, since younger units may have gone too (Corcoran & Dore 2005). Give dips for isopachs. The projection assumes a linear trend. A depositional edge or growth breaks that assumption. The uncertainty is the plane's RMS residual.

## `stg_gap_vs_fault`

Run it before you call a gap an unconformity.
- The tool searches the structural `fault_network`, or a fault set with throws, within a radius of the well. The network can carry each fault's trace, depth, dip and throw. The tool refuses a network with no fault traces (topology only). A fault without a throw cannot be matched: the tool lists it in `throw_unknown`, and a fault stays possible.
- It measures to the nearest point on each trace, with the well located at the gap's depth (`gap_tvdss_m`).
- A well with no position cannot be placed against the network. The position is unknown when the `position` fact of the `correlation_logs` sidecar says missing or incomplete, when x, y are blank, or when the first point of the path is exactly 0, 0. Otherwise the tools place the well from its path. An old product without the `position` fact can publish a missing location as 0 plus the path's offsets: the tools cannot tell that from a real location. If you doubt a well's coordinates, ask wells_petrophysics. The reading is then "not tested: well position unknown", with a `missing_data` entry. `stg_isopach` and `stg_eroded_thickness` leave such wells out and warn. `stg_eroded_thickness` refuses with fewer than three placed complete wells.
- If the fault set carries depth and dip, the tool carries the fault plane down to the well. The plane must cut the well within `depth_tolerance_m` of the gap (the way fault cuts are mapped: Tearpock & Bischke 2003). Without depth and dip, the match rests on throw and distance, and the result says so (`plane_not_tested`).
- At a normal fault, the missing section equals the vertical separation (Tearpock & Bischke 2003). So a fault explains missing section (a cut-out) or repeated section (a repeat) only when its throw matches the anomaly within the tolerance (20 % by default).
- Section missing at the same horizon in other wells (`other_wells_missing`) points to an unconformity instead. So do a change of dip across the surface and a gap that grows up-structure.
- If a fault matches, the gap stays a structural candidate. Ask `structural_geology` to confirm it. If faults were given and none matches, erosion or non-deposition remains.
- Without a fault network or faults (an empty list counts as none), the fault side is not tested (`fault_side`), with a `missing_data` entry for the structural fault network. Absent faults are no evidence for erosion. For missing section, section missing in other wells still points to an unconformity, but a fault is not excluded. A repeated section is not explained by an unconformity.

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

It is the skeleton that the other specialists hang their work on. Its status is the admitted panel's. A panel that is `lithostratigraphic_only` or declared lithostratigraphic is not admitted as a time framework: the product records it as a rock-unit framework (`admitted_as`), and a limitation says so. A declared lithostratigraphic panel is always a rock-unit framework. A crossing contradiction of the admitted panel is in `admitted_panel.contradicted_by_crossing`.

## Reporting

- Thicknesses, eroded thickness and missing time: `[measurement, derived]`.
- The erosion-versus-fault reading: `[interpretation]`, with the structural product cited.
- The framework: `[interpretation]`, with its ledger.

<!-- end of skill: strat-unconformities-thickness -->
