# Skills: The Agents' Operating Procedures

**88 procedure documents in 11 packs: one pack per agent.**

A skill is not code and nothing here executes. Each skill is a markdown
document the agent *reads* mid-task, telling it what to do, in what order,
with which tool, what it may not assume, and in what form to report. The
system prompt makes an agent competent in general; its skills make it
competent at a particular job, and they carry the rules that would otherwise
have to be repeated in every prompt.

The division of labour across the system is:

| Layer | Holds | Lives in |
| --- | --- | --- |
| System prompt | who the agent is, its ceilings, which skill to load when | `../system_instructions/` |
| **Skill** | **how to do one kind of work, and what to report** | **`agents/skills/` (here)** |
| Tool | the deterministic computation itself, with provenance | `src/geo_oracle/` |
| Gateway | routing, jobs, gates, result parsing | `geological_mcp.py` |

A skill never computes a number. It says which tool computes it, how to
parameterise it, how to read what comes back, and what may legitimately be
concluded from it.

---

## Schematic

Every skill is a directory containing exactly one `SKILL.md`. There are no
other file types anywhere in this tree.

```
agents/skills/
   |______ README.md                             this file
   |
   |______ geo-oracle-skills/                    THE ORCHESTRATOR (8)
   |          |______ geo-investigation-planning/SKILL.md
   |          |______ geo-evidence-inventory/SKILL.md
   |          |______ geo-specialist-tasking/SKILL.md
   |          |______ geo-result-review/SKILL.md
   |          |______ geo-investigation-ledger/SKILL.md
   |          |______ geo-play-prospect-evaluation/SKILL.md
   |          |______ geo-synthesis-audit/SKILL.md
   |          |______ geo-human-on-the-loop/SKILL.md
   |
   |______ literature-review-agent-skills/       THE TEN SPECIALISTS (7)
   |______ regional-geology-agent-skills/        (9)
   |______ seismic-interpreter-agent-skills/     (7)
   |______ structural-geology-agent-skills/      (8)
   |______ stratigraphy-geology-agent-skills/    (8)
   |______ sedimentology-agent-skills/           (6)
   |______ petrophysics-agent-skills/            (13)
   |______ subsurface-play-agent-skills/         (8)
   |______ prospect-target-agent-skills/         (7)
   |______ risk-uncertainty-agent-skills/        (7)
              |
              |______ <skill-name>/
                         |______ SKILL.md        name + description, then the procedure
```

Pack names match the saved TrueForge agent names with `-skills` appended, and
every skill inside a pack is prefixed with that agent's short tag (`seismic-`,
`petro-`, `strat-`, `sg-`…). Nothing is shared between packs: an agent sees
only its own pack, which is why the same rule sometimes appears in two packs
in two different wordings.

---

## Anatomy of a `SKILL.md`

```markdown
---
name: seismic-core
description: Core operating procedure for every seismic interpretation task -
  task reading, tool-result handling, provenance citation, evidence
  classification, claim status, missing-data format and the findings JSON.
  Load at the start of every task.
---

# Seismic core procedure

## 1. Read the task before touching a tool
...
```

Exactly two frontmatter keys, and all 88 files carry both:

- **`name`** — the skill's identifier. The system prompt refers to a skill by
  this name, so it must match the directory name.
- **`description`** — **the trigger.** It is the only part the agent sees
  before deciding whether to load the document, so it is written as *when to
  load this*, not as a summary. Several end with a literal instruction —
  "Load at the start of every task", "Load before `pt_render_closure_map`".

The body is ordinary markdown: numbered procedures, decision tables,
vocabularies, worked examples of good and bad statements. Tables carry most of
the weight, because most of what a skill enforces is a mapping — this
`analysis_mode` asks for that; this evidence kind gets that classification;
this failure means that status.

---

## The four roles a skill plays

Within every pack the names follow a fixed pattern, so an agent can tell from
the name alone whether it needs a document.

| Role | Count | What it holds |
| --- | --- | --- |
| `<tag>-core` | 9 | The pack's operating procedure. **Always loaded first.** Task reading, how to handle any tool result, evidence classification, claim status, the `missing_data` format, what must appear in `limitations`, and the shape of the findings to return. |
| `<tag>-data-qc` | 8 | Getting data in and judging whether it can bear the question: uploads, loading, units and mnemonics, depth rules, reading upstream products and their sidecars. |
| `<tag>-visualisation` | 7 | What the rendering tools enforce, the describe-then-interpret discipline, and — importantly — **what a figure may and may not be used to support.** |
| Domain skills | 64 | One per body of method: attributes, saturation, fault seal, sequence stratigraphy, flip tests, and so on. |

The `-core` / `-data-qc` / `-visualisation` triad is deliberate: it separates
*how to behave*, *whether the data supports the question*, and *what a picture
proves* from the geology itself.

The triad is not universal, and the gaps are informative:

- **No `-core`** in `geo-oracle-skills` or `literature-review-agent-skills`.
  The orchestrator's procedure is spread across planning, tasking and review
  rather than concentrated in one document; the literature specialist's sits
  in `lit-search-protocol` and `lit-evidence-extraction`.
- **No `-data-qc`** in those same two packs, nor in
  `regional-geology-agent-skills`, where data admission is part of
  `regional-data`.
- **No `-visualisation`** in `petrophysics-agent-skills` or
  `structural-geology-agent-skills` — both have figure tools, so the
  rendering rules live inside the relevant domain skills instead. In
  `risk-uncertainty-agent-skills` the role is combined into
  `risk-planning-and-visualisation`.

Two names look like the triad but are domain skills: `petro-core-scal` (core
*samples*, not core *procedure*) and `sed-core-description` (the description
vocabulary for drill core).

---

## How the packs integrate into the agentic system

### 1. The system prompt names the skill, the agent loads it

Each `agents/system_instructions/<agent>_system_prompt.md` ends with a
`## Skills` table of every skill in its pack and when to read it. The seismic
interpreter's begins:

| Skill | Read it when |
| --- | --- |
| `seismic-core` | At the start of every task |
| `seismic-data-qc` | Before ingesting data or interpreting amplitudes, frequencies or thin beds |
| `seismic-attributes` | Before any seismic_*_attribute call |

So the prompt holds the index and the skills hold the content. Adding a skill
without adding a row means it will rarely be read, and **renaming a skill
without editing the table sends the agent after a document that does not
exist** — it spends an iteration failing to find it. Nothing checks this: the
table is prose to everything but the model reading it.

### 2. The pack is mounted in the agent's TrueForge sandbox

The agent reads its skills from its own sandbox — the same system prompt says
plainly that *"your sandbox is only for reading your skills and seeing which
files were uploaded"*, and forbids computing there, because anything computed
outside the tools has no provenance.

**This folder is not uploaded to TrueForge. It is cloned from GitHub.** That is
why the packs are published ahead of the rest of the project, and it is the
most surprising thing about them. TrueForge stores a skill as a *git pointer*,
not as text:

```json
{"type": "git", "name": "seismic-core",
 "url":  "https://github.com/ARCHIBONG1/Geo-Oracle",
 "path": "seismic-interpreter-agent-skills/seismic-core",
 "ref":  "main",
 "description": "..."}
```

Three consequences worth holding onto:

- **An edited `SKILL.md` changes nothing until it is committed and pushed to
  `ref`.** The agent reads `main` on GitHub, never your working copy. A local
  edit you have not pushed is invisible to every agent.
- **`ref` is `main`, so it floats.** A push changes how the agents reason, with
  no deployment step and nothing recorded. That is convenient while the skills
  are still being written, and it is a deliberate choice rather than an
  oversight — `ref` also accepts a tag or a commit SHA if a release ever needs
  pinning.
- **The stored `description` is a separate copy** of the frontmatter, used for
  listing. It does not update when the clone does, so it can go stale while the
  body is current.

`ops/agents.toml [skills]` declares the `url`, the `ref`, and a `root` prefix,
with `[skills.packs]` listing the directory names; `ops/provision_trueforge.py`
registers them with `PUT /api/v1/settings/skills` and refreshes each
description from the local frontmatter. Because the registered path is
`<root>/<pack>/<name>`, moving these packs inside the repository — which is
what will happen when the whole project becomes one repository — is a one-line
change to `root` followed by `--apply`, not 88 edits. The local layout here
never changes.

### 3. Loading costs budget, which is why the split exists

Every skill read is one iteration against the agent's iteration limit, and a
run that hits the limit returns nothing to the orchestrator. That is the whole
reason the packs are split into many small documents rather than one long one:
an agent answering a porosity question should pay for `petro-core` and
`petro-porosity`, not for saturation-height and rock physics as well.

### 4. Skills name the tools in `src/geo_oracle/`

A skill is written against a specific tool surface — `seismic_describe_section`,
`petro_vshale`, `pt_render_closure_map`, `seismic_get_record`. The two must
move together: **renaming a tool in `src/geo_oracle/tools/tool_suites/` silently invalidates
every skill that names it**, and the agent will go looking for something that
is not in its tool list. Each skill also carries a "Not available" note where
no tool exists for an analysis, so the agent records the gap in `limitations`
instead of improvising.

### 4b. Some skills are written against tools this repo does not contain

Not every tool a skill names lives in `src/geo_oracle/`. These are verified and
working, but no test here can check them:

- **`literature-review-agent-skills`** is written against the **OpenAlex** MCP
  server (`search_works`, `get_work`, `resolve_references`, `list_citations`,
  `search_entities`, `get_entity`, `find_candidate_works`), **Exa** and
  **Tavily**. `lit-search-protocol` and `lit-source-verification` are largely
  procedures for using them correctly — checking the echoed `oql`, matching
  authorship, resolving every hit before citing it.
- **`geo-oracle-skills`** relies on two TrueForge built-ins:
  `get_current_datetime` (the turn clock behind
  `geo-investigation-planning`'s time budget) and `ask_user_question` (the
  only channel to the person, which every gate in `geo-human-on-the-loop`
  depends on).

The rule of thumb matches the one in `src/geo_oracle/README.md`: a **number** must
come from a tool in `src/geo_oracle/`, which carries provenance; a **citation** must be
resolved through the external index. A skill should never have an agent
compute a value with a retrieval tool, or cite a source it only found.

### 5. The `-core` skills define the contract the gateway parses

This is where the loop closes. `seismic-core` §3–§5 fix the evidence
classifications (`observation`, `measurement`, `interpretation`, `inference`,
`hypothesis`), require every interpretation to list its
`supporting_evidence_ids`, `assumptions` and `limitations`, set the claim
status rules, and fix the `missing_data` format. Those are **exactly** the
fields `SpecialistFindings` validates in `geological_mcp.py`, and
`parse_findings()` is what turns the agent's reply into them.

```
skill tells the agent what to emit
        │
        ▼
specialist returns one JSON object
        │
        ▼
geological_mcp.parse_findings()  ──▶  SpecialistFindings  ──▶  globalize_ids()
                                                                    │
                                              ids namespaced <task_id>/E1 …
                                                                    ▼
                                                        Geo Oracle reasons on it
```

A skill that drifts from the model does not raise an error. The reply simply
fails to parse, and the gateway returns the text in `raw_output` with a
warning — the content survives, but it is no longer structured evidence. **So
the `-core` skills and the output models in `geological_mcp.py` must be
changed together.**

### 6. `geo-oracle-skills` is a different kind of pack

The ten specialist packs govern *doing* geology. The orchestrator's pack
governs *running an investigation*, and each of its skills maps onto gateway
machinery rather than onto a compute tool:

| Skill | Governs |
| --- | --- |
| `geo-investigation-planning` | turning an objective into testable questions, specialist tasks and a dependency graph |
| `geo-evidence-inventory` | characterising available evidence and packaging it for specialists that cannot open files |
| `geo-specialist-tasking` | the brief templates for the ten specialist tools and the three job tools — field by field, with the common mistakes |
| `geo-result-review` | the checklist for judging a specialist result before relying on it |
| `geo-investigation-ledger` | the format of `investigation/ledger.md`, the running record the synthesis is written from |
| `geo-play-prospect-evaluation` | gates, element checklists, support labels and maturity rules for the play / prospect / risk stages |
| `geo-synthesis-audit` | structure, citation rules and the mandatory scientific audit of any synthesis |
| `geo-human-on-the-loop` | the activation question, the five-part checkpoint, accept / refuse / edit, and the work gate |

`geo-human-on-the-loop` is the counterpart of `hotl.py`: the gateway *enforces*
those gates by refusing calls, and this skill tells Geo Oracle how to satisfy
them — what to ask the user, and what to put in a checkpoint record so it
passes the checks against the job registry.

---

## What each pack contains

### `geo-oracle-skills` (8) — the orchestrator
See the table above.

### `literature-review-agent-skills` (7)
| Skill | Covers |
| --- | --- |
| `lit-search-protocol` | search planning: term expansion across formation, historical and stratigraphic names |
| `lit-inputs-library` | librarian of the shared inputs folder — complete listings, confirming what is actually there |
| `lit-source-appraisal` | authority, evidence basis, geographic relevance and independence of a source |
| `lit-source-verification` | resolving works to DOIs or OpenAlex IDs; every citation verified |
| `lit-evidence-extraction` | mapping extracted geological evidence onto the gateway's JSON output |
| `lit-geoscience-question-bank` | question sets and evidence checklists for decomposing a request |
| `lit-synthesis-report` | synthesis across sources, conflicting literature, careful consensus language |

### `regional-geology-agent-skills` (9)
| Skill | Covers |
| --- | --- |
| `regional-core` | task procedure, scope tags, the transfer checklist, the memo |
| `regional-data` | areas of interest, the geodata snapshot catalogue, supplied GIS files, grid sampling |
| `regional-stratigraphy` | time-scale discipline, chronostratigraphic charts, hiatuses and overlaps |
| `regional-structure-stress` | fault elements and trends, SHmax orientation, stress regime, domain boundaries |
| `regional-thermal` | heat-flow statistics, conductive geotherms, calibration to corrected well temperatures |
| `regional-basin` | backstripping and decompaction, McKenzie stretching fits, burial history |
| `regional-potential-fields` | gravity and magnetic transforms, lineaments as observations, depth-to-source |
| `regional-seismicity` | catalogue summaries, magnitude of completeness, b-values and their limits |
| `regional-visualisation` | maps, rose diagrams, profiles; describe-then-interpret |

### `seismic-interpreter-agent-skills` (7)
| Skill | Covers |
| --- | --- |
| `seismic-core` | the operating procedure and the findings contract |
| `seismic-data-qc` | uploads, SEG-Y header scanning, ingest decisions, resolution |
| `seismic-attributes` | choosing, parameterising, running and reading the attribute tools |
| `seismic-structural` | horizon tracking, faults and throw, structure and isochron maps, closure and spill |
| `seismic-stratigraphic` | termination detection, key surfaces, seismic facies, stratal slices |
| `seismic-qi` | when amplitudes can be trusted; wavelet, synthetic and well tie, AVO |
| `seismic-visualisation` | how the agent "sees" seismic — reading `seismic_describe_section` as its only view |

### `petrophysics-agent-skills` (13)
| Skill | Covers |
| --- | --- |
| `petro-core` | task procedure, analysis mode, domain emphasis |
| `petro-data-qc` | imports, LAS/CSV loading, mnemonics, units and unit conflicts, headers |
| `petro-shale-volume` | VSH method choice, endpoints, comparing at least two methods |
| `petro-porosity` | density / neutron-density / sonic by lithology, fluid and hole condition |
| `petro-saturation` | the Rw hierarchy, temperature correction, Archie versus shaly-sand |
| `petro-saturation-height` | Leverett J and Brooks-Corey fits, lab-to-reservoir conversion |
| `petro-permeability` | core transforms with 10th–90th percentile bands, flow units, Timur |
| `petro-net-pay` | cutoffs, integrated evaluation with bad-hole exclusion, zone summaries |
| `petro-lithology` | matrix identification, multimineral solves with a reconstruction check |
| `petro-core-scal` | core-to-log depth shift, overburden correction, log-versus-core statistics |
| `petro-rock-physics` | fluid properties incl. CO2, Vs measured or predicted, Gassmann substitution |
| `petro-pressure-temperature` | pretest QC, gradient legs, contacts from gradient intersections |
| `petro-time-depth` | checkshots, datums, offset correction, sonic integration with drift |

### `structural-geology-agent-skills` (8)
| Skill | Covers |
| --- | --- |
| `structural-core` | task procedure, the never-invent rule, the validity ledger |
| `structural-data-qc` | upstream products and sidecars — domain, pick uncertainty, scope, validity |
| `structural-geometry` | orientation statistics, mean planes and sets, fold axes by the pi method |
| `structural-displacement` | throw profiles, displacement-length against the global population, growth |
| `structural-networks` | fault-network topology (I, Y, X nodes), connectivity, relative ages |
| `structural-modelling` | fault planes and horizons per block, validity tests, seeded realizations |
| `structural-geomechanics` | the stress tensor, slip and dilation tendency, critical pore pressure |
| `structural-fault-seal` | juxtaposition, shale gouge ratio, shale smear factor, column heights |

### `stratigraphy-geology-agent-skills` (8)
| Skill | Covers |
| --- | --- |
| `strat-core` | the two ledgers, the tie-point hierarchy, the consistency ledger |
| `strat-data-qc` | correlation logs and well tops, sidecars, the depth rule for deviated wells |
| `strat-log-correlation` | ranking tie points, dynamic time warping constrained to anchors |
| `strat-sequence` | stacking patterns from GR, candidate flooding surfaces and sequence boundaries |
| `strat-chronostratigraphy` | age models from dated samples and datums, the seeded envelope, missing time |
| `strat-biostratigraphy` | biostratigraphic events as tie points, event ordering, graphic correlation |
| `strat-unconformities-thickness` | isopachs, eroded thickness, erosion versus fault cut-out |
| `strat-visualisation` | correlation panels, the two-datum check, what a panel may support |

### `sedimentology-agent-skills` (6)
| Skill | Covers |
| --- | --- |
| `sed-core` | the interpretation ladder, diagnostic before suggestive |
| `sed-core-description` | the fixed vocabulary — Wentworth classes, structure and contact codes, bioturbation index |
| `sed-data-qc` | registering descriptions, analyses, analogue tables and photo indexes; the depth rule |
| `sed-facies-analysis` | transitions, the embedded Markov test, Walther's law, the key-surface exclusion |
| `sed-depositional-systems` | the diagnostic matrix per environment; how an association becomes an environment |
| `sed-visualisation` | graphic logs and transition diagrams; what a figure may support |

### `subsurface-play-agent-skills` (8)
| Skill | Covers |
| --- | --- |
| `play-core` | the ordinal status scale, the scope rule, the weakest link, dependencies |
| `play-data-qc` | upstream products and claims, their ledgers, scope and transfer |
| `play-petroleum-systems` | hydrocarbon elements and processes, the events chart, the critical moment |
| `play-co2-storage` | CO2 elements with containment kept separate; the pressure limit; trapping |
| `play-geothermal` | convection- and conduction-dominated types; heat, permeability, fluid, recharge |
| `play-hydrogen` | natural hydrogen and underground storage; why timing differs |
| `play-well-outcomes` | every well as a success, a failure on a named element, or undetermined |
| `play-visualisation` | status matrices, events charts, dependency graphs; discrete classes |

### `prospect-target-agent-skills` (7)
| Skill | Covers |
| --- | --- |
| `prospect-core` | the maturity ladder, readiness tests R1–R7, the refusal rules |
| `prospect-data-qc` | closures, play matrices, ledgers and companion tables; registering constraints |
| `prospect-definition` | closures into candidates, what each readiness failure means, the maturation plan |
| `prospect-scenarios` | trap, fill and contact alternatives; naming what would separate them |
| `prospect-volumetrics` | gross rock volume from a closure's cells; sourced inputs |
| `prospect-target` | the well target under licence, keep-out and existing-well constraints |
| `prospect-visualisation` | closure maps and maturity tables; what a figure may support |

### `risk-uncertainty-agent-skills` (7)
| Skill | Covers |
| --- | --- |
| `risk-core` | no scores; materiality by flip test with its provenance id; nothing new |
| `risk-data-qc` | importing Geo Oracle's ledger and reading it as data; product sidecars |
| `risk-register` | collecting every uncertainty from the ledger, findings and sidecars; typing it |
| `risk-materiality` | the four flip tests — play status, maturity, threshold straddle, scenario comparison |
| `risk-failure-modes` | per-domain failure modes from a fixed library: mechanism, elements, detection |
| `risk-audit` | contradictions carried with both sides; load-bearing assumptions |
| `risk-planning-and-visualisation` | the reduction plan — actions, owners, modes, neutral framing |

---

## Working on skills

**Adding one.** Four steps, and the skill does not exist for an agent until all
four are done:

1. Create `<pack>/<tag>-<subject>/SKILL.md` with `name` matching the directory
   and a `description` written as *when to load this*. If it governs a tool,
   name that tool explicitly.
2. Add a row to the `## Skills` table in
   `agents/system_instructions/<agent>_system_prompt.md`, or it will not be read.
3. Add the directory name to its pack in `ops/agents.toml` `[skills.packs]`,
   then `python ops/provision_trueforge.py --apply` to register and attach it.
   The script refuses to run while the TOML and this folder disagree, so a
   missed entry is loud rather than silent.
4. **Commit and push.** TrueForge clones the body from `main` (see §2), so
   until the push the skill is registered but empty.

**Changing one.** Four couplings to check:

1. **Tool names.** A skill that names a tool no longer in the suite sends the
   agent after something that is not there.
2. **The findings contract.** Anything a `-core` skill says about evidence
   classes, claim status, `scientific_status` or `missing_data` must match the
   models in `geological_mcp.py`. They are not validated against each other.
3. **The `## Skills` table** in the system prompt. Nothing checks it.
4. **The pack listing** in `ops/agents.toml`. The provisioner checks this one.

Renaming a skill touches 1, 3 and 4 at once, which is why it is worth doing
deliberately. Editing the body touches none of them — but still needs a push.

**Deleting one.** Remove the table row first; a prompt that names a missing
skill makes the agent spend an iteration failing to find it. Then remove it
from `[skills.packs]` and re-apply. The provisioner never deletes a
registration, so the orphan stays in TrueForge Settings as a `NOTE` until it is
removed there by hand — harmless unless an agent attaches it.

**Conventions that hold throughout.** Describe before interpreting. Never
upgrade a classification, only downgrade it. Cite the provenance id of every
number. Record a capability gap in `limitations` rather than improvising
around it. State what a figure may *not* be used to support. None of these are
enforced by software — they hold because every `-core` skill repeats them.
