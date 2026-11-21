---
name: regional-stratigraphy
description: Time-scale discipline, chronostratigraphic charts from the literature basin_synthesis, hiatuses and overlaps, the regional_framework product and competing tectonic models. Load before reg_time_scale, reg_chronostrat_chart or reg_framework.
---

# Regional stratigraphy

## Time scale: `reg_time_scale`

- The bundled chart is the ICS International Chronostratigraphic Chart v2023/09. Every output names it.
- Ages from older scales (GTS2012, GTS2004) can differ by about a million years at some boundaries. Say which scale each age uses, and never mix them silently.
- Stage names resolve to ages (`name='Tithonian'`), and ages resolve to stages. The tool flags approximate boundaries. Where the chart gives a +/- for a base age, the tool returns it with the age (`base_uncertainty_myr`). Quote both.

## The chart: `reg_chronostrat_chart`

- Input: the literature specialist's `basin_synthesis` product (its `unit` rows), or units given with ages or stage names.
- Output: units ordered by age, with durations and stages, **gaps** (hiatuses or unconformities between successive units) and **overlaps** (conflicting ages or interfingering units).
- Check each gap against the synthesis's `unconformity` rows. A gap that matches a named unconformity is corroborated. A gap that does not is a question for literature_review or stratigraphy.
- The chart is regional (`[regional]`). The study-area correlation belongs to the stratigraphy specialist, who receives the `chronostrat_chart` product.

## The framework: `reg_framework`

- The tool assembles `regional_framework` from the synthesis (settings, dated phases, events), the chart (units per phase), and the measured products you attach (stress_field, seismicity_summary, thermal_model, structural_elements, burial_history).
- **Competing models**: if sources give different phases or ages for the same interval, carry each as a model. Give it the evidence for and against, and the test that would discriminate the models. Never resolve by majority.
- Every statement in the framework carries its source. A phase is the cited author's interpretation. A measured product is this agent's measurement.

<!-- end of skill: regional-stratigraphy -->
