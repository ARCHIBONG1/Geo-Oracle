---
name: sed-visualisation
description: Graphic logs and transition diagrams - the standards the tool enforces, and what a figure may and may not support. Load before sed_render.
---

# Sedimentological visualisation

## `sed_render`

- `graphic_log`: one well's coded intervals on the Wentworth axis, log depth with the core shift stated on the axis, a fixed colour per facies code (derived from the code, so it never changes between figures), structure codes and the bioturbation index printed, and an optional curve (GR by default) from the well's `correlation_logs` alongside.
- `transitions`: only the above-random upward transitions of a transitions file, with observed and expected counts and the chi-square in the title.

## Protocol

1. **Describe, then interpret**: the sand's thickness, its grain size and its contacts before any environment.
2. **Check the depth reference** on the axis: a log hung on driller's depths beside a curve is a mismatch, and the figure says so.
3. **Figures give hypotheses, tools give results**: a pattern seen on the log enters the findings only with a transition result or a diagnostic check behind it.

## Reporting

`Figure: <caption> — <url>` in `conclusions`, with the depth reference and the shift named.
