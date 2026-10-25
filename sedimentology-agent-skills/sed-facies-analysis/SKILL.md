---
name: sed-facies-analysis
description: Facies transitions, the embedded Markov test, and Walther's law and the key-surface exclusion. It also covers pooling across wells, and facies associations with their successions. Load before sed_transitions_all, sed_transitions and sed_associations.
---

# Facies analysis

## Transitions (`sed_transitions_all`; `sed_transitions` for one well)

- The tool counts upward transitions between facies across bed boundaries in one well.
- The embedded Markov chain test (Powers & Easterling 1982) compares the counts with those expected under quasi-independence (Goodman 1968). Quasi-independence is the model of a random ordering in which a facies never follows itself. The test reads `chi_square` on `dof` against `critical_95`.
- The tool singles out transitions only when randomness is rejected (`non_random`). It singles out those with a normalized difference Z = (O - E)/sqrt(E) above 2 (`above_random`). These are the evidence of an association.
- If randomness is accepted, the tool singles out no transition, since a positive difference is then chance.
- Walther's law holds only within conformable successions. Give the framework's key surfaces (`key_surfaces_m`, same depth reference), and the tool excludes the boundaries at them. Without the framework, say that the test includes unconformable boundaries.
- Too few transitions give the test no power. S3 then reads not tested (`min_transitions`, ten by default, a power convention that you can change with a reason). The result gives the count available, so that a reader can judge it.

## Pooling and associations

- `sed_transitions_all` pools the counts across the wells, reruns the test, and finds the associations in the same call. S3 uses its `pooled_transitions` file. `sed_associations` does the pooling by hand from per-well files.
- Pooling first tests whether the wells share their transition probabilities (`homogeneity`, Powers & Easterling 1982). If they do not, the composite singles out no transition, and the tool warns. Read the wells separately, or ask whether they record different environments.
- An association is a connected group of above-random links, with its most likely upward succession. Name it (A1, A2) and keep the facies list with it.
- `sed_transitions` draws the well's transition diagram. `sed_transitions_all` and `sed_associations` draw the pooled one. Each returns it in `result.figures`. Publish it (sed-visualisation).

## Reading the result

- A non-random matrix with a cycle (channel base, cross-bedded sand, rippled sand, mud) is an association. A non-random matrix with one link only is weak evidence.
- The tool does not count self-transitions. A facies split too finely creates artificial links, and one coded too coarsely hides them. Revisit the scheme before the environment.

## Reporting

- Counts, chi-square and above-random links: `[measurement, derived]`.
- The association and its succession: `[interpretation]`, at the association rung.

<!-- end of skill: sed-facies-analysis -->
