# Skills: the agents' operating procedures

This folder holds 90 skills in 11 packs, one pack per agent. A skill is a
Markdown document that an agent reads during a task. It says what to do and in
what order, which tool to use, what may not be assumed, and in what form to
report. Nothing here executes.

The system prompt makes an agent competent in general; its skills make it
competent at a particular job, and they hold the rules that would otherwise have
to be repeated in every prompt. A skill never computes a number: it names the
tool that computes it, how to set its parameters, how to read what comes back,
and what may legitimately be concluded.

| Layer | Holds | Location |
| --- | --- | --- |
| System prompt | who the agent is, its limits, which skill to read when | `agents/system_instructions/` |
| Skill | how to do one kind of work, and what to report | `agents/skills/` (this folder) |
| Tool | the computation itself, with provenance | `src/geo_oracle/tools/` |
| Gateway | routing, jobs, supervision gates, parsing of replies | `src/geo_oracle/geological_mcp.py` |

## Layout

Every skill is a directory holding a single `SKILL.md`.

```
agents/skills/
   |______ README.md                             this file
   |
   |______ geo-oracle-skills/                    the orchestrator (8)
   |______ literature-review-agent-skills/       (7)
   |______ regional-geology-agent-skills/        (9)
   |______ seismic-interpreter-agent-skills/     (8)
   |______ structural-geology-agent-skills/      (8)
   |______ stratigraphy-geology-agent-skills/    (8)
   |______ sedimentology-agent-skills/           (7)
   |______ petrophysics-agent-skills/            (13)
   |______ subsurface-play-agent-skills/         (8)
   |______ prospect-target-agent-skills/         (7)
   |______ risk-uncertainty-agent-skills/        (7)
              |
              |______ <skill-name>/
                         |______ SKILL.md
```

A pack is named after its TrueForge agent with `-skills` appended, and each skill
in it starts with the agent's short tag (`seismic-`, `petro-`, `strat-`,
`structural-` and so on). An agent sees only its own pack, which is why the same
rule sometimes appears in two packs in different words.

## How a skill is written

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

<!-- end of skill: seismic-core -->
```

- **`name`** must match the directory name, because the system prompt refers to
  the skill by it.
- **`description`** is the only part an agent sees before deciding whether to read
  the skill, so it says *when* to read it rather than summarising it. Several end
  with an explicit instruction, such as "Load at the start of every task".
- **The body** is ordinary Markdown: numbered procedures, decision tables,
  vocabularies and worked examples of good and bad statements. Tables carry most
  of it, because most of what a skill enforces is a mapping: this analysis mode
  asks for that, this kind of evidence gets that classification, this failure
  means that status.
- **The last line** is `<!-- end of skill: <name> -->`. TrueForge replaces a tool
  result of more than about 6,000 tokens with a short preview, and a skill read
  that is cut off fails without any other error. An agent that does not see the
  end marker knows its read was incomplete, and the gateway does not count such
  a read as done. `tests/test_live_run_1019_fixes.py` checks that every skill
  ends with its marker. Keep each skill well under that size; the longest are
  about 3,500 words.

## The four kinds of skill

Within each pack the names follow a pattern, so an agent can tell from the name
alone whether it needs a document.

| Kind | Count | Holds |
| --- | --- | --- |
| `<tag>-core` | 9 | The pack's operating procedure, read first: how to read a task, handle a tool result, classify evidence, set claim status, report missing data, and shape the findings. |
| `<tag>-data-qc` | 8 | Getting data in and judging whether it can bear the question: uploads, units, depth conventions, reading other agents' products. |
| `<tag>-visualisation` | 7 | What the figure tools enforce, describing before interpreting, and what a figure may and may not support. |
| Domain skills | 66 | One per body of method: attributes, saturation, fault seal, sequence stratigraphy, flip tests and so on. |

Not every pack has all three of the first kinds. Geo Oracle's procedure is spread
across planning, tasking and review, and the literature specialist's sits in
`lit-search-protocol` and `lit-evidence-extraction`, so neither has a `-core`
skill. Data admission is part of `regional-data` for the regional specialist.
Petrophysics and structural geology keep their figure rules inside the domain
skills, and the risk specialist combines planning and figures in
`risk-planning-and-visualisation`. Two names look like core skills but are not:
`petro-core-scal` concerns core samples, and `sed-core-description` the
vocabulary for describing drill core.

## How skills work with the rest of the system

### The system prompt indexes the skills

Each system prompt in `agents/system_instructions/` ends with a table of every
skill in the agent's pack and when to read it. The prompt holds the index and
the skills hold the content. A skill without a row in that table is rarely read,
and a skill renamed without editing the table sends the agent looking for a
document that does not exist. Nothing checks the table.

### Skills are read from GitHub, not from this folder

Each agent reads its skills in its own TrueForge sandbox. TrueForge stores a
skill as a reference to a Git repository, not as text:

```json
{"type": "git", "name": "seismic-core",
 "url":  "https://github.com/ARCHIBONG1/Geo-Oracle",
 "path": "seismic-interpreter-agent-skills/seismic-core",
 "ref":  "main",
 "description": "..."}
```

This has three consequences.

- An edited `SKILL.md` has no effect until it is committed and pushed to the
  branch in `ref`. The agents read GitHub, never the working copy.
- `ref` is `main`, so a push changes how the agents work immediately, with no
  deployment step. A tag or commit can be used instead if a release needs
  pinning.
- The stored `description` is a separate copy of the front matter, used for
  listing. `ops/provision_trueforge.py` refreshes it from the local file.

`ops/agents.toml` declares the repository URL, the branch and a path prefix in
`[skills]`, and lists each pack's directories in `[skills.packs]`. The
provisioner registers them with TrueForge and refuses to run if the listing and
the directories disagree. Because the registered path is
`<root>/<pack>/<name>`, moving the packs within the repository needs only a
change to `root` and a re-provision.

### Reading costs budget

Every skill read counts against the agent's iteration limit, and a run that
reaches the limit returns nothing. That is why procedures are split into many
short documents: an agent answering a porosity question reads `petro-core` and
`petro-porosity`, not saturation height and rock physics as well. Geo Oracle
makes one tool call per step, so each of its skill reads is alone in its step.

### The gateway checks two of Geo Oracle's reads

The gateway refuses a specialist call until Geo Oracle's chat has read
`geo-specialist-tasking`, and refuses a checkpoint until it has read
`geo-result-review`. A read that came back as a truncated preview, without its
end marker, does not count. After a context compaction Geo Oracle reads both
again, because the gateway cannot see a compaction.

### Skills name the tools

A skill is written against a particular set of tools (`seismic_describe_section`,
`petro_vshale`, `pt_render_closure_map` and so on), and the two must change
together: renaming a tool silently invalidates every skill that names it. Where
no tool exists for an analysis, the skill says so, so the agent records the gap
in its limitations instead of improvising.

Skills also carry method identifiers from the provenance registry
(`m:seis/edge-semblance`) and name the pack's method lookup tool
(`seismic_method_provenance`). Each core skill tells the specialist to carry a
result's method identifiers into its findings next to the provenance identifier,
and to answer any question about an equation or a source from the lookup tool
rather than from memory. Geo Oracle's pack does the reverse: it has no lookup
tool, quotes identifiers from the ledger as names only, and refers deeper
questions to the specialist that owns the method.
`tests/test_provenance_registry.py` checks offline that every identifier a skill
quotes exists and that every pack names its lookup tool.

A renamed tool must also be renamed in the provenance registry, which every tool
server checks as it starts. The full list of places a tool appears is the
checklist "Adding, renaming or changing a tool" in
[src/geo_oracle/tools/README.md](../../src/geo_oracle/tools/README.md).

### Some skills depend on tools outside this repository

The literature pack is written for the OpenAlex index (`search_works`,
`get_work`, `resolve_references`, `list_citations`, `search_entities`,
`get_entity`, `find_candidate_works`) and the Exa and Tavily search services;
`lit-search-protocol` and `lit-source-verification` are largely procedures for
using them correctly. Geo Oracle's pack relies on two TrueForge built-in tools:
`get_current_datetime`, the clock behind the time budget in
`geo-investigation-planning`, and `ask_user_question`, the only channel to the
person, on which the supervision gates depend. No test here can check these.

The principle matches the rest of the system: a number must come from a tool in
this repository, which carries provenance; a citation must be resolved through
the external index.

### The core skills define what the gateway parses

The core skills fix the evidence classes (`observation`, `measurement`,
`interpretation`, `inference`, `hypothesis`), require each interpretation to
list its supporting evidence, assumptions and limitations, set the rules for
claim status, and fix the format of missing data. These are exactly the fields
of the `SpecialistFindings` model that `parse_findings()` in `geological_mcp.py`
builds from a specialist's reply.

```
the skill says what to return
        │
        ▼
the specialist returns one JSON object
        │
        ▼
parse_findings()  ──▶  SpecialistFindings  ──▶  identifiers scoped as <task_id>/E1 ...  ──▶  Geo Oracle
```

A skill that drifts from the model raises no error: the reply fails to parse and
reaches Geo Oracle as raw text with a warning. The content survives, but it is
no longer structured evidence. The core skills and the reply models in
`geological_mcp.py` must therefore be changed together.

## The packs

### `geo-oracle-skills` (8): the orchestrator

The specialist packs govern doing geology; this pack governs running an
investigation, and each skill corresponds to gateway machinery rather than to a
compute tool.

| Skill | Governs |
| --- | --- |
| `geo-investigation-planning` | turning an objective into testable questions, tasks and a dependency graph; waves of tasks started one call at a time; the time budget; the risk gate and the reduction loop, including settled failures |
| `geo-evidence-inventory` | characterising the available evidence and packaging it for specialists |
| `geo-specialist-tasking` | the brief for each specialist tool and the job tools, field by field, with common mistakes; starting several specialists one call at a time |
| `geo-result-review` | the checklist for judging a specialist result before relying on it |
| `geo-investigation-ledger` | the format of `investigation/ledger.md`, the running record the synthesis is written from |
| `geo-play-prospect-evaluation` | the gates before the play, prospect and risk stages; the five domains' elements; the element status scale; the maturity ladder; materiality classes in place of a chance of success |
| `geo-synthesis-audit` | the structure and citation rules of a synthesis, and its scientific audit |
| `geo-human-on-the-loop` | the supervision question, the objective and deliverables recorded at activation (one result per deliverable), the checkpoint, accept, refuse and edit, partial deliveries, changing mode mid-investigation, and the work gate |

`geo-human-on-the-loop` is the counterpart of `hotl.py`: the gateway enforces the
gates by refusing calls, and this skill tells Geo Oracle how to satisfy them.

### `literature-review-agent-skills` (7)

| Skill | Covers |
| --- | --- |
| `lit-search-protocol` | search planning: expanding terms across formation, historical and stratigraphic names |
| `lit-inputs-library` | listing the shared inputs folder completely and confirming what is there |
| `lit-source-appraisal` | the authority, evidence basis, geographic relevance and independence of a source |
| `lit-source-verification` | resolving works to DOIs or OpenAlex identifiers; verifying every citation |
| `lit-evidence-extraction` | mapping extracted evidence onto the findings structure |
| `lit-geoscience-question-bank` | question sets and evidence checklists for decomposing a request |
| `lit-synthesis-report` | synthesis across sources, conflicting literature, careful language for consensus |

### `regional-geology-agent-skills` (9)

| Skill | Covers |
| --- | --- |
| `regional-core` | the task procedure, scope tags, the transfer checklist, the memo |
| `regional-data` | areas of interest, the reference-data catalogue, supplied GIS files, grid sampling |
| `regional-stratigraphy` | time-scale discipline, chronostratigraphic charts, hiatuses and overlaps |
| `regional-structure-stress` | fault trends, the orientation of maximum horizontal stress, stress regime, domain boundaries |
| `regional-thermal` | heat-flow statistics, conductive geotherms, calibration to corrected well temperatures |
| `regional-basin` | backstripping and decompaction, stretching fits, burial history, the basis of each expectation |
| `regional-potential-fields` | gravity and magnetic transforms, lineaments as observations, depth to source |
| `regional-seismicity` | catalogue summaries, magnitude of completeness, b-values and their limits |
| `regional-visualisation` | maps, rose diagrams and profiles; describing before interpreting |

### `seismic-interpreter-agent-skills` (8)

| Skill | Covers |
| --- | --- |
| `seismic-core` | the operating procedure and the findings contract |
| `seismic-data-qc` | uploads, SEG-Y header scanning, ingest decisions, resolution |
| `seismic-attributes` | choosing, setting up, running and reading the attribute tools |
| `seismic-structural` | horizon tracking with seeds guided by the synthetic, faults and throw, structure and isochron maps, closure and spill |
| `seismic-attribute-interpretation` | bodies by recipe: channels by stratal slices first and per sample second, in a window hung on a horizon, with wells; faults when the fault-likelihood check looks poor; karst and other bodies; constraints, seeds and the catalogue |
| `seismic-stratigraphic` | terminations, key surfaces, seismic facies, stratal slices |
| `seismic-qi` | when amplitudes can be trusted; wavelet, synthetic and well tie, including the range of phase the tie searches; AVO |
| `seismic-visualisation` | reading `seismic_describe_section`, the agent's only view of a section |

### `petrophysics-agent-skills` (13)

| Skill | Covers |
| --- | --- |
| `petro-core` | the task procedure, analysis mode, domain emphasis |
| `petro-data-qc` | imports, LAS and CSV loading, mnemonics, units and their conflicts, headers |
| `petro-shale-volume` | choosing a shale-volume method and endpoints; comparing at least two methods |
| `petro-porosity` | density, neutron-density and sonic porosity by lithology, fluid and hole condition |
| `petro-saturation` | the order of preference for water resistivity, temperature correction, Archie against shaly-sand models |
| `petro-saturation-height` | Leverett J and Brooks–Corey fits, laboratory-to-reservoir conversion |
| `petro-permeability` | core transforms with 10th to 90th percentile bands, flow units, Timur |
| `petro-net-pay` | cut-offs, evaluation with bad hole excluded, zone summaries |
| `petro-lithology` | matrix identification, multimineral solutions checked by reconstruction |
| `petro-core-scal` | core-to-log depth shift, overburden correction, log against core statistics |
| `petro-rock-physics` | fluid properties including CO2, measured or predicted shear velocity, Gassmann substitution and the clay moduli in the mineral mix |
| `petro-pressure-temperature` | pretest quality control, gradient legs, contacts from gradient intersections |
| `petro-time-depth` | checkshots, datums, offset correction, sonic integration with drift |

### `structural-geology-agent-skills` (8)

| Skill | Covers |
| --- | --- |
| `structural-core` | the task procedure, the rule against inventing structure, the validity record |
| `structural-data-qc` | other agents' products and sidecars: domain, pick uncertainty, scope, validity |
| `structural-geometry` | orientation statistics, mean planes and sets, fold axes by the pi method |
| `structural-displacement` | throw profiles, displacement against length, growth by expansion index |
| `structural-networks` | fault-network topology, connectivity, relative ages |
| `structural-modelling` | fault planes and horizons per block, every horizon asked for, the rule for closure size, validity tests, realisations |
| `structural-geomechanics` | the stress tensor with depths by datum, slip and dilation tendency, critical pore pressure |
| `structural-fault-seal` | juxtaposition, shale gouge ratio, shale smear factor, column heights |

### `stratigraphy-geology-agent-skills` (8)

| Skill | Covers |
| --- | --- |
| `strat-core` | the two ledgers, the order of preference for tie points, the consistency record |
| `strat-data-qc` | correlation logs and well tops, sidecars, the depth rule for deviated wells |
| `strat-log-correlation` | ranking tie points, dynamic time warping constrained to anchors |
| `strat-sequence` | stacking patterns from gamma ray, candidate maximum flooding and maximum regressive surfaces |
| `strat-chronostratigraphy` | age models from dated samples and datums, the seeded envelope, missing time |
| `strat-biostratigraphy` | biostratigraphic events as tie points, event ordering, graphic correlation |
| `strat-unconformities-thickness` | isochores and isopachs, minimum eroded thickness, missing time at a hiatus, erosion against fault cut-out |
| `strat-visualisation` | correlation panels, the two-datum check, what a panel may support |

### `sedimentology-agent-skills` (7)

| Skill | Covers |
| --- | --- |
| `sed-core` | the interpretation ladder; diagnostic evidence before suggestive |
| `sed-core-description` | the fixed vocabulary: Wentworth classes, structure and contact codes, bioturbation index |
| `sed-data-qc` | registering descriptions, analyses, analogue tables and photo indexes; the depth rule |
| `sed-facies-analysis` | transitions, the embedded Markov test, Walther's law, excluding key surfaces |
| `sed-depositional-systems` | the diagnostic matrix for each environment; how an association becomes an environment |
| `sed-carbonates` | carbonate texture (Dunham; Embry and Klovan), grains and fabrics, the carbonate code, the facies belts |
| `sed-visualisation` | graphic logs and transition diagrams; what a figure may support |

### `subsurface-play-agent-skills` (8)

| Skill | Covers |
| --- | --- |
| `play-core` | the ordinal status scale, including disfavoured for evidence against; the scope rule; the weakest link; dependencies |
| `play-data-qc` | other agents' products and claims, their ledgers, scope and transfer |
| `play-petroleum-systems` | hydrocarbon elements and processes, the events chart, the critical moment as stated by the investigator |
| `play-co2-storage` | CO2 storage elements with containment kept separate; the pressure limit; trapping through time |
| `play-geothermal` | convection- and conduction-dominated plays; heat, permeability, fluid, recharge |
| `play-hydrogen` | natural hydrogen and underground hydrogen storage; why timing differs |
| `play-well-outcomes` | every well as a success, a failure on a named element, or undetermined |
| `play-visualisation` | status matrices, events charts, dependency graphs |

### `prospect-target-agent-skills` (7)

| Skill | Covers |
| --- | --- |
| `prospect-core` | the maturity ladder, readiness tests R1 to R7, the refusal rules |
| `prospect-data-qc` | closures, play matrices, ledgers and companion tables; registering constraints |
| `prospect-definition` | closures into candidates, what each readiness failure means, the maturation plan |
| `prospect-scenarios` | trap, fill and contact alternatives, and what would separate them |
| `prospect-volumetrics` | gross rock volume from a closure's cells, cut at the reservoir base and computed per realisation; sourced inputs; the CO2 displacement efficiency |
| `prospect-target` | the well target under licence, keep-out and existing-well constraints |
| `prospect-visualisation` | closure maps and maturity tables; what a figure may support |

### `risk-uncertainty-agent-skills` (7)

| Skill | Covers |
| --- | --- |
| `risk-core` | no scores; materiality by flip test, carrying the test's provenance identifier; no new geology |
| `risk-data-qc` | importing Geo Oracle's ledger and reading it as data; product sidecars |
| `risk-register` | collecting every uncertainty from the ledger, findings and sidecars, and typing it |
| `risk-materiality` | the four flip tests (play status, maturity, threshold straddle, scenario comparison) and the classes they give, including a settled failure |
| `risk-failure-modes` | failure modes by domain from a fixed library: mechanism, elements, detection |
| `risk-audit` | contradictions carried with both sides; load-bearing assumptions |
| `risk-planning-and-visualisation` | the reduction plan: actions, owners, modes, neutral framing |

## Working on skills

**Adding a skill.** The skill does not exist for an agent until all four steps
are done.

1. Create `<pack>/<tag>-<subject>/SKILL.md` with `name` matching the directory
   and a `description` that says when to read it, naming any tool it governs. End
   it with `<!-- end of skill: <name> -->`.
2. Add a row to the skills table at the end of
   `agents/system_instructions/<agent>_system_prompt.md`, or it will not be read.
3. Add the directory name to the agent's pack in `[skills.packs]` in
   `ops/agents.toml`, then run `python ops/provision_trueforge.py --apply`.
4. Commit and push. TrueForge reads the skill from GitHub, so until it is pushed
   the skill is registered but empty.

**Changing a skill.** Check four links: the tool names it mentions; anything a
core skill says about evidence classes, claim status, scientific status or
missing data, which must match the models in `geological_mcp.py`; the skills
table in the system prompt, which nothing checks; and the pack listing in
`ops/agents.toml`, which the provisioner checks. Renaming a skill touches three
of these at once. Editing only the body touches none of them, but still needs a
push, and the end marker must stay the last line.

**Deleting a skill.** Remove its row from the system prompt first, because a
prompt that names a missing skill makes the agent spend an iteration looking for
it. Then remove it from `[skills.packs]` and re-provision. The provisioner never
deletes a registration, so the old entry stays in TrueForge's settings until it
is removed there by hand; it is harmless unless an agent is attached to it.

**Conventions throughout.** Describe before interpreting. Never upgrade a
classification, only downgrade it. Cite the provenance identifier of every
number. Record a missing capability in the limitations rather than working
around it. State what a figure may not be used to support. Software enforces
none of these; they hold because every core skill repeats them.
