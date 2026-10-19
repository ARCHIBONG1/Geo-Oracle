---
name: sed-carbonates
description: Carbonate rocks in the sedimentology specialist - the texture-first vocabulary (Dunham; Embry & Klovan), allochems, skeletal groups, fabrics and diagenesis, the carbonate facies code, the platform-to-basin environment matrix and how diagenesis is reported. Load before standardising, coding or interpreting any interval whose lithology is a carbonate, a mixed carbonate-clastic succession, an evaporite or a coal.
---

# Carbonates

## 1. Texture first

A carbonate is classified by its depositional texture, not its grain size: how much mud it holds and whether the grains or the mud support it (Dunham 1962), with Embry & Klovan's (1971) terms for coarse and organically bound rocks. `sed_standardise` reads it from the description's words:

| Code | Texture | Read from |
|---|---|---|
| M | lime mudstone | "lime mudstone", micrite, calcilutite, chalk, marl; "mudstone" in a carbonate interval |
| W | wackestone | mud-supported, more than 10 % grains |
| P | packstone | grain-supported with mud |
| G | grainstone | grain-supported, no mud; calcarenite, oosparite, oolite |
| B, Bf, Bb, Ba | boundstone: framestone, bindstone, bafflestone | organisms bound the sediment in place |
| F, R | floatstone, rudstone | more than 10 % of grains over 2 mm, mud- or grain-supported |
| X | crystalline | dolostone, recrystallised, sucrosic |

A described Wentworth size is kept when the describer gave one, but a grainstone's "grains" are ooids or shells: the carbonate code carries the allochem, not a sand class.

## 2. Grains, fabrics, diagenesis

- **Allochems**: `oo` ooids and pisoids, `pe` peloids, `in` intraclasts, `on` oncoids, `bi` bioclasts.
- **Skeletal groups**: `Cor` corals, `Rud` rudists, `Str` stromatoporoids, `Spg` sponges (framebuilders); `Alg` calcareous algae, `For` benthic foraminifera (platform); `Mol`, `Ech`, `Bry`, `Bra` (open marine); `Mic` microbial fabrics (stromatolites, thrombolites); `Pel` pelagic fauna (planktonic foraminifera, coccoliths, radiolaria, ammonites). Miliolids, dasyclads, peneroplids, ostracods and "low diversity" mark a **restricted fauna**.
- **Fabrics** (depositional): `Fe` fenestral, `Tp` tepee, `Hg` hardground, `Ev` evaporite nodules or casts, `Br` breccia or debrite.
- **Diagenesis** (overprint, never an environment): `Do` dolomitisation, `Vg` dissolution and moldic or vuggy porosity, `Ka` karst, `Ce` cementation, `St` stylolites, `Ch` chert.
- The clastic structure codes (`Xb`, `Pl`, `Ng`, `Hcs`, `Wr`, `Cr`, `Bi`, `Tf`, `Cv`, `Md`, `Rt`) apply to carbonates too: a cross-bedded ooid grainstone is `cG-oo-Xb`.

`sed_code_lithofacies` codes a carbonate interval **c + texture-allochem-fabric** (`cG-oo-Xb`, `cM-x-Fe`, `cBf-x-Ms`, `cX-x-Do`), the dominant fabric chosen by a priority that puts the diagnostic fabrics first, then diagenesis, then the structures. The leading `c` keeps a carbonate code apart from a clastic one: the Dunham letters M and G are also the clastic mud and gravel letters, so a lime mudstone `cM-…` is never the same facies as a mud `M-…`, nor a grainstone `cG-…` a conglomerate `G-…`; the described-thickness totals count carbonates under `c`. Evaporites code `E`, coals `C`; a lithology no class recognises is `U` and is reported in `lithologies_outside_the_vocabulary`.

## 3. The carbonate matrix (`sed_diagnostic_check`)

Environments are Wilson's (1975) standard facies belts as Flügel (2010, Box 14.1) defines the facies zones (FZ); each result names its FZ and carries a note from the texts:

| Environment (FZ) | Diagnostic | Suggestive | Against |
|---|---|---|---|
| pelagic basin / outer ramp (FZ 1A-1B) | pelagic fauna (planktonic forams, coccoliths, radiolaria, calpionellids, ammonites) | lime mudstone, planar lamination, bioturbation, chert, grading | fenestral, tepee, evaporites, ooid grainstone, framebuilders, cross-bedding |
| carbonate slope (FZ 3-4) | graded beds or breccias with a pelagic matrix | grading, breccia, rudstone, floatstone, slumps, pelagic fauna | fenestral, tepee, evaporites, ooid grainstone, roots, cross-bedding |
| reef / platform margin (FZ 5) | boundstone with framebuilders (corals, rudists, stromatoporoids, sponges) | rudstone, floatstone, the framebuilders, algae, early cement | fenestral, tepee, evaporites, pelagic, roots |
| ooid shoal (FZ 6) | ooid grainstone | grainstone, cross-bedding, ooids, peloids, coated grains, hardgrounds | lime mudstone, fenestral, evaporites, pelagic |
| open marine platform (FZ 7, and FZ 2 deep shelf) | a diverse normal-marine fauna (two or more groups including echinoderms, brachiopods or bryozoans) in a non-bound texture | wackestone, packstone, lime mudstone, bioturbation, algae, benthic forams, molluscs | fenestral, tepee, evaporites, pelagic, roots |
| lagoon / platform interior (FZ 8) | restricted fauna (miliolids, dasyclads, peneroplids, ostracods, low diversity) | wackestone, lime mudstone, packstone, peloids, algae, benthic forams, bioturbation, oncoids, dolomitisation | fenestral, tepee, pelagic, ooid grainstone, hummocky |
| carbonate tidal flat (FZ 8, its tidal-flat part) | fenestral fabric, tepees, microbial lamination | evaporite nodules, planar lamination, dolomitisation, mud drapes, roots, lime mudstone, peloids, restricted fauna | framebuilders, pelagic, hummocky |
| sabkha / evaporitic platform (FZ 9A) | nodular, chicken-wire or laminated gypsum/anhydrite | planar lamination, microbial lamination, dolomitisation, lime mudstone, fenestral, tepee | open-marine fauna, pelagic, framebuilders, ooid grainstone |

The composites are formed by the tool from the features and your flags: an ooid grainstone needs `oo` and texture `G`; a graded bed with pelagic matrix needs `Ng` and either `Pel` in the fauna or `hemipelagic=true`; the open-marine fauna is formed from the skeletal groups (or `open_marine_fauna=true`); `pelagic=true` and `restricted_fauna=true` are flags for what the fauna list shows. The S1-S7 ledger, the transition test and the association rung are the clastic ones; only the vocabulary, the code and the matrix differ.

**Three cautions from the texts.** Fenestrae, tepees, mudcracks and evaporites show subaerial exposure, not a tidal flat as such (Tucker & Wright 1990, p. 138): report "peritidal / exposed" unless the succession shows the flat. FZ 7 and FZ 8 grade into each other and FZ 2 (deep shelf) and FZ 7 share their microfacies (Flügel 2010, p. 661), so the open-platform and lagoon verdicts are decided by the succession and the neighbouring belts as much as by one association. On a ramp the ooid shoal lies in the inner or mid ramp, not at a platform edge (Flügel 2010, p. 681).

## 4. What to say and not say

- A belt, not a water depth: "lagoon / platform interior" is a statement about restriction and energy, and the matrix cannot separate a ramp from a rimmed shelf. A ramp passes seaward without a marked break in slope, lacks a continuous reef trend, keeps its high-energy sands near the shoreline, and its deep-water breccias, if any, lack clasts of shallow shelf-edge facies (Read 1985, p. 2; restated by Flügel 2010, sect. 14.1.4; Tucker & Wright 1990, sect. 2.6, pp. 47-49). A distally steepened ramp has its slope break many kilometres seaward of the shoals, so its breccias hold deep-ramp clasts, not shallow-platform ones (Read 1985, p. 3). Which platform it was is a model you state with its alternative.
- Bafflestone is the most interpretive texture (Embry & Klovan 1971, p. 737: "a good imagination on the part of the geologist"; Wright 1992 retains it with reservations): report it as an interpretation. Wright (1992) prefers "calcimudstone" for the lime mudstone; both words are read.
- Diagenesis is reported as features with the code (`Do`, `Vg`, `Ka`, `Ce`), as `[observation]`; its timing and its control on porosity are `[interpretation]`s that need petrography or core measurements (`wells_petrophysics`), and reservoir-quality statements about carbonates rest on them more than on the belt.
- Mixed successions: the class is decided per interval by the lithology noun ("sandy limestone" is a carbonate, "calcareous sandstone" a clastic); the transition test runs on the whole coded succession.
- A carbonate word the vocabulary misses is a limitation to report, as for the clastic vocabulary.

## Methodological provenance

The matrix and the vocabulary carry method ids (`m:sed/dunham-texture-classification`, `m:sed/carbonate-lithofacies-coding`, `m:sed/carbonate-environment-matrix`); carry them into `source_reference` as `sed-core` says. Their sources were read against the code (Dunham 1962; Embry & Klovan 1971; Wright 1992; Wilson 1975; Flügel 2010; Tucker & Wright 1990): the registry holds the locators (`sed_method_provenance`).

<!-- end of skill: sed-carbonates -->
