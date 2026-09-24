---
name: geo-investigation-planning
description: How Geo Oracle turns an exploration objective into testable geological questions, specialist tasks and a dependency graph, runs them in parallel waves, judges whether another specialist call is worth making, and decides when to stop. Use when starting an investigation, when the user changes the objective or supplies major new data, when a result changes the model enough to re-plan, when choosing the next specialist call, and when deciding whether the investigation is finished.
---

# Investigation planning

A good plan asks the fewest specialist questions that can change the answer, and runs them in an order where each one receives the evidence it needs.

## 1. Frame the objective

Write down the following items, and ask the user for anything that is unclear and material:

- the exploration objective, and the decision it should support;
- the resource type: hydrocarbons, CO₂ storage, geothermal, minerals or other. This sets the play elements (see `geo-play-prospect-evaluation`);
- the area, target intervals and scale;
- constraints: available data, deadlines, and the depth of analysis wanted (screening or detailed);
- what "done" looks like, for example "a ranked list of supported play concepts with the evidence gaps for each".

## 2. Write testable questions

Each question must be answerable from evidence and must name what would count as an answer.

| Weak | Testable |
|---|---|
| "Analyse the seismic." | "Is the high-amplitude package between H2 and H3 on inlines 1200–1350 consistent with a channelised sand body, and what alternative explanations fit the same response?" |
| "Look at the wells." | "Does the Main Sand in W1–W3 show net reservoir, and is its thickness trend consistent with a southward-thinning fan?" |
| "Find literature on the basin." | "What published source-rock evidence (TOC, HI, maturity) exists for the Lower Shale in this basin, and from which wells or outcrops?" |

For each question, record:
- the evidence that could answer it;
- the specialist who should answer it;
- the downstream question it feeds.

## 3. Involve only the disciplines needed

- A literature question needs `literature_review` and nothing else.
- A well-correlation question may need `wells_petrophysics` and `stratigraphy`, but not `prospect_target`.

Calling every specialist "for completeness" adds cost and noise, and creates an illusion of corroboration.

## 4. Map dependencies

The following are typical constraint flows, not a fixed order. Use them to spot prerequisites.

- `wells_petrophysics` → `stratigraphy`: log-based tops, facies and net reservoir.
- `stratigraphy` → `seismic_interpretation`: well ties and horizon identity.
- `seismic_interpretation` → `structural_geology`: fault and horizon geometry.
- `sedimentology` → reservoir distribution → `subsurface_play`.
- `regional_geology` and `literature_review` → context and analogues for every discipline. These are context, not local evidence.
- Integrated model → `subsurface_play` → `prospect_target` → `risk_uncertainty`.
- `risk_uncertainty` → back to the specialist who can reduce each critical uncertainty.

Mark iterative loops explicitly. For example, stratigraphy and seismic often need an initial pass each, then a reconciliation once the horizon interpretation exists.

## 5. Plan waves

- **Wave 1:** tasks whose inputs exist now. These are often literature, regional context, well-data QC and seismic data characterisation.
- **Later waves:** tasks that need earlier outputs.
- **Parallelism:** run the independent tasks of a wave in one step so they execute in parallel.
- **Result size:** about three full results fit in one step. For wider waves, start the jobs with `wait_seconds: 0` and collect them one by one.

Write the plan into the ledger's *Tasks* table before running it. For a large investigation, show the user the plan in a few lines first.

Example plan:

| Task | Tool | Question (short) | Depends on | Wave |
|---|---|---|---|---|
| T01-lit | literature_review | Published source-rock and reservoir evidence, Lower Shale / Main Sand | none | 1 |
| T02-wells | wells_petrophysics | QC and net reservoir in the Main Sand, W1–W3 | none | 1 |
| T03-strat | stratigraphy | Main Sand correlation W1–W3 | T02 | 2 |
| T04-seis | seismic_interpretation | H2/H3 mapping and amplitude character | T03 (well ties) | 2 |
| T05-struct | structural_geology | Closure geometry at H3, fault-seal considerations | T04 | 3 |
| T06-play | subsurface_play | Is the Main Sand structural play supported? | T01–T05 | 4 |

## 6. Is the next call worth it?

Before any additional call, write one line in the ledger containing:
- the question the call resolves;
- which interpretation or decision it could change;
- why the existing evidence cannot answer it.

If you cannot write that line, do not make the call.

Good reasons for another call:
- discriminating between competing interpretations;
- validating a load-bearing claim with independent evidence;
- challenging an interpretation that plays or prospects depend on;
- re-running after an upstream change;
- recovering from a QC failure.

Poor reasons:
- completeness;
- "every specialist should weigh in";
- asking again for the same evidence in the hope of more confidence;
- polishing a result that is already adequate for the decision.

## 7. Budget, time and checkpoints

- **Turn time.** A TrueForge turn has a time limit, one hour by default. Plan roughly how many waves fit.
- **Checkpoints.** At a natural checkpoint, or before the limit, stop with:
  - the ledger updated;
  - the job ids of any running jobs;
  - a short summary of results so far;
  - the next steps.
- **Retries.** Retry an execution failure at most once, with a stated reason.
- **Repeated failure.** If a specialist fails repeatedly, record the analysis as not executed and continue with what can be done.

## 8. When to stop

Stop when all of the following are true:
1. The major questions have been addressed as far as the evidence allows.
2. The material specialist analyses are complete, or recorded as not executed.
3. Contradictions are resolved or explicitly preserved.
4. Material assumptions and uncertainties are recorded in the ledger.
5. Downstream conclusions have been re-checked after every upstream change.
6. Plays (if relevant) have been evaluated by `subsurface_play`; prospects (if justified) are traceable to the model; risk has been reviewed to the depth the decision needs.
7. Further calls are unlikely to change the interpretation, or they would need evidence that does not exist.

Then read `geo-synthesis-audit` and produce the synthesis.
