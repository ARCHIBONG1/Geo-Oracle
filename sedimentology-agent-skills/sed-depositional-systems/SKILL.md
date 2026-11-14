---
name: sed-depositional-systems
description: The diagnostic matrix per depositional environment, and how an association becomes an environment. It also covers the interpretation tests S1-S7, competing models and the depositional_model product. Load before sed_diagnostic_check and sed_interpretation_tests, and before naming any environment.
---

# Depositional systems

## The matrix (as the tools apply it)

| Environment | Diagnostic | Suggestive only | Against |
|---|---|---|---|
| Fluvial channel | Lateral accretion with unidirectional currents (accretion surfaces dipping at a high angle to flow: Miall 2010) | Roots or palaeosols in the overbank (Rt: subaerial exposure, found in many settings), cross-bedding, current ripples, grading, lags, fining upward | Mud drapes, tidal bundles, tidal rhythmites (Rh), herringbone, marine traces |
| Tidal channel | Mud couplets or double mud drapes (Md:couplet), tidal bundles (Tb), mud drapes with tidal bundles (Md+Tb) or with herringbone (Md+Xbb) (Visser 1980) | A single mud drape (Md), bidirectional (herringbone) cross-bedding (Xbb, not unique to tides: Nio & Yang 1991), heterolithic bedding, cross-bedding, fining upward, brackish traces | Roots (hummocky is not against: it occurs in estuaries and on intertidal flats, Dott & Bourgeois 1982) |
| Tidal flat | Cyclical tidal rhythmites (Rh: neap-spring cyclic laminae), "diagnostic of tidal-flat deposition" (Fan 2013, p. 74). The same page says estuarine-channel margins also form them: a pass names the estuary in `shared_with`, and the ledger caps the status at `partially_supported` with `unresolved_with`. Otherwise intertidal flats have no unique bedding type (p. 73). Rhythmites are never read as tidal bundles, and they count against a fluvial channel | Tidal bundles (Tb, mainly subtidal), mud drapes (Md), heterolithic, current ripples, bioturbation | None (roots and hummocky are not against: vegetated upper flats, storm HCS on open-coast flats, Fan 2013) |
| Wave-dominated shoreface | The succession of Plint 2010, p. 183, Fig. 20: hummocky (Hcs) sandstone below cross-bedded (Xb) sandstone, coarsening upward. It is diagnostic only when you bind `sed_diagnostic_check` to the coded file (`coded`, `well`) and the tool finds that order in one contiguous sand succession (no gap, no mud between) and an upward coarsening in its coded grain sizes. The `coarsening_upward` flag alone, and a carbonate texture, never make it diagnostic. No single feature, and no wave ripple, is diagnostic. A pass names the rivals the same succession fits (`shared_with`: delta front, lake, storm shelf, carbonate inner and mid ramp). The ledger then caps the status at `partially_supported`, the rung at `association`, and gives `unresolved_with`: report them as unresolved | Hummocky or swaley cross-stratification (Hcs) and wave ripples (Wr) (wave influence: also on the shelf, in lakes and on delta fronts, Dott & Bourgeois 1982), coarsening upward, planar lamination, marine traces, bioturbation | Mud drapes, tidal bundles (roots top the foreshore, Fig. 20D: not against) |
| Deep-water turbidite system | Grading with planar lamination and hemipelagic mud | Grading with sole marks (turbidity currents also build delta fronts: Bhattacharya 2010), grading, convolute lamination, massive sands | Roots, wave ripples, hummocky, herringbone |
| Aeolian dune | Grainflow and pin-stripe lamination (Gf) | Large cross-sets, good sorting, red colour | Bioturbation, marine traces, mud drapes, hummocky |
| Lacustrine | Varves (annual laminae in a lake) | Evaporites with lamination (also in marine basins: Kendall 2010), planar lamination, organic-rich mud | Marine traces (hummocky is not against: it occurs in lakes, Dott & Bourgeois 1982) |
| Delta front | None: the coarsening-upward succession is shared with wave-formed shorefaces (Bhattacharya 2010, p. 246) | Coarsening upward, cross-bedding, current ripples, planar lamination, grading, convolute lamination, wave ripples, hummocky (Bhattacharya 2010, p. 247) | None |
| Storm-dominated shelf | None | Hummocky, planar lamination, bioturbation (Dott & Bourgeois 1982, p. 663) | None |
| Prodelta | None | Heterolithic bedding, planar lamination, grading, massive beds (Bhattacharya 2010, p. 246) | None |
| Estuary | None | Cross-bedding, current ripples, planar lamination, lags, mud drapes, bioturbation, lateral accretion (IHS), fining upward, brackish traces, organic-rich mud (Dalrymple 2010, p. 222; Boyd 2010, pp. 273-274) | None |
| Clastic lagoon | None | Organic-rich mud, bioturbation, planar lamination, coarsening upward at the margins (Boyd 2010, p. 274) | None |

The delta front, storm-dominated shelf, prodelta, estuary and clastic lagoon rows are rivals for the shoreface, the turbidite, the tidal channel and the lake. They have no diagnostic feature, so S2 for them is at most `suggestive_only`. The matrices cannot test an alluvial fan: report such a rival as not tested, never as rejected.

`sed_diagnostic_check` takes two inputs:
- the features seen in the association (from the scheme);
- the succession pattern (fining or coarsening upward; unidirectional palaeocurrents when known).

It returns S2 with the lists.

## From association to environment

1. One diagnostic feature at least, in the association, not in a neighbouring unit.
2. The succession consistent with the environment (S3 from the pooled transitions).
3. The framework and setting consistent (S4). A tidal channel below a flooding surface in a marine succession fits. One inside a red-bed alluvial unit does not.
4. The biota consistent (S7): brackish traces in a tidal channel, marine traces in a shoreface, none in an aeolian dune.
5. Palaeocurrents (S6) and geometry (S5), if evidence for them exists. No tool computes them yet: pass the booleans with their evidence cited, or leave them not tested.

## Competing models

Whenever the features admit two environments, run `sed_diagnostic_check` and `sed_interpretation_tests` for both. Then report:
- the admitted one, with its ledger;
- the other as a hypothesis, with its failing or suggestive-only test;
- the discriminator, in `missing_data`.

## Reporting

- The environment: `[interpretation]`, with the ledger and rung in the statement.
- The product `depositional_model`, with its tests, rung, status and `depends_on`.
- The alternative: `[hypothesis]`.

<!-- end of skill: sed-depositional-systems -->
