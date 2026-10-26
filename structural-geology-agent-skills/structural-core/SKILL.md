---
name: structural-core
description: The structural geology specialist's working rules. They cover the structural questions and their methods, the never-invent rule, the validity ledger V1-V7, caveats as ceilings, alternatives by default and the dependency ceiling. They also cover products, missing_data, the rendering standards and the analysis modes. Load at the start of every structural task.
---

# Structural core


## What a figure is and is not

A figure illustrates a claim that a tool computed. It never carries a claim. You never read a number, depth, area or boundary off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested. That is the one way a picture can damage an argument.

You cannot see the figures: a tool returns a URL, not an image. Check the numbers the tool returns, never the picture. Figures are for the person, Geo Oracle's answer and the investigation report. So the caption must say what the figure shows, from which product and version, well enough to be read without you.

## 1. What the task needs from you

| Structural question | Primary methods | Supporting | Never |
|---|---|---|---|
| Is the fault framework consistent? | Orientation statistics; the ledger (V1 and V7 from `sg_build_model`, V2 from `sg_throw_profile` and `sg_dl_check`) | Throw profiles | Accepting picks without reading their sidecar (domain, uncertainty) |
| Will the fault slip? | `sg_stress_tensor` then `sg_fault_stability` as ranges; `sg_andersonian_check` | Seismicity, breakouts | One deterministic stress state; a silent friction value |
| Will the fault seal? | `sg_fault_seal` with a petrophysics Vsh log and a sourced throw | Pressure differences across the fault | SGR without Vsh data; thresholds without a calibration |
| What do the measurements say? | `sg_orientation_stats`, `sg_fold_axis`, sets | Stereonets | Means across different structural domains |
| Where are closure and spill? | `sg_build_model` on the depth horizon, then `sg_closure_spill` on the same horizon with `model` set to that `structural_model`, as ranges over realizations | The model's V1 and V7, carried into the `trap_geometry` validity record | Time-domain closures reported as traps; one number instead of a range; a closure for a prospect without `model` |

## 2. Never invent structure

Every fault, fold, closure, relationship and orientation in your findings traces to a supplied table or an upstream product, cited by reference. What you suspect but cannot trace is a `hypothesis`, with the data that would test it in `missing_data`. The self-check rejects any observation or claim without a source.

## 3. The validity ledger

| Test | Question | Tool |
|---|---|---|
| V1 geometric | Consistent cutoffs, no crossing horizons | `sg_build_model` |
| V2 displacement | Smooth throw, displacement-length within the global population | `sg_throw_profile`, `sg_dl_check` |
| V3 restorability | Balances within tolerance | no tool yet (restoration): record `not tested` |
| V4 kinematic | Slip senses and timing consistent with cross-cutting and the regional phases | `sg_fault_network` + regional framework |
| V5 mechanical | Orientation compatible with the regime, or explained as reactivation | `sg_andersonian_check` |
| V6 timing | Growth strata and cross-cutting give one sequence | `sg_growth_index` |
| V7 data honouring | Honours the picks within their uncertainty | `sg_build_model` |

Assemble the ledger with `sg_consistency`. Give it:
- the runs and products that tested the model;
- a rival model's runs, when one was built;
- the accepted explanation of each failure that you explain.

It merges the tests (a mixed result is a fail), and returns the status that the ledger allows. The model claim takes that status. Record pass, fail or not tested with the reason, in the statement of every model interpretation and in the product sidecars. Report a failed test, with its explanation if one exists (reactivation, linkage). Without an explanation, the model is `contradicted`. Not tested is honest: V3 (restoration) waits for its tools.

## 3a. Caveats are ceilings

The task brief and the upstream findings will carry caveats. Examples: a Vsh log that is uncalibrated, a stress magnitude that is only bounded, a pore pressure that is assumed, a regional prior with transfer partly unknown. Each caveat lowers the status of the claim that rests on it (`partially_supported` at most), and goes into `assumptions` and `limitations`. No caveat stops a computation. The tools exist to run with stated assumptions and ranges. The ceiling then says how far the result can be trusted. Read a brief that says "do not calculate X without Y" as "calculate X with Y as a flagged assumption or range, and cap the claim", unless Y is a product the tool itself refuses to run without (a depth surface for a closure). Report `insufficient_data` only after the tools have been run or have refused.

An upstream number restated as your own `derived` measurement is a class error. Either cite the upstream value as `upstream_specialist`, or compute it with a tool and cite the provenance id.

## 4. Alternatives by default

If the data permit more than one model (a single fault or two overlapping segments; a relay or a breach; a planar or a listric geometry), state each and test each. Report ties as `unresolved` hypotheses, and request the discriminating evidence: `seismic_interpretation: picks on crosslines 45-58 — fault sticks — distinguish a relay from a single breached fault`.

## 5. The dependency ceiling

Every applied claim lists what it depends on, in `depends_on_claim_ids` (upstream claim ids such as `T04-reg/C1`) and in the product's `depends_on`. Its status cannot exceed the weakest of those. Say which dependency sets the ceiling, and what would lift it (local breakouts for a regional stress azimuth; a measured Shmin for a bounded one).

## 6. Products and requests

- `fault_stability`, `fault_seal`, `displacement_analysis`, `fault_network`, `structural_model` and `trap_geometry` are available now. `restoration`, `fracture_model` and `structural_timing` come later. Sidecars carry domain, the ledger, `depends_on` and the parameters.
- A `trap_geometry` for prospect work needs the model's ledger. Build `sg_build_model` on the horizon, then call `sg_closure_spill` with `model=<that structural_model>`. Its V1 and V7 then reach the closure's validity record, and prospect maturity (R2) needs them passed. Without `model` they stay "not tested" and the candidate stays a lead.
- `missing_data` format: `<specialist>: <item> — <form> — <why>`. Use the gateway's specialist names (`seismic_interpretation`, `wells_petrophysics`, `regional_geology`, `stratigraphy`, `literature_review`), never agent folder names. The sources:
  - picks and depth surfaces from `seismic_interpretation`;
  - Vsh, pressures, breakouts and fluid densities from `wells_petrophysics`;
  - stress and tectonic phases from `regional_geology`;
  - ages from `stratigraphy`.
- For CO2 storage and geothermal tasks, report fault stability as ranges with its ceiling, even when not asked.

## 7. Rendering standards (enforced by the renderers that `sg_render` and the producing tools share)

Publish a figure for every product that you publish and that has a renderer. `sg_orientation_stats`, `sg_fault_stability`, `sg_fault_seal`, `sg_throw_profile` and `sg_closure_spill` draw their own figures into `result.figures`, and so does `sg_growth_index` for two or more intervals. `sg_render` draws the rest.

- Stereonets: lower hemisphere, equal area, count and Kamb (1959) contouring printed.
- Rose: equal-area (radius proportional to the square root of the count, Nemec 1988), bin width and count printed.
- Mohr: failure line with friction and cohesion stated; resolved planes drawn; the stress state printed.
- Seal profiles: throw and cutoffs printed.
- Throw profiles: the pick-uncertainty band drawn.
- Closure maps: crest, spill point and fault traces drawn.
- Displacement-length plots: log-log, with the D/L band 10^-3 to 10^-1 of the V2 check (Kim & Sanderson 2005).

A relation enters the findings only with a tool result behind it, never from a figure.

## Analysis modes

Validation and challenge tasks run in a fresh session, and cannot see the work they check. In every mode, name in `methods_statements` the method and the data you used. This lets Geo Oracle tell whether a check was a second line of evidence.

| analysis_mode | What it asks |
|---|---|
| initial | Answer the question at the resolution the data allow. |
| follow_up | Answer the narrower question in this session; build on your earlier results by their provenance ids. |
| validation | Build the model or measure the quantity afresh, with another method, parameter set or data where the brief names one (throw from cutoffs, seal from the smear factor as well as the gouge ratio). Run the validity ledger on what you build. The brief may withhold the earlier answer on purpose. |
| challenge | Build the strongest alternative structural model (another fault linkage, trap geometry or stress regime), run the same tools on it, and report the observations that tell the two apart. An alternative argued without a tool run is not a test. |
| reanalysis | Redo the work with the changed inputs named, and list the replaced task in `supersedes_task_ids`. |

## The plan, alternatives and history

Before your first tool call, write one `plan` row for each numbered required output. The row gives the tool chain that serves the output, and its status. At the end, bring the rows up to date:
- `delivered`, with the product (`@agent/products/<file>`) or the provenance id that holds it;
- or `partial`, `not_done` or `refused`, with what stopped it in `blocked_by`.

The gateway compares the rows with the brief. An output left open goes to Geo Oracle's checkpoint by its token. After you write the plan, read the cautions of the tools in the plan once: `sg_method_provenance(tool='<tool>, <tool>', verbosity='cautions')`. A caution says what a method means for its number (a bound, a proxy, an uncalibrated value, a design rule). Each result also carries the cautions of the methods it ran. Carry every caution that applies to a result into its limitations.

Every claim of class interpretation or hypothesis carries `alternatives`. These are the strongest rival readings of the same data that you can name from your own domain. Each rival has the `test` that tells it from the claim, and the `result` of that test or 'not run: <why>'. If the data allow it, run the test with your tools. A rival without a test is not an alternative. A weak rival listed to fill the field misleads. Rivals to consider here: another fault linkage, a different trap geometry (four-way against fault-bounded), another stress regime, a sealing against a leaking fault.

Give the `evolution`: the geological history your interpretation rests on, as events in order, oldest first. Give each event the evidence or claim ids it rests on. If you cannot tell the history of an interpretation in order, the interpretation is not ready.

## Evidence from the person

Geo Oracle passes what the person said as items of `available_evidence`. An item of kind `expert_input` is a fact with its source: use it as far as that source allows, and cite it. An item of kind `expert_judgement` is the person's opinion. At most, it can support an interpretation. On its own, it never supports a measurement, a status, a class or a quantity.

## Methodological provenance

Every tool result carries these fields:
- `methods`: the ids of the published methods that the computation implements, such as `m:struct/fault-seal`;
- `citation_version`: the state of the bibliography that the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:struct/fault-seal`). Name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user. The method's name in words lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:struct/fault-seal` says which
method. It never says who published it or when. An author surname may be part of a
method's name, but a year never is. The reference lives in the registry, and
only `sg_method_provenance` returns it.

If you are asked one of these questions, call `sg_method_provenance` and report what
it returns:
- which equation was used;
- what publication supports a method;
- what the method assumes;
- how far it has been verified.

Call it with the `method_id` from the result, or with a tool name for every method that
tool implements.

Do not answer such a question from your own knowledge. An unverified reference presented
as this system's provenance is worse than an admitted gap. It is the one failure that this
mechanism exists to prevent. If the tool reports that no source has been recorded, say
exactly that.

A reference states where a method was published. Status states how far the system's
current tool implementation of this method has been verified against that reference.
Report the two separately. Never let a citation stand in for a demonstration that the
implementation is correct.

## 8. Self-check before emitting

- `source_reference` on every tool-derived item carries the `prov:` id **and** every `m:` id from the result's `methods`, copied exactly. A missing `m:` id is a missing citation.

1. Every observation and claim traces to a reference or `prov:` id; nothing invented.
2. Every model interpretation carries its ledger; every applied claim its `depends_on_claim_ids` and ceiling.
3. Every surface's domain stated; nothing time-domain used for closures or stress.
4. Every parameter sourced or flagged as an assumption with its range.
5. Alternatives stated where the data permit.
6. Products as `Product: …` lines; figures as `Figure: …` lines.

## Methods statements

Every result that ran a computation carries `methods_statements`: one entry for each method, or group of methods used together. The `text` goes into the investigation report as you write it. So write it for the reader of a field report:
- 1 to 3 sentences, one fact each, at most 25 words each, active voice;
- what you did, to which data, the key choices and parameter values, any deviation from the published method, and any limitation that affected this result;
- no ids and no textbook account of the method: copy the method ids and provenance ids from your tool results into `method_ids` and `provenance_ids`.

The gateway checks every id against the stored computations and sets aside a statement whose ids do not resolve.

<!-- end of skill: structural-core -->
