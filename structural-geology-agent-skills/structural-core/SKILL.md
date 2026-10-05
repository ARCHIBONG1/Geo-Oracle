---
name: structural-core
description: The structural geology specialist's working rules - task procedure, the never-invent rule, the validity ledger V1-V7, alternatives by default, the dependency ceiling, claim status, products, missing_data and the rendering standards. Load at the start of every structural task.
---

# Structural core


## What a figure is and is not

A figure illustrates a claim a tool computed. It never carries a claim, and no number, depth, area or boundary is read off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested, which is the one way a picture can damage an argument.

You do not see the figures you render: the tool returns a URL, not an image. They are for the reader and for the audit, so the caption must say what the figure shows, from which product and version, well enough to be read without you.

## 1. What the task needs from you

| Structural question | Primary methods | Supporting | Never |
|---|---|---|---|
| Is the fault framework consistent? | Orientation statistics; the ledger (V1, V2, V7 when the tools exist) | Throw profiles | Accepting picks without reading their sidecar (domain, uncertainty) |
| Will the fault slip? | `sg_stress_tensor` then `sg_fault_stability` as ranges; `sg_andersonian_check` | Seismicity, breakouts | One deterministic stress state; a silent friction value |
| Will the fault seal? | `sg_fault_seal` with a petrophysics Vsh log and a sourced throw | Pressure differences across the fault | SGR without Vsh data; thresholds without a calibration |
| What do the measurements say? | `sg_orientation_stats`, `sg_fold_axis`, sets | Stereonets | Means across different structural domains |
| Where are closure and spill? | `sg_closure_spill` on depth surfaces, as ranges over realizations | `sg_build_model` for the ledger | Time-domain closures reported as traps; one number instead of a range |

## 2. Never invent structure

Every fault, fold, closure, relationship and orientation in your findings traces to a supplied table or an upstream product, cited by reference. What you suspect but cannot trace is a `hypothesis` with the data that would test it in `missing_data`. The self-check rejects any observation or claim without a source.

## 3. The validity ledger

| Test | Question | Tool (phase) |
|---|---|---|
| V1 geometric | Consistent cutoffs, no crossing horizons | `sg_build_model` |
| V2 displacement | Smooth throw, displacement-length within the global population | `sg_throw_profile`, `sg_dl_check` |
| V3 restorability | Balances within tolerance | sg_balance_check (2) |
| V4 kinematic | Slip senses and timing consistent with cross-cutting and the regional phases | `sg_fault_network` + regional framework |
| V5 mechanical | Orientation compatible with the regime, or explained as reactivation | `sg_andersonian_check` (0) |
| V6 timing | Growth strata and cross-cutting give one sequence | `sg_growth_index` |
| V7 data honouring | Honours the picks within their uncertainty | `sg_build_model` |

Record pass, fail or not tested with the reason, in the statement of every model interpretation and in the product sidecars. A failed test is reported, with its explanation if one exists (reactivation, linkage); without one the model is `contradicted`. Not tested is honest: V3 (restoration) waits for its tools.

## 3a. Caveats are ceilings

The task brief and the upstream findings will carry caveats: a Vsh log that is uncalibrated, a stress magnitude that is only bounded, a pore pressure that is assumed, a regional prior with transfer partly unknown. Each one lowers the status of the claim that rests on it (`partially_supported` at most) and goes into `assumptions` and `limitations`. None of them stops a computation. The tools exist to run with stated assumptions and ranges; the ceiling then says how far the result can be trusted. A brief that says "do not calculate X without Y" is read as "calculate X with Y as a flagged assumption or range, and cap the claim", unless Y is a product the tool itself refuses to run without (a depth surface for a closure). `insufficient_data` is reported only after the tools have been run or have refused.

Restating an upstream number as your own `derived` measurement is a class error: either cite the upstream value as `upstream_specialist`, or compute it with a tool and cite the provenance id.

## 4. Alternatives by default

Where the data permit more than one model (a single fault or two overlapping segments; a relay or a breach; a planar or a listric geometry), state each, test each, and report ties as `unresolved` hypotheses with the discriminating evidence requested: `seismic_interpretation: picks on crosslines 45-58 — fault sticks — distinguish a relay from a single breached fault`.

## 5. The dependency ceiling

Every applied claim lists what it depends on in `depends_on_claim_ids` (upstream claim ids such as `T04-reg/C1`) and in the product's `depends_on`. Its status cannot exceed the weakest of those. Say which dependency sets the ceiling and what would lift it (local breakouts for a regional stress azimuth; a measured Shmin for a bounded one).

## 6. Products and requests

- `fault_stability`, `fault_seal`, `displacement_analysis`, `fault_network`, `structural_model` and `trap_geometry` now; `restoration`, `fracture_model` and `structural_timing` later. Sidecars carry domain, the ledger, `depends_on` and the parameters.
- `missing_data` format: `<specialist>: <item> — <form> — <why>`, with the gateway's specialist names (`seismic_interpretation`, `wells_petrophysics`, `regional_geology`, `stratigraphy`, `literature_review`), never agent folder names. Picks and depth surfaces from `seismic_interpretation`; Vsh, pressures, breakouts and fluid densities from `wells_petrophysics`; stress and tectonic phases from `regional_geology`; ages from `stratigraphy`.
- For CO2 storage and geothermal tasks, fault stability is reported as ranges with its ceiling even when not asked.

## 7. Rendering standards (enforced by `sg_render`)

Stereonets: lower hemisphere, equal area, count and contouring printed. Rose: bin width and count printed. Mohr: failure line with friction and cohesion stated; resolved planes drawn; the stress state printed. Seal profiles: throw and cutoffs printed. Describe a plot before interpreting it; a relation seen in a plot enters the findings only with a tool result behind it.

## 8. Self-check before emitting

1. Every observation and claim traces to a reference or `prov:` id; nothing invented.
2. Every model interpretation carries its ledger; every applied claim its `depends_on_claim_ids` and ceiling.
3. Every surface's domain stated; nothing time-domain used for closures or stress.
4. Every parameter sourced or flagged as an assumption with its range.
5. Alternatives stated where the data permit.
6. Products as `Product: …` lines; figures as `Figure: …` lines.
