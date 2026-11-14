---
name: sed-core-description
description: The fixed description vocabulary (Wentworth grain-size classes, structure and contact codes, the bioturbation index). It also covers how to standardise a description table, and how to code lithofacies by the stated rules. Load before sed_standardise and sed_code_lithofacies.
---

# Core description and lithofacies coding

## The vocabulary (`sed_standardise`)

- Grain size: Wentworth classes clay, silt, mud, very fine, fine, medium, coarse and very coarse sand, granule, pebble, cobble. Each class has a phi midpoint and a grain code (cl, si, mu, vf, f, m, c, vc, gr, pb, cb: mud is mu, medium sand m). The tool maps aliases (vf, med, sandstone, shale). It reads the leading Wentworth phrase of a written size ("fine-grained", "medium sand, 1.4 phi", "fL"). Without such a phrase, it reads a stated phi, mm or micron value. A value on a class boundary goes to the coarser class, a decimal comma is read as a point, and a fraction ("1/4 mm") is read. A bounded value ("<0.063 mm", "up to 4 mm", "max 2 mm", "at least 1 mm") gives no class. Of several values, it uses the one labelled mean, median or D50; with no such label it gives no class. One range rule applies to written and numeric ranges ("fine to medium", "fL-mU", "2-3 phi", "125-250 um"): the class at the phi midpoint of the range, a midpoint on a class limit going to the coarser class (Wentworth classes are equal steps in phi). The two limit classes stay in `grain_size_range`. An unlabelled stated value beside a written range wins ("very fine to fine (3.3 phi)" is very fine). A clay-silt range ("clay to fine silt", "silt and clay") is mud. A range class never contradicts the rock noun: a sandstone described "silt to very fine sand" is very fine sand (the finest sand class in the range), and a mudstone, siltstone, shale or claystone whose range midpoint is sand takes the coarsest non-sand class. It codes an interval without a recognisable grain size `x`, and that interval cannot carry grain-size statements.
- Structures: codes from keywords in the description: Xb cross-bedding, Xbb bidirectional (herringbone), Md mud drapes, Tb tidal bundles, Rh tidal rhythmites (neap-spring cyclic laminae), Hcs hummocky, Wr wave ripples, Cr current ripples, Pl planar lamination, Ng normal grading, Sm sole marks, Ht heterolithic, Rt roots or palaeosol, Bi bioturbation, Tf trace fossils, Lac lateral accretion, Ms massive, Cv convolute, Gf grainflow and pin-stripe, Lg lag or intraclasts.
- Fauna: freshwater-biota (charophytes, gyrogonites, land snails, "freshwater fauna" or a named freshwater group; the bare word "freshwater" is not biota), Tf:marine (Ophiomorpha, Skolithos, Thalassinoides, Cruziana, foraminifera, ammonites), Tf:brackish (low diversity, Planolites, Teichichnus).
- Contacts: sharp, erosive, gradational, loaded.
- Bioturbation index 0-6 (Taylor & Goldring 1993). The tool sets any other value to none, keeps the original (`bioturbation_index_rejected`), and warns. A 7 is a typo or another scale, and the tool never clamps it.
- A feature that the text names as absent ("no mud drapes", "roots absent", "bioturbation: none") or with doubt ("possible", "?") gets no code. `sed_standardise` lists these per interval in `result.negated_or_uncertain_features`, and the `facies_log` keeps them in its `negated_features` and `uncertain_features` columns. The reading is rule-based and can be wrong: read these lists against the description before you say that a feature is absent. A feature coded present can still be negated or doubted in wording the rules do not read, and abbreviations are not read. The largest error is invisible: on two unseen tests about 4 in 10 doubtful mentions (41-47 %) were still coded present and are in no list. The causes are doubt words outside the rules (putative, equivocal, perhaps, believed, unconfirmed), a doubt or negation in a following clause that refers back ("..., but this is uncertain"), and abbreviations. The wrong negations (about 2 %, all listed) and missed negations (about 2 %) are smaller. Read the description's own words for any feature that carries a conclusion. "trace", "rare" and "minor" features are observations, and get their codes. A partial absence is not a negation: "absent to rare" and "locally absent" are observations, and "mostly absent" or "absent above 25 m" are uncertain.
- The tool keeps the describer's original words beside the codes. A keyword that the vocabulary misses is a limitation to report, not a feature to invent.
- A grain size comes only from grain-size words. "sandstone" with no size stated is coded `S-x-…` (sand, size unknown), never medium sand. "conglomerate" is coded `G-x-…`, never pebble.
- Mud, clay and silt are grain sizes in themselves: "mud", "mudstone" and "shale" are mud, "clay" and "claystone" clay, "silt" and "siltstone" silt.
- The rock word that names the lithology is its last one before "with", "and", "/" or a comma: "silty sandstone" is a sandstone (size unknown), "sandy claystone" a claystone, "sandstone with mud drapes" a sandstone. "A and B, interbedded" takes A as the rock. When A and B are sand or gravel with mud, the interval gets `heterolithic: true` (sand with gravel is not heterolithic). `sed_interval_totals` then totals it under `H` (heterolithic), and `sed_baffle_summary` counts it as intra-bed mud, not as clean sand. A bare "mudstone" is the carbonate only beside another Dunham class word ("mudstone/wackestone") or when a carbonate qualifier modifies it ("dolomitic mudstone", "mudstone, dolomitic", "calcareous mudstone", "mudstone, calcareous", and any later comma or bracket item such as "mudstone, silty, calcareous" or "mudstone (calcareous)"). An item that names another noun ("calcareous nodules") or is qualified ("calcareous in part") does not count. A qualifier in a "with" phrase does not change the class: "mudstone with calcareous nodules" is clastic. The tool also reads the lithology text for structures.
- The lithology noun decides the lithology class per interval:
  - clastic intervals are coded as above;
  - carbonates are coded texture-allochem-fabric by the `sed-carbonates` skill's rules (read it);
  - evaporites `E` and coals `C`;
  - a lithology that no class recognises is `U`, listed in `lithologies_outside_the_vocabulary`.

## Coding (`sed_code_lithofacies`)

Code = lithology letter (S sand, M mud, G gravel) + grain code + dominant structure. The tool chooses the dominant structure by a fixed priority that puts diagnostic structures first (Rh, Tb, Md, Xbb, Hcs, Gf, Lac, Xb, Sm, Ng, Wr, Cr, Ht, Vv, Pl, Cv, Rt, Lg, Bi, Ms). An interval with no structure code ends in `x` ("no structure coded", for example `S-f-x` or `cW-x-x`). It is not massive and not laminated. The description can still hold words outside the vocabulary: they stay in each interval's `original` in the standardised file, and the tool warns with the counts. A described massive bed keeps `Ms`. The scheme (`facies_scheme` product) records every feature seen in each facies, with its version. The `facies_log` carries the code per interval, with its source (core description or cuttings description) and the depth reference.

`sed_code_lithofacies` also draws the graphic log of each well (the six wells with most described thickness), and returns it in `result.figures`. Publish it (sed-visualisation).

## Thicknesses

Quote thicknesses from `sed_interval_totals`, not from your own addition. It gives:
- thickness by well, facies and lithology;
- the gaps between described intervals (uncored or undescribed, not absent rock);
- the true vertical thickness, if you supply the cosine of the hole angle for a deviated well. The tool refuses a factor outside 0 to 1, and refuses intervals of one well that overlap.

For baffles and heterogeneity, use `sed_baffle_summary`. Per well, it gives:
- the baffle beds: intervals of mud (M), or beds that the description states as cemented and tight ("calcite-cemented", "well cemented", "tight streak", "cement occludes the porosity", "cemented, no visible porosity"). "Cemented throughout", "impermeable" and "pervasive ... cement" also make one. "Poorly cemented", "minor cement", rim or isopachous cement, cement-filled fractures and concretions do not make a baffle. Leaching, dissolution or grain-coating "pervasive" cement is not tight, and "impermeable" or "tight" said of another rock ("sealed by impermeable shale above") is not this bed. A tight statement limited to part of the interval ("in the lower 10 cm only", "upper half", "basal 20 cm", "locally", "patchy") is counted apart (`intervals_with_partial_tight_cement`), not as a baffle. Touching baffle intervals count as one bed. You get their thickness and the spacing between them (`min_baffle_m` sets the thinnest bed that counts, default 0);
- the sand packages between baffles, other lithologies and undescribed gaps (amalgamation);
- the described-sand fraction (a ratio of described thickness, not net to gross). A cemented, tight sand is a baffle, not sand;
- the count of intervals with mud drapes or heterolithic bedding, and the count of intervals with concretions;
- the evaporite beds, as a class of their own. A mud and an evaporite bed that touch count as one seal unit, and no spacing is measured across an evaporite bed. Coal is not a baffle, but it ends a sand package.

The result gives the gap tolerance (`gap_tolerance_m`). The tool refuses intervals of one well that overlap. It leaves out cuttings. It is a one-dimensional record at the well: it cannot show whether a baffle or a sand body is continuous between wells. Say so wherever you quote it.

A described thickness is not net reservoir, not connected thickness and not pore volume. The difference matters to every consumer of the number.

## Cuttings

From cuttings, the structures are unreliable, and the lithology proportions are smeared over the sample interval. Code lithology and grain size only, and say so. The tool does this for a table registered as `cuttings_description`: it lists the structure words, the depositional carbonate fabrics, microbial fabric and fauna or trace-fossil codes in `structures_from_cuttings` and does not code them, it marks the `facies_log` rows as cuttings, and the transition tools refuse the table. Give the `facies_log` in `depends_on` of `sed_interpretation_tests`: a model on cuttings gets S1 not tested, never passed as described.

## Reporting

- Codes: `[observation, derived]`, with the scheme version.
- The scheme's features: the basis of every later step.

<!-- end of skill: sed-core-description -->
