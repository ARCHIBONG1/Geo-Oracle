---
name: play-petroleum-systems
description: Hydrocarbon play elements and processes, and what evidence lifts each. Also the events chart and the critical moment, the reasoning on timing, migration and preservation, and the common errors. Load before linking the elements of a hydrocarbon play.
---

# Petroleum systems

## The elements and what lifts each

| Element | Demonstrated by | Inferred from | Analogue-supported from | Common error |
|---|---|---|---|---|
| source | study-area samples with TOC and Rock-Eval; an oil-source correlation | shows or seeps with a stated source | — | regional source maps used as local proof |
| maturation | measured Ro or Tmax in the kitchen | the regional burial and thermal history calibrated locally | — | present-day temperature used as the history |
| migration | shows or hydrocarbons along the path | a path on the depth surface respecting fault seal | — | migration through a fault the structural agent found sealing |
| reservoir | net reservoir and porosity in study-area wells; described facies | facies model | analogue net-to-gross | analogue net-to-gross taken as local |
| seal | pressure differences or retained columns | seal facies correlated between wells; fault seal | — | seal from lithology alone |
| trap | none from a map alone; a well result on the closure (a column or contact seen in a well inside it) | a depth closure passing V1 and V7, closed in every realization (a trap_geometry link: inferred at most) | — | time-domain closures; a closure with a failing ledger used as if it passed |
| timing | inferred at most: trap age and expulsion are dated by interpretation | trap age and expulsion both dated, trap first | — | one side undated and assumed |
| preservation | no post-charge uplift, breach or cool exposure in the burial history | nothing later known (then unknown, not safe) | — | preservation assumed |

Trap and timing usually come from interpretation links (the structural trap_geometry, dated claims), and from an interpretation link they are inferred at best. Never relink an interpretation as an observation to reach demonstrated. Regional or analogue evidence lifts an element to analogue-supported at most.

A discovery in the play area lifts the charge chain to inferred: source, maturation, migration and timing, where they stood lower.

The template also holds the supporting element `overburden`.

## The events chart (`play_events_chart`)

The chart holds these events, each from a cited claim with a range: deposition of source, reservoir, seal and overburden; trap formation; generation and expulsion; migration; preservation.

The critical moment is the point in time you select that best depicts the generation-migration-accumulation of most hydrocarbons (Magoon and Dow 1994, p. 11). Read it from the burial history or maturity model, and give it as `critical_moment: {age_ma, basis}`. It lies inside the generation-migration interval (in their example 250 Ma within 260-240 Ma). The tool flags an age outside it. Without it, the chart reports that interval and no single age. It is never derived from the end of expulsion.

Each `trap` event is a separate trap: give one event per trap. The growth stages of one trap are one event, whose end_ma is the end of its last stage. Give each charge pulse as its own `expulsion` (or `generation`) event. The earliest-completed trap is read against each pulse, and `timing.pairs` lists which trap catches each pulse.

PT4 tests trap formation against expulsion:
- a trap complete before expulsion passes;
- one formed during expulsion passes, with partial charge noted;
- one formed after it fails.

The tool draws the events chart in `result.figures`. Publish it.

## Migration (no tool yet: your reasoning, with cited evidence)

The fetch area of a closure is what drains into it on the depth surface. Faults that the structural agent found sealing are barriers, and leaking ones are conduits. Shows along the path are the study-area evidence.

## Preservation

The threats are uplift after charge (burial history), breach by faulting (fault stability, structural timing), biodegradation at cool shallow depths (temperature history) and flushing. State an unknown as unknown.

## Reporting

Give the chart as `[interpretation]` with its events' sources, and each element in the fixed claim form. Give the play claim capped by the weakest.

<!-- end of skill: play-petroleum-systems -->
