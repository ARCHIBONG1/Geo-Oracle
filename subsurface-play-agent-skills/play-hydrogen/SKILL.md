---
name: play-hydrogen
description: Natural (native) hydrogen plays and underground hydrogen storage - the element sets, what lifts each, why timing and seal differ from hydrocarbons (active generation, a small mobile molecule, microbial consumption), and how the upstream products map to each. Load before linking the elements of a natural_hydrogen or hydrogen_storage play.
---

# Hydrogen plays

## Natural hydrogen (domain `natural_hydrogen`)

Most natural hydrogen systems are active: the gas is being made now by serpentinisation of iron-rich rocks, by radiolysis in uranium- and thorium-rich basement, or by degassing. So the questions differ from a fossil petroleum charge: is generation ongoing, is there a path, is there a seal that holds hydrogen, and is the gas surviving its sinks.

| Element | Demonstrated by | Inferred from | Owner |
|---|---|---|---|
| source | ultramafic or iron-rich rocks, or U-Th-rich basement, in the study area; hydrogen isotopes pointing at the process | a demonstrated flux (a measured product proves its source) | regional_geology, supplied fluid_geochem, literature_review |
| generation_activity | present-day hydrogen in soil gas or wells with a flux; temperatures in the serpentinisation window; isotopes showing recent production | the source rock at the right temperature | supplied seeps and fluid_geochem; wells_petrophysics (temperature_profile) |
| migration | seeps or shows along faults or fractures from the source | conductive faults in the network | supplied seeps and shows; structural_geology |
| reservoir | net reservoir and porosity in wells; fractured basement with tested permeability | facies model; fault network | wells_petrophysics, sedimentology, structural_geology |
| seal | salt or evaporite; clay with a measured hydrogen entry pressure | a conventional top seal (inferred at most: a seal that holds methane is not shown to hold hydrogen) | wells_petrophysics, stratigraphy, structural_geology |
| trap | a depth closure passing its ledger; a demonstrated stratigraphic trap | a closure with a failing ledger | structural_geology |
| preservation | temperature above the microbial window (about 90-120 °C) or hydrogen present despite it; no abiotic sink; low diffusive loss | nothing known: unknown | wells_petrophysics, supplied fluid_geochem, literature_review |

Timing (PT4): with ongoing generation (`holds: true` on the generation event), a trap and seal present today suffice. A fossil hydrogen charge needs a trap that predates it and a seal that retained hydrogen since; that is rare and is said so.

A measured hydrogen flux demonstrates generation_activity and lifts source to inferred; seeps and shows demonstrate migration and lift generation and source; a discovery proves the chain. The matrix applies these.

## Underground hydrogen storage (domain `hydrogen_storage`)

| Element | Demonstrated by | Owner |
|---|---|---|
| storage_unit | a salt body thick and pure enough for caverns, or a porous unit with net reservoir and porosity | wells_petrophysics, sedimentology, stratigraphy |
| injectivity_deliverability | permeability from core or tests; cycling rates | wells_petrophysics |
| caprock_containment | hydrogen entry pressure measured; continuity across the site | wells_petrophysics, stratigraphy |
| fault_containment | fault seal and stability over the operating pressure range with measured stress | structural_geology |
| well_containment | legacy-well inventory with integrity records; hydrogen-rated completions | supplied legacy_wells; literature_review |
| geochemical_microbial_stability | brine and mineralogy showing no hydrogen-consuming reactions (sulphate reduction to H2S, methanogenesis, pyrite reduction) at reservoir temperature | wells_petrophysics, sedimentology |
| cycling_integrity | caprock and wells holding through repeated pressure cycles | structural_geology, supplied test_summaries |

Containment is kept separate as for CO2; the microbial and geochemical element is the one storage assessments most often omit, and a porous-media site without brine chemistry leaves it unknown.

## Pitfalls

- A hydrocarbon seal reported as a hydrogen seal: inferred at most without a hydrogen entry pressure.
- Hydrogen in soil gas taken as a reservoir: it demonstrates generation and migration, not a trap.
- A storage site screened on capacity without the microbial element.

## Reporting

As for the other domains: the matrix, the weakest link, the discriminating evidence (usually seal entry pressure, preservation data and, for storage, brine chemistry and well integrity first).
