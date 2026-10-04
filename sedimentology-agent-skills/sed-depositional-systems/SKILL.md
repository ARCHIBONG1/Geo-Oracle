---
name: sed-depositional-systems
description: The diagnostic matrix per depositional environment, how an association becomes an environment, the interpretation tests S1-S7, competing models and the depositional_model product. Load before sed_diagnostic_check and sed_interpretation_tests, and before naming any environment.
---

# Depositional systems

## The matrix (as the tools apply it)

| Environment | Diagnostic | Suggestive only | Against |
|---|---|---|---|
| Fluvial channel | Roots or palaeosols in the overbank (Rt); lateral accretion with unidirectional currents | Cross-bedding, current ripples, grading, lags, fining upward | Mud drapes, tidal bundles, herringbone, marine traces |
| Tidal channel | Mud drapes on foresets (Md), tidal bundles (Tb), bidirectional cross-bedding (Xbb) | Heterolithic bedding, cross-bedding, fining upward, brackish traces | Roots, hummocky |
| Tidal flat | Md, Tb | Heterolithic, current ripples, bioturbation | Hummocky, roots |
| Wave-dominated shoreface | Hummocky or swaley cross-stratification (Hcs), wave ripples (Wr) | Coarsening upward, planar lamination, marine traces, bioturbation | Roots, mud drapes, tidal bundles |
| Deep-water turbidite system | Grading with sole marks; grading with planar lamination and hemipelagic mud | Grading, convolute lamination, massive sands | Roots, wave ripples, hummocky, herringbone |
| Aeolian dune | Grainflow and pin-stripe lamination (Gf) | Large cross-sets, good sorting, red colour | Bioturbation, marine traces, mud drapes, hummocky |
| Lacustrine | Varves; evaporites with lamination | Planar lamination, organic-rich mud | Marine traces, hummocky |

`sed_diagnostic_check` takes the features seen in the association (from the scheme) and the succession pattern (fining or coarsening upward; unidirectional palaeocurrents when known) and returns S2 with the lists.

## From association to environment

1. One diagnostic feature at least, in the association, not in a neighbouring unit.
2. The succession consistent with the environment (S3 from the pooled transitions).
3. The framework and setting consistent (S4): a tidal channel below a flooding surface in a marine succession fits; one inside a red-bed alluvial unit does not.
4. The biota consistent (S7): brackish traces in a tidal channel; marine traces in a shoreface; none in an aeolian dune.
5. Palaeocurrents (S6, phase 1) and geometry (S5, phase 2) when their products exist.

## Competing models

Whenever the features admit two environments, run `sed_diagnostic_check` for both and `sed_interpretation_tests` for both; report the admitted one with its ledger, the other as a hypothesis with its failing or suggestive-only test, and the discriminator in `missing_data`.

## Reporting

The environment as `[interpretation]` with the ledger and rung in the statement; the product `depositional_model` with its tests, rung, status and `depends_on`; the alternative as `[hypothesis]`.
