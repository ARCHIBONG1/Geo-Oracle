---
name: sed-facies-analysis
description: Facies transitions, the embedded Markov test, Walther's law and the key-surface exclusion, pooling across wells, and facies associations with their successions. Load before sed_transitions and sed_associations.
---

# Facies analysis

## Transitions (`sed_transitions_all`; `sed_transitions` for one well)

- Upward transitions between facies across bed boundaries in one well; the embedded Markov chain test (Powers & Easterling 1982) compares the counts with those expected under quasi-independence (Goodman 1968), the model of a random ordering in which a facies never follows itself. The test reads `chi_square` on `dof` against `critical_95`. Only when randomness is rejected (`non_random`) are transitions singled out: those with a normalized difference Z = (O - E)/sqrt(E) above 2 (`above_random`); these are the evidence of an association. With randomness accepted, no transition is singled out, since a positive difference is then chance.
- Walther's law holds only within conformable successions: give the framework's key surfaces (`key_surfaces_m`, same depth reference) and the boundaries at them are excluded. Without the framework, say that the test includes unconformable boundaries.
- Too few transitions give the test no power: S3 reads not tested (`min_transitions`, ten by default, a power convention you can change with a reason), and the count available is reported with the result so a reader can judge it.

## Pooling and associations

- `sed_transitions_all` pools the counts across the wells, reruns the test and finds the associations in the same call; its `pooled_transitions` file is what S3 uses. `sed_associations` does the pooling by hand from per-well files.
- Pooling first tests whether the wells share their transition probabilities (`homogeneity`, Powers & Easterling 1982). If they do not, the composite singles out no transition and the tool warns: read the wells separately, or ask whether they record different environments.
- An association is a connected group of above-random links, with its most likely upward succession. Name it (A1, A2) and keep the facies list with it.

## Reading the result

- A non-random matrix with a cycle (channel base, cross-bedded sand, rippled sand, mud) is an association; a non-random matrix with one link only is weak evidence.
- Self-transitions are not counted; a facies split too finely creates artificial links, one coded too coarsely hides them. Revisit the scheme before the environment.

## Reporting

Counts, chi-square and above-random links as `[measurement, derived]`; the association and its succession as `[interpretation]` at the association rung.
