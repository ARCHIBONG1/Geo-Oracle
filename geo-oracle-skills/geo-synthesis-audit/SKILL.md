---
name: geo-synthesis-audit
description: Structure, citation rules and the mandatory scientific audit for Geo Oracle's syntheses, from interim model summaries to final investigation reports and prospectivity conclusions. Use before presenting any major synthesis, final report or conclusion about geology, plays, prospects or risk, and whenever the user asks "what do we know", "summarise the findings", "what's your conclusion" or "is this prospective".
---

# Synthesis and final audit

A synthesis is not a concatenation of specialist reports. It integrates their findings into one model, shows how the conclusions depend on the evidence, and is never stronger than its weakest essential link. A competent geoscientist should be able to reproduce the reasoning from the stated evidence and ids.

## 1. Citations

Attach a source to every material statement:

| Tag | For |
|---|---|
| `[T03-seis/E2]`, `[T05-struct/C1]` | Specialist evidence and claims (global ids) |
| `[user evidence: W1-tops]` | Evidence supplied by the user (inventory ID) |
| `[literature via T01-lit/E4]` | Published evidence returned by `literature_review` |
| `[Geo Oracle synthesis: from T03/E2, T05/C1]` | Your own integration, with its inputs |
| `[general knowledge — not project evidence]` | Background you supply for context |

Do not use material statements without a source. If you cannot cite something, it does not belong in the conclusions.

## 2. Structure of a full report

Scale this down for interim summaries, but always keep the citations, the uncertainties and the gaps.

1. **Objective and scope.** Include what was run: task, tool, status and disposition, including tasks that failed or were not executed.
2. **Evidence base.** The inventory summary, and the key data gaps.
3. **Established evidence.** Observations and measurements that are directly supported.
4. **Interpreted framework.** Regional, stratigraphic, sedimentological, structural, seismic and petrophysical interpretations, each with its support.
5. **Working hypotheses.** Each with its status from the ledger, and the test that would change it.
6. **Integrated geological model.** How the pieces fit, and which interpretation depends on which.
7. **Plays.** An element-by-element table of support labels, with ids.
8. **Prospects or targets.** Only those that passed the gate, with their maturity and the basis for it.
9. **Uncertainty.** The critical uncertainties, their consequences, and whether each is reducible.
10. **Alternative interpretations.** The competing models that remain plausible, and what would discriminate between them.
11. **Evidence gaps.** What cannot currently be established.
12. **Recommended next analyses or data acquisition.** Each linked to the uncertainty (`U#`) or hypothesis (`H#`) it would resolve.
13. **Decision implications.** What the current evidence permits the user to conclude, and what it does not.

Put limitations where they apply, not in a closing paragraph. Never let a prospectivity conclusion be stronger than the geological model beneath it.

## 3. The audit

Run this audit before presenting a major synthesis. Fix every failure. If a failure cannot be fixed, disclose it in the synthesis.

**Fabrication and honesty**
- Is every data item, number, reference and finding traceable to user evidence, a returned tool result, or a shown derivation?
- Was every specialist I say I consulted actually called, with a result returned?
- Was every piece of literature I cite returned by `literature_review`?
- Did I perform, myself, any substantive analysis that a specialist should have done?

**Evidence classes**
- Did I present an interpretation, inference or analogue as an observation?
- Did I treat dataset availability as evidence of a geological feature?
- Did any assumption become a fact along the way?

**Provenance and independence**
- Does each material statement carry a citation?
- Did I count correlated evidence as independent, e.g. several conclusions built on one seismic interpretation?
- Is consensus or repetition doing the work that evidence should do?

**Dependencies and change**
- After every upstream change, did I revisit the downstream conclusions? Are any still marked `needs review` in the ledger?
- Are all contradictions resolved or explicitly preserved?

**Plays, prospects and risk**
- Were plays evaluated by `subsurface_play`, with element labels that match their evidence?
- Does every prospect or target pass the gate, and is its maturity justified?
- Is there any circular reasoning (see `geo-play-prospect-evaluation`)?

**Uncertainty and numbers**
- Are the material uncertainties stated where they apply, including in the conclusions?
- Is every numerical estimate backed by a stated method, inputs and assumptions?
- Are unresolved questions that affect the conclusion stated as such?

**Execution**
- Are failed or not-executed analyses reported as such, rather than silently omitted?

## 4. After presenting

Update the ledger with a summary of the synthesis, the open questions and the next steps, so that the investigation can continue from exactly this state.
