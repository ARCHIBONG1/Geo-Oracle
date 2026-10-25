---
name: sed-depositional-systems
description: The diagnostic matrix per depositional environment, and how an association becomes an environment. It also covers the interpretation tests S1-S7, competing models and the depositional_model product. Load before sed_diagnostic_check and sed_interpretation_tests, and before naming any environment.
---

# Depositional systems

## The matrix (as the tools apply it)

| Environment | Diagnostic | Suggestive only | Against |
|---|---|---|---|
| Fluvial channel | Lateral accretion with unidirectional currents (accretion surfaces dipping at a high angle to flow: Miall 2010) | Roots or palaeosols in the overbank (Rt: subaerial exposure, found in many settings), cross-bedding, current ripples, grading, lags, fining upward | Mud drapes, tidal bundles, herringbone, marine traces |
| Tidal channel | Mud drapes on foresets (Md), tidal bundles (Tb) | Bidirectional (herringbone) cross-bedding (Xbb, not unique to tides: Nio & Yang 1991), heterolithic bedding, cross-bedding, fining upward, brackish traces | Roots, hummocky |
| Tidal flat | Md, Tb | Heterolithic, current ripples, bioturbation | Hummocky (roots are not against: the upper flats are vegetated, Fan 2013) |
| Wave-dominated shoreface | Hummocky or swaley cross-stratification (Hcs), wave ripples (Wr) | Coarsening upward, planar lamination, marine traces, bioturbation | Roots, mud drapes, tidal bundles |
| Deep-water turbidite system | Grading with planar lamination and hemipelagic mud | Grading with sole marks (turbidity currents also build delta fronts: Bhattacharya 2010), grading, convolute lamination, massive sands | Roots, wave ripples, hummocky, herringbone |
| Aeolian dune | Grainflow and pin-stripe lamination (Gf) | Large cross-sets, good sorting, red colour | Bioturbation, marine traces, mud drapes, hummocky |
| Lacustrine | Varves (annual laminae in a lake) | Evaporites with lamination (also in marine basins: Kendall 2010), planar lamination, organic-rich mud | Marine traces, hummocky |

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
