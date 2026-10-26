---
name: geo-investigation-planning
description: How Geo Oracle turns an exploration objective into testable geological questions, specialist tasks and a dependency graph. It also covers how Geo Oracle runs them in parallel waves, budgets the turn's time, judges whether another specialist call is worth making, and decides when to stop. Use when you start an investigation; when the user changes the objective or supplies major new data; when a result changes the model enough to re-plan; when you choose the next specialist call; and when you decide whether the investigation is finished.
---

# Investigation planning

A good plan asks the fewest specialist questions that can change the answer. It runs them in an order where each one receives the evidence it needs.

## 1. Frame the objective

Write down these items. Ask the user for anything that is unclear and material:

- the exploration objective, and the decision it should support;
- the resource type: hydrocarbon, CO₂ storage, geothermal, natural hydrogen or hydrogen storage. This sets the play elements (see `geo-play-prospect-evaluation`). Play and prospect evaluation covers only these five. The other specialists can still investigate another resource type;
- the area, target intervals and scale;
- constraints: available data, deadlines, and the depth of analysis wanted (screening or detailed);
- what "done" looks like, for example "a ranked list of supported play concepts with the evidence gaps for each".

## 2. Write testable questions

Each question must be answerable from evidence. It must name what would count as an answer.

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

If you call every specialist "for completeness", you add cost and noise, and create an illusion of corroboration.

## 4. Map dependencies

The following are typical constraint flows, not a fixed order. Use them to spot prerequisites.

- `wells_petrophysics` → `stratigraphy`: correlation logs, reported tops, facies and net reservoir.
- `stratigraphy` → `seismic_interpretation`: well ties and horizon identity.
- `seismic_interpretation` → `structural_geology`: fault and horizon geometry (depth products).
- `regional_geology` → `structural_geology`: the stress field as a regional prior.
- `sedimentology` → reservoir distribution → `subsurface_play`.
- `regional_geology` and `literature_review` → context and analogues for every discipline. These are context, not local evidence.
- Integrated model → `subsurface_play` → `prospect_target` → `risk_uncertainty`.
- `risk_uncertainty` → the reduction wave → `risk_uncertainty` again in `follow_up` mode (section 4a).

Mark iterative loops explicitly. For example, stratigraphy and seismic often need an initial pass each. Then they need a reconciliation once the horizon interpretation exists.

## 4a. The risk gate and the reduction loop

Risk runs after the play and prospect waves. The investigation does not end at its first answer:

1. **Gate.** Task `risk_uncertainty` with:
   - the ledger's absolute sandbox path, as kind `investigation_ledger`: the path you gave `set_hotl_mode` as `ledger_path`. Do not build it again from `pwd`. The gateway adds the path on record when the brief has none, and corrects a path that differs;
   - every finding relied on, as `upstream_findings` with ids, statuses, scope tags and `depends_on`;
   - every product, by reference;
   - the objectives in scope, and the decision context.

   It returns the register, the materiality report and a `reduction_plan`. The plan splits its items into `actionable_now` and `needs_acquisition`, each with an owner, an `analysis_mode` and a `task_framing`. The plan also lists:
   - `irreducible` entries, with the reason;
   - `settled_failures`: conclusions decided against in every case, which no reduction can change. They are not sent for reduction. The only checks on them are their threshold's source and the basis of their inputs.
2. **Reduction wave.** Run every `actionable_now` item as one wave. Send each item as a task to the named owner, in the named mode (`challenge` or `validation`, never `initial`). Use the item's `task_framing` as the geological question, unchanged. It is framed as a test. The one thing that would make the loop harmful is to reframe it as a confirmation. Record each task in the ledger as an attempt: entry, task id, owner, mode, what it tries, why. If a human-on-the-loop setting is on, present the plan first, and run only what the human approves. If it is off, run the wave automatically. The `needs_acquisition` items are not tasks. They go to the user in the final answer.
3. **Follow-up.** Task `risk_uncertainty` again in `follow_up` mode, with the new products, the prior materiality file and every attempt. It reruns the same flip tests and returns `reduction_outcomes`. For each attempt, these give what was tried, why, and what happened to the uncertainty: settled, settled as a failure (every case now fails), narrowed, unchanged, newly material, overturned (a settled failure no longer fails), newly a settled failure, changed, or entry withdrawn (the entry is gone from the current register).
4. **One wave by default.** Run a second reduction wave only when the user asks. Never reissue an attempt already recorded in the ledger. Report an unchanged outcome; do not retry it.
5. **Report every attempt.** The final answer says what was tried, why, and what happened, whichever way it went. A reduction that did not settle its uncertainty is one of the more useful things the investigation can report. The acquisition items are its recommendations.

## 4b. Human on the loop

Before the first dispatch of a plan with more than one specialist, ask the activation question and record the answer. See `geo-human-on-the-loop`: it also holds the checkpoint summary, the accept / refuse / edit rules and the work gate. The gateway:
- allows one specialist call before the mode is set, and refuses every call after that;
- refuses a wave whose checkpoint is unfiled;
- refuses a specialist the person struck from the plan;
- stops at the work ceiling.

Those refusals name what to do. The work already done is never lost.

## 4c. Record at activation: constraints, the decision chain, the reserve

Pass these to `set_hotl_mode` with the person's answer:

- `constraints`: what the person stated that every specialist must respect, each as a short quote of their words: the data they named, datums and units, thresholds, exclusions, the decision. The gateway puts them in every brief under the person's name, so you do not repeat them in briefs. If the person stated none, leave it empty. Never fill it with your own reading of a vague request.
- `decision_chain`: the results the decision depends on, in order. For a drilling decision: the trap mapped in depth, then the volume against the threshold, then the well location. Spend runs on this chain first.
- `reserve_runs`: runs held back for reruns after an upstream result changes (two or three for most investigations). The gateway warns when other work starts to spend them.

## 5. Plan waves

- **Wave 1:** tasks whose inputs exist now. These are often literature, regional context, well-data QC and seismic data characterisation.
- **Later waves:** tasks that need earlier outputs.
- **Parallelism:** Geo Oracle makes one tool call per step. So start the independent tasks of a wave one after another, with `wait_seconds: 0`. Each call returns its job id at once, and the specialists run in parallel. Then collect them one by one.

Write the plan into the ledger's *Tasks* table before you run it. For a large investigation, first show the user the plan in a few lines.

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

Before any additional call, write one line in the ledger with:
- the question the call resolves;
- which interpretation or decision it could change;
- why the existing evidence cannot answer it.

If you cannot write that line, do not make the call.

Good reasons for another call:
- a link of the decision chain (section 4c) that is not yet delivered;
- to discriminate between competing interpretations;
- to validate a load-bearing claim with independent evidence;
- to challenge an interpretation that plays or prospects depend on;
- to re-run after an upstream change;
- to recover from a QC failure.

Poor reasons:
- completeness;
- "every specialist should weigh in";
- to ask again for the same evidence, in the hope of more confidence;
- to polish a result that is already adequate for the decision.

## 7. The turn budget: time and iterations

A TrueForge turn has two ceilings:
- **Time.** The platform's turn time limit is **three hours in this deployment** (`SERVER_EXECUTION_TIMEOUT_SECONDS=10800`; check it before you assume any other number). It cuts the turn where it stands, with no failure message: the sentence simply ends.
- **Iterations.** The iteration limit in the agent's runtime configuration (**500 for Geo Oracle here**) counts every model step. Each tool call, each poll of `get_specialist_result`, each skill read and each ledger update is one iteration.

If the ledger is current, a turn cut at either ceiling loses nothing, because the next turn resumes from it. A turn cut before the ledger was written loses the reasoning. So manage both budgets explicitly:

1. **At the start of every turn**, call `get_current_datetime`. Write the start time and an iteration count of 0 into the ledger's header.
2. **Estimate before you start.** A specialist call costs three to five minutes and two to four iterations (start, poll, collect, review). A wave of three parallel calls costs about the same time as one call, but three times the iterations. A skill read or a ledger update is one iteration. If the estimate exceeds either limit, plan the turn as the first part of the investigation, and say so to the user.
3. **Before every specialist call and every poll**, compute the remaining turn time from `get_current_datetime` and the start time in the ledger: `remaining = limit - elapsed`. Then:
   - `wait_seconds = min(cap, remaining - 120)`: a call may never run past the turn's end. If the wait would be under 60 s, it is not worth making: checkpoint instead.
   - Keep launching work until about ten minutes of the turn remain, or about 90 % of the iterations are spent. If you stop earlier, you waste the turn. A three-hour turn abandoned at forty minutes leaves two hours unused, and hands the person a resumption they did not ask for.
   - If the turn limit cuts a specialist call, the call returns a timeout error and the turn ends there. The job itself is unharmed and the next turn collects it, but the checkpoint was never written. The arithmetic above prevents that.
4. **Checkpoint before the cut:** update the ledger (running job ids, open questions, next steps). Then end the turn with a short summary drawn from the ledger, and the sentence "Continue, and I resume from the ledger." The next turn first reads the ledger and collects any jobs still running.
5. **Never start a specialist call you cannot collect.** A job started ten minutes before the cut will finish after it. Start it with `wait_seconds: 0` and record its job id, so the next turn collects it. Or leave it for the next turn. Specialist runs on several wells take 10 to 20 minutes, and waves run in parallel. So a three-hour turn holds ten to fourteen runs across several waves: most investigations fit in one turn.
6. **A running job is not a reason to end a turn.** While the clock allows, poll it with a real wait, and carry on. If you hand back because something is in flight, you waste the rest of the turn. You also give the person a checkpoint they did not ask for.

**The checkpoint is where a person joins.** With a human on the loop, the checkpoint summary is the point at which Geo Oracle asks them whether to continue, change direction or stop (`geo-human-on-the-loop`). `ask_user_question` exists for that moment and for material ambiguities in the objective, not for routine progress. Specialists never ask: their "ask user" capability is off, and a question from one stalls its job.

## 8. Checkpoints and retries

- **Checkpoints.** A human-on-the-loop checkpoint is never a reason to stop: ask it with `ask_user_question`, and carry on. When the turn's time limit is near (section 7), end the turn with:
  - the ledger updated;
  - the job ids of any running jobs;
  - a short summary of results so far;
  - the next steps.
- **Retries.** Retry an execution failure at most once, with a stated reason.
- **Diagnose, then change the approach.** If a result missed, send it back to the same session as a `follow_up` that asks why, before any rerun. From the third attempt at one result, the gateway refuses the task unless `reason_for_reanalysis`:
  - names what is different (another method, seed, input or scope);
  - and does not repeat an earlier attempt's reason.

  If nothing different is available, record the result as not delivered, or ask the person.
- **Repeated failure.** If a specialist fails repeatedly, record the analysis as not executed and continue with what can be done.
- **Replaced results.** Give every task the `upstream_task_ids` it builds on. When a result replaces another (the specialist's `supersedes_task_ids`, a `reanalysis`, or your judgement), the gateway lists the results built on the old one. At the next checkpoint, give each of those results a line in `record.reviewed`: `rerun` (planned in the next wave) or `kept`, with the reason it still stands. Declare in `record.superseded: [{task_id, by}]` any replacement that the specialist did not name. Reruns use `analysis_mode: reanalysis`, which may spend the reserve.

## 9. When to stop

Stop when all of the following are true:
1. The major questions have been addressed as far as the evidence allows.
2. The material specialist analyses are complete, or recorded as not executed.
3. Contradictions are resolved or explicitly preserved.
4. Material assumptions and uncertainties are recorded in the ledger.
5. Downstream conclusions have been re-checked after every upstream change.
6. If plays are relevant, `subsurface_play` has evaluated them. If prospects are justified, they are traceable to the model. Risk has been reviewed to the depth the decision needs.
7. Further calls are unlikely to change the interpretation, or they would need evidence that does not exist. Such evidence is absent from the listing (`list_shared_inputs`). A name in a specialist's `missing_data` is not enough: that field often describes a tool failure, or a brief that asked for less.
8. Every result the person asked for is delivered, is not deliverable for an input that does not exist, or is declined by the person. If a follow-up could still produce a requested result, the follow-up comes first. A result that came back in part (closures on one horizon but not the other) is `partial`. Its missing part gets the follow-up. The checkpoint that ends the work carries this as `deliverables` (see `geo-human-on-the-loop`).

The checkpoint that ends the work also records the final state. The gateway refuses that checkpoint without:
- `hypotheses`: every hypothesis of the ledger, including the ones that lost. Each has its status from the ledger's scale, and its basis (claim and evidence ids);
- `objective_check`: the answer to the objective in one sentence, its status (answered, partly answered, not answerable) and what it lacks;
- `pre_mortem`, when the risk specialist ran: up to three reasons the leading interpretation could be wrong. Each reason is covered by the test that addressed it, or by the open uncertainty that holds it. Ask the risk specialist for them in its follow-up.

Then read `geo-synthesis-audit` and produce the synthesis.

<!-- end of skill: geo-investigation-planning -->
