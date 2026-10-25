---
name: sed-visualisation
description: Graphic logs, transition diagrams and core-photograph panels. It covers the tools that draw them, the standards they enforce, and what a figure may and may not support. Load before sed_render and before you publish the figures the tools drew.
---

# Sedimentological visualisation

## What a figure is and is not

A figure illustrates a claim that a tool computed. It never carries a claim, and you never read a number, depth, area or boundary off it: those come from tools, as they always did. A striking figure can make an untested claim feel tested. That is the one way a picture can damage an argument.

You do not see the figures: the tool returns a URL, not an image. Check the numbers that the tool returns, never the picture. The figures are for the person, Geo Oracle's answer and the report. So the caption must say what the figure shows, from which product and version. It must say it well enough that a reader can read it without you.

## Figures the tools draw themselves

These arrive in `result.figures`. Publish each one.

- `sed_code_lithofacies`: the graphic log of each well (the six wells with most described thickness). If a claim rests on another well, draw it with `sed_render`.
- `sed_transitions`: the well's transition diagram.
- `sed_transitions_all` and `sed_associations`: the pooled transition diagram, from the file that feeds S3.

## `sed_render`

Draw with it every other product that has a renderer. It also returns `result.figures`.

- `graphic_log`: one well's coded intervals on the Wentworth axis. It shows:
  - log depth, with the core shift stated on the axis;
  - a fixed colour per facies code (derived from the code, so it never changes between figures);
  - the structure codes and the bioturbation index, printed;
  - an optional curve (GR by default) from the well's `correlation_logs`, alongside.
- `transitions`: only the above-random upward transitions of a transitions file, with observed and expected counts and the chi-square in the title.
- `core_photos`: one well's photographs from a registered `photo_index`, in depth order. Each is labelled with its top and base and its lighting, at its own proportions.
  - To draw one interval, give `top_m` and `base_m`. At most 24 photographs fit one figure.
  - To draw the graphic log of the same interval beside them on one depth axis, give `coded`.
  - The photo index and the coded intervals must have the same depth reference. Shift both to log depth with `sed_apply_depth_shift` first, or the tool refuses.

## Protocol

1. **Describe, then interpret**: the sand's thickness, its grain size and its contacts before any environment.
2. **Check the depth reference** that the tool returns. A log hung on driller's depths beside a curve is a mismatch, and the caption says so.
3. **Tools give results.** A pattern enters the findings only with a transition result or a diagnostic check behind it.

## Reporting

For each figure, add a `Figure: <caption> — <url> — supports <claim ids>` line in `conclusions`, with the depth reference and the shift named. Also add a `figures` entry.

<!-- end of skill: sed-visualisation -->
