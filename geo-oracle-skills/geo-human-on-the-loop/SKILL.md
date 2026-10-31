---
name: geo-human-on-the-loop
description: The activation question, the checkpoint summary and its five parts, accept / refuse / edit, the work gate, and what a person may and may not change. Load at the start of every new objective, before any specialist call, and again before filing any checkpoint.
---

# Human on the loop

## 0. How to ask — every question in this skill, without exception

Call **`ask_user_question`**, by that name. Both parameters are required:

| Parameter | Shape |
|---|---|
| `question` | the decision only, at most 500 characters: the card does not scroll, so a longer one hides the options. Write the summary as text in the message that makes the call |
| `options` | 0 to 5 mutually exclusive choices, plain strings. **A free-text box is always rendered beside them**, so the person can pick one or type something else |

The question **blocks until it is answered and the turn survives**, so the investigation carries on in the same turn. You cannot perceive the wait: the answer simply appears. Never reason about how long it took, and never say that a question returned immediately.

Before you ask, always **update the ledger, including its Pending question block.** A question holds the turn open. If nobody answers, the harness cuts the turn at its limit with no message. The ledger makes the work recoverable.

A worked example: the activation question. Every other trigger copies its shape.

```
ask_user_question(
  question: "This is a multi-specialist CO2 storage investigation: inventory and QC, then structure, "
            "then the play and site gates, then a risk review and its reduction wave. "
            "Do you want a human on the loop?",
  options: ["No - run automatically to the objective",
            "Yes, every wave - a checkpoint after each reviewed wave",
            "Yes, key decisions only - the plan, scope changes, the reduction wave, contradictions and refusals"]
)
```

**Three rules about options.**
- Five is the hard maximum, so design for three or four and let the free-text box carry the rest. An edit is free text, so accept / refuse / stop plus the box reads best.
- Append the exact suffix `" (Recommended)"` to one option **only** where you ask the person to sanity-check your judgement: a plan, a retry, a reduction wave. **Never at an alternatives checkpoint**: a recommendation is pre-selection in a thin disguise. That question asks for their judgement, not your agreement (section 7).
- More than five alternatives: section 7.

## 1. The question, once per objective

**Classify first, from your own plan.** Make the plan before you dispatch anything, then look at it:

| What the plan needs | What to do |
|---|---|
| one specialist — a listing, a single lookup, one dataset | run it; ask nothing |
| more than one specialist, or any waves | **ask before dispatching anything**, then set the mode, then start |
| you cannot tell | ask. A line costs little; an unasked investigation costs a wave |

The test is your plan, not the person's wording. If the planning skill gives you more than one task, the objective is multi-specialist, however casually the person asked it. If a one-shot grows (a follow-up needs a second specialist), ask at that moment, before the second dispatch.

Ask it with `ask_user_question` as section 0 shows, and carry on in the same turn when they answer:

> **Do you want a human on the loop?**
> **No** — everything runs automatically to the objective.
> **Yes, every wave** — a checkpoint after each reviewed wave.
> **Yes, key decisions only** — the plan before the first wave, any change of scope or direction, the reduction wave, and anything a specialist flags as a contradiction or a refusal.

**Mint a new `investigation_id` for a new investigation**, with the date and time in it: `INV-dome-co2-20261006T0504`. If you use the same name twice, the second investigation inherits the first's run count, approved plan and any checkpoint left due. A rerun of yesterday's question would then start two-thirds through its work ceiling. The gateway refuses activation on an id that already carries work, and the refusal tells you to pick a new one.

Then call `set_hotl_mode(investigation_id, mode, verbatim, deliverables=[...], objective=...)` with:
- their reply **word for word**, not a paraphrase (the gateway checks it, section 3);
- the objective in their words, in one sentence. The gateway records it once and shows it in every checkpoint's state. Open each checkpoint summary with it;
- the results they asked for, one per item, in their words. Each gets an id (`dlv1`, `dlv2`, ...). The checkpoint that ends the work gives each a status by its id: delivered, partial, not_deliverable or declined.

The record holds what they said, and the confirmation line quotes it back.

**One result per deliverable.** An item is one product or one answer the person can check: "closures on horizon A" and "closures on horizon B", not "closures on the main horizons"; "Shmin at the reservoir depth", not "stress tensor, slip tendency and SGR". The gateway refuses an item that lists several results, because one status cannot be honest about a bundle. A bundle closed as delivered hides its missing parts. At the closing checkpoint, give each result its own entry:
- `delivered`, with the job, task or product (`@agent/products/<file>`) of this investigation that holds it. The gateway checks that it exists;
- `partial`, with that reference and `missing` naming the part not there. It needs the person's decision or a follow-up wave, never an automatic close;
- `not_deliverable`, with the input that does not exist;
- `declined`, with the person's words.

The gateway refuses a "delivered" basis that says part of the result is unresolved, pending or not run: that is `partial`. Surface the returned `confirmation_line` unchanged, in the message that makes your next tool call (section 3c). Ask once per objective. A follow-up message inside an investigation inherits the setting. If a person asks to change it mid-flight, call `set_hotl_mode` again with `resume=true` and their words: the gateway records the change as an intervention. Never use `resume` to reuse an investigation id for new work.

The gateway allows **one** specialist call before the mode is set, so a single lookup costs nobody a question. It refuses every call after that. **The refusal is the backstop, not the route.** If you reach it, you asked too late. Ask now. The gateway keeps the work already done, but the person should have been asked before the first specialist ran.

## 2. What a checkpoint says

Five parts, short enough to read in a minute:

| Part | Content | Checked |
|---|---|---|
| **Done** | which specialists ran, what each established, what it could not, and what you did about every `Process audit:` flag (followed up, accepted with the reason, or rejected) | the gateway records the jobs, their status and figures itself, and refuses a wave with an unanswered process-audit flag |
| **Why** | why those specialists, in that order | nothing — your reasoning, attested only |
| **Contribution** | how this moves the objective, and what remains | nothing — attested only |
| **Next** | the proposed next wave: specialists, inputs, what each settles | tool names must exist |
| **Work** | "n of 20 specialist runs used", from `hotl_status` | the gateway's counter |

Mark the two halves plainly, so the reader knows which parts the system can vouch for. For example: *"Done and Next are checked against the jobs that ran; Why and Contribution are my reasoning."* Name a specialist that returned `insufficient_data` as such in **Done**. The gateway records that status itself and names it back in `needs_attention`, because a summary flatters the work in exactly that way.

## 3. Filing it

`hotl_checkpoint(investigation_id, decision, verbatim, approved_plan, record)`. `decision`, `verbatim` and `approved_plan` are arguments of the call, not fields of `record`.
- `record` holds `why`, `contribution`, `next` [{specialist, objective, settles}] and `items`. The gateway records the jobs, their status and their figures itself. Your prose is kept, not checked.
- **`items` answers the wave's items by id.** `hotl_status` lists them under `checkpoint`. First come the items with `actions`: process-audit flags and check outcomes (`chk`). Answer each: `{"T03-sed#audit1": {"action": "accepted", "note": "why"}}`. The other items keep their `default`: missing data, follow-ups and open outputs are carried, rivals kept open, figures carried. Answer one only to change it: dismiss or plan it, or mark a rival tested or resolved. One answer can cover every item of one kind of one task (`{"T03-sed#md*": {...}}`), never check outcomes. A note is required, except to confirm a default.
- The gateway keeps a refused record as a draft. File only what the refusal names, by id.
- From the sixth refusal of a wave, stop filing. In `off`, write the unanswered items into the ledger under *Pending checkpoint*, tell the person, and end the turn. In a supervised mode, put them to the person. **Never file a decision the person did not make to get past a refusal.**
- `decision` is what the person said. `verbatim` is their whole reply, exactly as given, or whole lines of their message. The gateway looks for it in this chat and refuses words the person did not write. `approved_plan` is the specialists they approved, **and only those will run**. An edit is binding in every mode: do not re-propose a struck specialist in the same wave.

After a refuse or stop, no specialist runs until you file their next decision.

**File a checkpoint record after every wave in every mode.** The gateway refuses the next wave until you file it. Only the pause differs:
- in `every_wave`, file the record **after** the person answers, with their words;
- in `off` and `key_decisions`, file it with `decision: "automatic"` and carry straight on.

An unattended run so leaves the same audit trail as a supervised one.

**`automatic` is only for those two modes.** In `every_wave` the gateway refuses it: it would open the next wave without anyone being asked. There, every checkpoint carries a real decision (accept, edit, refuse or stop) and the person's own words. If you have not asked them yet, use `ask_user_question` (section 3a). Their reply is what you file.

## 3a. Each wave needs its own answer

The activation answer approves wave 1 and nothing else. Every later wave is a question you put to the person, and an answer they give to **that** question. The gateway refuses a decision whose words are the activation answer, or any reply already on file from an earlier checkpoint. Recycled quotes are how a supervised investigation quietly becomes an unsupervised one.

At every wave boundary, follow this sequence:
1. File nothing yet.
2. **Update the ledger, including its Pending question block.**
3. Write the five-part summary as text, and in that message call **`ask_user_question`** (section 0). Give accept / refuse / stop as options, and let the free-text box carry any edit.
4. They answer in the same turn.
5. File the checkpoint with their words, and clear the pending block.

If you file before they answer, the refusal catches that mistake.

**Do not end your turn to ask.** `ask_user_question` blocks and the turn survives it. An answered checkpoint carries straight on, and the whole investigation runs in one turn.

**If nobody answers, the harness cuts the turn at its limit with no message.** That is the accepted cost of supervision: a supervised investigation is only as alive as its supervisor. Nothing is lost if you wrote the ledger first. That is why the pending block goes in before the question, not after. An investigation meant to run unattended is `off` mode, not `every_wave`.

## 3b. Key decisions only

The same machinery as every-wave mode, fired at four moments instead of at every boundary. **Each of them is an `ask_user_question` call, answered in the same turn, exactly as in every-wave mode (section 0).** Never prose alone, never a turn ending, never a statement of what you intend followed by a stop.

| Trigger | When | What you ask |
|---|---|---|
| **The plan** | once, before the first wave | the waves you intend to run, the specialists in each, and what each should settle. Their answer approves it |
| **A change of scope or direction** | whenever the plan you had approved no longer fits what you found | what changed, what you now propose instead, and why |
| **The reduction wave** | when the risk specialist's plan has actionable items | the items, their owners and modes, and what each would settle. The gateway sees a published `reduction_plan` and refuses a wave filed as "no key decision here" that contains one |
| **A contradiction or a refusal** | when a specialist flags either | both sides of the contradiction, or what was refused and why, and what you propose to do. The gateway sees contradictions: a wave filed as "no key decision here" that contains one is refused |

The last two can fire **mid-wave**, not only at a boundary. Ask when you reach them. Do not save them for the end of the wave: by then you may have built on the thing in question.

**Between triggers, waves open without waiting.** File the record with `decision: "automatic"` and carry straight on. The gateway still requires the record, so the audit trail matches a supervised run. Do not ask at a wave boundary in this mode. The person chose not to be asked there. To ask anyway is the same failure as not to ask in every-wave mode, from the other side.

**The plan trigger fires once.** Its answer approves the plan, not the whole investigation: a later change of scope is its own question. File the answer with `hotl_checkpoint` (accept or edit, with their words and the specialists they approved). The gateway then holds you to it for the wave it opens. It refuses a specialist outside the plan until the person agrees to a change. The gateway does not enforce a plan that you file as `automatic` between triggers.

## 3c. The confirmation line

Every `hotl_checkpoint` returns one. The gateway builds it from state you cannot set: the wave number, the specialists opening, the run count. Surface it to the person **unchanged**. Each line says which setting it acted under, so nobody can mistake a wave that opened without asking for one that skipped them:

```
Wave 2: Approved as recorded — opening: structural_geology. 4 of 20 runs used.
        Your words on file: "yes, but skip the seismic rerun"
Wave 2: HOTL Selection — Key Decisions Only (no key decision here) — opening: structural_geology. 4 of 20 runs used.
Wave 2: HOTL Selection — Unsupervised — opening: subsurface_play. 4 of 20 runs used.
Wave 3: Work gate — 20 runs used, continuing to 30. This gate holds even unsupervised.
        Your words on file: "yes, keep going, raise it to 30"
Wave 3: Refused — opening nothing. 7 of 20 runs used.
Wave 3: Stopped at your request — nothing further opened. 7 of 20 runs used.
```

Read the second and third lines out: to someone who asked for key decisions, a bare "Wave 2 automatic" reads as if you skipped them.

**Where it goes.** Put the line, unchanged, in the message that makes your next tool call. If nothing else runs, put it in your final answer. Never send it as a reply on its own: a reply with no tool call ends your turn, and in `off` or `key_decisions` nothing else opens the next wave. This also holds for the line that `set_hotl_mode` returns.

**Two of the four triggers are checked, not trusted.** If any specialist in the wave returned a contradiction, or published a `reduction_plan`, the gateway refuses a record filed as "no key decision here" and tells you to ask. The gateway cannot see a change of scope and some refusals, so those remain your judgement. If the plan you had approved no longer fits what you found, that is a trigger, whether or not anything flags it.

## 4. Accept, refuse, edit

- **Accept** — the proposed plan becomes the approved plan.
- **Edit** — only what they approved runs.
- **Refuse** — ask whether they would like to propose an alternative, or would rather you did.
  - If they propose one, say what is sound about it, what is weak, and whether the data in hand can support it. Do this before you act.
  - If you propose, and they refuse that, propose again. When your alternatives are exhausted, say so plainly, stop, and summarise what is established.
  - **Refuse and stop** is theirs to choose at any point.

## 5. The work gate

**The ceiling is theirs, not yours.** It starts at 20. The gateway refuses a different one unless the person's own words contain that number, in digits or spelled out. Do not decide that a long investigation deserves more rope. The work gate is the only gate that holds in every mode, so it is the one thing between an unattended run and unlimited spending. To choose your own ceiling is to choose how much rope you get. If you think 20 is too low for the scope, say so when you ask the activation question, and let them name a number. Or let the gate fire at 20 and ask then, which costs one interruption.

Every specialist run counts, including a repeat of the same specialist, a follow-up, a challenge run and a reduction task. At the ceiling (20 by default) the gateway stops and waits, **in every mode**, and again at every later ceiling. Put the work line in every checkpoint summary, so the ceiling is never a surprise. The person can raise it at a checkpoint before it bites. Clear the gate with `decision: "work_acknowledged"` and their words, and with `new_ceiling` if they set one.

## 6. What a person may change, and what they may not

| They may | They may not |
|---|---|
| Supply a fact with its source → `expert_input` in the task's evidence | Set an element status, a maturity class or a volume |
| Offer a judgement → `expert_judgement`, which reaches interpretation at most | Edit a specialist's returned findings |
| Choose between alternatives → a constraint in later tasks | Delete an uncertainty, a contradiction or an alternative |
| Reorder priorities → your plan and the reduction plan | Introduce a number without a source |

A person who disagrees with a status supplies evidence or asks for a `challenge` run. The tools set statuses, and they set them from evidence. An uncertainty can be marked accepted or out of scope, with a reason and an author, but it stays in the register.

## 7. Presenting alternatives

When a checkpoint asks the person to choose between competing models, show them blind:
- neutral labels and randomised order;
- the same number of lines and the same figures for each;
- evidence and test records first;
- no option pre-chosen: **no `" (Recommended)"` here** (section 0).

With more than five alternatives, say how many there are and offer those that differ most, or ask them to narrow it first. Do not drop one silently to fit. Give your own preference only **after** they have answered. If the models genuinely tie, say so and offer none. A confident recommendation is a strong anchor, and the checkpoint asks for their judgement, not your agreement.

## 8. What the system cannot vouch for

Say it once, where it matters, not everywhere. The gateway checks the record against the jobs it ran, and the person's words against what they wrote in this chat. It cannot check that **your decision matches what those words meant**, or who the person is (it records a name and a role as given). That is why the confirmation line quotes their words back, and why you surface it unchanged.

<!-- end of skill: geo-human-on-the-loop -->
