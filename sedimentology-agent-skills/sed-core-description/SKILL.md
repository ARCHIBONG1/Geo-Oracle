---
name: sed-core-description
description: The fixed description vocabulary (Wentworth grain-size classes, structure and contact codes, the bioturbation index), standardising a description table, and coding lithofacies by the stated rules. Load before sed_standardise and sed_code_lithofacies.
---

# Core description and lithofacies coding

## The vocabulary (`sed_standardise`)

- Grain size: Wentworth classes clay, silt, mud, very fine, fine, medium, coarse and very coarse sand, granule, pebble, cobble, with a phi midpoint; aliases (vf, med, sandstone, shale) are mapped; an interval without a recognisable grain size is coded `x` and cannot carry grain-size statements.
- Structures: codes from keywords in the description: Xb cross-bedding, Xbb bidirectional (herringbone), Md mud drapes, Tb tidal bundles, Hcs hummocky, Wr wave ripples, Cr current ripples, Pl planar lamination, Ng normal grading, Sm sole marks, Ht heterolithic, Rt roots or palaeosol, Bi bioturbation, Tf trace fossils, Lac lateral accretion, Ms massive, Cv convolute, Gf grainflow and pin-stripe, Lg lag or intraclasts.
- Fauna: Tf:marine (Ophiomorpha, Skolithos, Thalassinoides, Cruziana, foraminifera, ammonites), Tf:brackish (low diversity, Planolites, Teichichnus).
- Contacts: sharp, erosive, gradational, loaded. Bioturbation index 0-6 (Taylor & Goldring 1993).
- The describer's original words are kept beside the codes; a keyword the vocabulary misses is a limitation to report, not a feature to invent.

## Coding (`sed_code_lithofacies`)

Code = lithology letter (S sand, M mud, G gravel) + grain code + dominant structure, the dominant chosen by a fixed priority that puts diagnostic structures first (Tb, Md, Xbb, Hcs, Gf, Lac, Xb, Sm, Ng, Wr, Cr, Ht, Pl, Cv, Rt, Lg, Bi, Ms). The scheme (`facies_scheme` product) records every feature seen in each facies, with its version; the `facies_log` carries the code per interval with its source (described) and the depth reference.

## Thicknesses

Quote thicknesses from `sed_interval_totals`, not from your own addition: it gives thickness by well, facies and lithology, the gaps between described intervals (uncored or undescribed, not absent rock), and the true vertical thickness when you supply the cosine of the hole angle for a deviated well. A described thickness is not net reservoir, not connected thickness and not pore volume, and the difference matters to every consumer of the number.

## Cuttings

From cuttings the structures are unreliable and the lithology proportions are smeared over the sample interval; code lithology and grain size only, and say so.

## Reporting

Codes as `[observation, derived]` with the scheme version; the scheme's features as the basis of every later step.
