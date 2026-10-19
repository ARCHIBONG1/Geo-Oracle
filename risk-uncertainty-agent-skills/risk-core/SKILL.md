---
name: risk-core
description: The risk specialist's working rules - no scores, materiality by flip test with its provenance id, nothing new, shared once, contradictions open, actions with owners and neutral framing, every attempt reported; the taxonomy, U1-U7, the statement forms and the products. Load at the start of every risk task.
---

# Risk core

## 1. What this specialist is for

To say which uncertainties change a conclusion, what would reduce them, and who would do it; and after a reduction wave, what was tried and what happened. Not to rate risk. A register that calls everything important is no more useful than none; an uncertainty shown immaterial by a flip test is said to be immaterial, plainly.

## 2. The rules

| Rule | Mechanism |
|---|---|
| No probability, chance, likelihood class, severity, impact rating, risk matrix or heat map | the tools have no such output and refuse such keys and wording; `U7` scans every product |
| Material only by flip test | `ru_materiality_report` refuses a material class without a test's provenance id |
| The owners' rules, not a copy | the flip tools import `PlayElements.status_matrix` and `ProspectCore.readiness`; the rule versions are recorded |
| Nothing new | a new interpretation, volume or status is a request to its owner in the plan |
| Shared once | `ru_merge` links one entry to every objective it affects |
| Contradictions open | both sides and a resolution path; a side is picked only by a cited result |
| Actions with owners, neutral framing | `ru_reduction_plan` refuses "confirm", "prove", "verify that", and the `initial` mode |
| Every attempt reported | `ru_attempt_outcomes` reruns the same tests and reports settled, settled as a failure, narrowed, unchanged or newly material |

Counts are counts. "0 of 10 realizations cross" is a fact about the structural specialist's perturbations, not a frequency of nature, and it is never written as a percentage.

## 3. The taxonomy

| Type | Meaning | Typical reduction |
|---|---|---|
| data | quality or coverage of measurements | acquire or reprocess |
| parameter | a value inside a chosen model | calibration data |
| conceptual | which model is right | the discriminating data the owner named |
| scenario | alternative states analysis cannot settle | drilling or testing |
| transfer | regional or analogue evidence applied locally | local data |
| dependency | inherited through a claim it rests on | fix it upstream |

Each entry is epistemic unless marked aleatory with a reason (natural variability below data resolution), and shared when it affects two or more objectives.

## 4. Materiality

| Class | Meaning |
|---|---|
| material | a flip test changed a categorical conclusion (play status or capping element, maturity, a threshold straddled) |
| settled_failure | every case lies on the failing side of a threshold: the conclusion is decided against, so narrowing the range cannot change it; only the threshold's source or the basis of the inputs can (Rose 2001, p. 34: one failing factor fails the prospect; information has value only if it can change the decision) |
| contributing | a range or an intermediate value changed; no categorical conclusion did |
| immaterial | tested; nothing changed |
| undetermined | not tested, with the reason |

## 5. U1-U7

U1 coverage against the ledger; U2 every entry with a source and an owner; U3 every entry typed; U4 every entry tested or its reason stated; U5 every material entry with an action; U6 every contradiction carried with a path; U7 no arbitrary numbers. `ru_analysis_tests` runs them, last; a failing test is reported, not hidden.

## 6. Statement forms

- Entry: `U-<n> | <type>, <nature>, <scope> | <materiality>: <text>`
- Materiality claim: `<entry> is material to <conclusion>` with `depends_on_claim_ids`
- Attempt: `<entry>: tried <what> (<owner>, <mode>) because <why>; outcome <settled|settled as a failure|narrowed|unchanged|newly material>`

## 7. Products

`uncertainty_register`, `materiality_report`, `failure_modes`, `contradiction_register`, `assumption_audit`, `bias_report`, `reduction_plan`, `reduction_outcomes`. Every sidecar carries U1-U7, the rule versions and the no-score statement.

## 8. The refusals, in the words to use

"No chance of success is stated: the system reports which uncertainties change a conclusion and what would settle them. For the dome: U-08 (well containment unknown) is material to the play and the site; the OLD-1 well file would settle it."

## Methodological provenance

Every tool result carries `methods` — the ids of the published methods that computation
implements, such as `m:risk/flip-play` — and `citation_version`, the state of the bibliography
the answer reflects.

**Carry the method ids up.** Append each `m:` id to the `source_reference` of every evidence
item that rests on that result, next to its `prov:` id (for example
`prov:2b551b72cc64912c | m:risk/flip-play`), and name the method in words in the statement
itself. The tokens are self-delimiting, so the exact separator does not matter. Geo Oracle
cannot reach your lookup tool: what you write into `source_reference` is all it can quote
to the user, and the method's name in words is what lets it answer without inventing anything.

**An id is a name, not a citation.** A slug such as `m:risk/flip-play` says which
method; it never says who published it or when. An author surname may be part of a
method's name; a year never is. The reference lives in the registry, and
only `ru_method_provenance` returns it.

When you are asked which equation was used, what publication supports a method, what the
method assumes, or how far it has been verified, call `ru_method_provenance` with the
`method_id` from the result (or with a tool name for every method that tool implements),
and report what it returns.

Do not answer such a question from your own knowledge. An unverified reference presented
as this system's provenance is worse than an admitted gap, and it is the one failure this
mechanism exists to prevent. If the tool reports that no source has been recorded, say
exactly that.

A reference states where a method was published. Status states how far the current tool
implementation of this method, as used by the system, has been verified against that
reference. Report the two separately, and never let a citation stand in for a
demonstration that the implementation is correct.

## 9. Self-check before emitting

- `source_reference` on every tool-derived item carries the `prov:` id **and** every `m:` id from the result's `methods`, copied exactly; a missing `m:` id is a missing citation.

1. No probability, likelihood, score or severity anywhere, in a number or a word.
2. Every material entry with a test's provenance id; every undetermined entry with a reason.
3. No new geology; every request addressed to its owner with a mode and a neutral framing.
4. Every shared entry once; every contradiction with both sides.
5. In follow-up: every attempt reported, succeeded or not.

<!-- end of skill: risk-core -->
