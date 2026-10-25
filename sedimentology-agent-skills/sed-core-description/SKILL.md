---
name: sed-core-description
description: The fixed description vocabulary (Wentworth grain-size classes, structure and contact codes, the bioturbation index). It also covers how to standardise a description table, and how to code lithofacies by the stated rules. Load before sed_standardise and sed_code_lithofacies.
---

# Core description and lithofacies coding

## The vocabulary (`sed_standardise`)

- Grain size: Wentworth classes clay, silt, mud, very fine, fine, medium, coarse and very coarse sand, granule, pebble, cobble. Each class has a phi midpoint and a grain code (cl, si, mu, vf, f, m, c, vc, gr, pb, cb: mud is mu, medium sand m). The tool maps aliases (vf, med, sandstone, shale). It codes an interval without a recognisable grain size `x`, and that interval cannot carry grain-size statements.
- Structures: codes from keywords in the description: Xb cross-bedding, Xbb bidirectional (herringbone), Md mud drapes, Tb tidal bundles, Hcs hummocky, Wr wave ripples, Cr current ripples, Pl planar lamination, Ng normal grading, Sm sole marks, Ht heterolithic, Rt roots or palaeosol, Bi bioturbation, Tf trace fossils, Lac lateral accretion, Ms massive, Cv convolute, Gf grainflow and pin-stripe, Lg lag or intraclasts.
- Fauna: Tf:marine (Ophiomorpha, Skolithos, Thalassinoides, Cruziana, foraminifera, ammonites), Tf:brackish (low diversity, Planolites, Teichichnus).
- Contacts: sharp, erosive, gradational, loaded.
- Bioturbation index 0-6 (Taylor & Goldring 1993). The tool sets any other value to none, keeps the original (`bioturbation_index_rejected`), and warns. A 7 is a typo or another scale, and the tool never clamps it.
- The tool keeps the describer's original words beside the codes. A keyword that the vocabulary misses is a limitation to report, not a feature to invent.
- A grain size comes only from grain-size words. "sandstone" with no size stated is coded `S-x-…` (sand, size unknown), never medium sand. "conglomerate" is coded `G-x-…`, never pebble.
- Mud, clay and silt are grain sizes in themselves: "mud", "mudstone" and "shale" are mud, "clay" and "claystone" clay, "silt" and "siltstone" silt.
- The rock word that names the lithology is its last one: "silty sandstone" is a sandstone (size unknown), "sandy claystone" a claystone.
- The lithology noun decides the lithology class per interval:
  - clastic intervals are coded as above;
  - carbonates are coded texture-allochem-fabric by the `sed-carbonates` skill's rules (read it);
  - evaporites `E` and coals `C`;
  - a lithology that no class recognises is `U`, listed in `lithologies_outside_the_vocabulary`.

## Coding (`sed_code_lithofacies`)

Code = lithology letter (S sand, M mud, G gravel) + grain code + dominant structure. The tool chooses the dominant structure by a fixed priority that puts diagnostic structures first (Tb, Md, Xbb, Hcs, Gf, Lac, Xb, Sm, Ng, Wr, Cr, Ht, Pl, Cv, Rt, Lg, Bi, Ms). The scheme (`facies_scheme` product) records every feature seen in each facies, with its version. The `facies_log` carries the code per interval, with its source (described) and the depth reference.

`sed_code_lithofacies` also draws the graphic log of each well (the six wells with most described thickness), and returns it in `result.figures`. Publish it (sed-visualisation).

## Thicknesses

Quote thicknesses from `sed_interval_totals`, not from your own addition. It gives:
- thickness by well, facies and lithology;
- the gaps between described intervals (uncored or undescribed, not absent rock);
- the true vertical thickness, if you supply the cosine of the hole angle for a deviated well.

A described thickness is not net reservoir, not connected thickness and not pore volume. The difference matters to every consumer of the number.

## Cuttings

From cuttings, the structures are unreliable, and the lithology proportions are smeared over the sample interval. Code lithology and grain size only, and say so.

## Reporting

- Codes: `[observation, derived]`, with the scheme version.
- The scheme's features: the basis of every later step.

<!-- end of skill: sed-core-description -->
