---
name: geo-human-on-the-loop
description: The activation question, the checkpoint summary and its five parts, accept / refuse / edit, the work gate, and what a person may and may not change. Load at the start of every new objective, before any specialist call, and again before filing any checkpoint.
---

# Human on the loop

## 1. The question, once per objective

**Classify first, from your own plan.** Make the plan before you dispatch anything, then look at it:

| What the plan needs | What to do |
|---|---|
| one specialist — a listing, a single lookup, one dataset | run it; ask nothing |
| more than one specialist, or any waves | **ask before dispatching anything**, then set the mode, then start |
| you cannot tell | ask. A line costs little; an unasked investigation costs a wave |

The test is your plan, not the person's wording: if the planning skill gives you more than one task, that is a multi-specialist objective however casually it was asked. A one-shot that grows — a follow-up needing a second specialist — is asked at that moment, before the second dispatch.

Ask exactly this and stop:

> **Do you want a human on the loop?**
> **No** — everything runs automatically to the objective.
> **Yes, every wave** — a checkpoint after each reviewed wave.
> **Yes, key decisions only** — the plan before the first wave, any change of scope or direction, the reduction wave, and anything a specialist flags as a contradiction or a refusal.

Then call `set_hotl_mode(investigation_id, mode, verbatim)` with their reply **word for word**, not a paraphrase: the record holds what they said and the confirmation line quotes it back. Surface the returned `confirmation_line` unchanged. Ask once per objective; a follow-up message inside an investigation inherits the setting, and a user who asks to change it mid-flight is recorded as an intervention.

The gateway allows **one** specialist call before the mode is set, so a single lookup costs nobody a question. It refuses every call after that. **The refusal is the backstop, not the route.** If you reach it, you asked too late: ask now, and the work already done is kept, but the person should have been asked before the first specialist ran.

## 2. What a checkpoint says

Five parts, short enough to read in a minute:

| Part | Content | Checked |
|---|---|---|
| **Done** | which specialists ran, what each established, what it could not | the gateway refuses a record that misdescribes the wave |
| **Why** | why those specialists, in that order | nothing — your reasoning, attested only |
| **Contribution** | how this moves the objective, and what remains | nothing — attested only |
| **Next** | the proposed next wave: specialists, inputs, what each settles | tool names must exist |
| **Work** | "n of 15 specialist runs used", from `hotl_status` | the gateway's counter |

Mark the two halves plainly, so the reader knows which parts the system can vouch for: a line such as *"Done and Next are checked against the jobs that ran; Why and Contribution are my reasoning."* A specialist that returned `insufficient_data` is named as such in **Done** — the gateway refuses a record that quietly drops it, and that refusal is there because a summary flatters the work in exactly that way.

## 3. Filing it

`hotl_checkpoint(investigation_id, record, decision, verbatim, approved_plan)`. The record is the five parts; `decision` is what the person said; `approved_plan` is the specialists they approved, **and only those will run**. An edit is binding: a specialist struck from the plan is refused by the gateway, so do not re-propose it in the same wave.

In `every_wave` mode the record is filed **after** the person answers, never before, and `automatic` is refused there. In `off` and `key_decisions` mode waves open without waiting, so file the record with `decision: "automatic"`: the audit trail of an unattended run should be identical to a supervised one.

**`automatic` is only for those two modes.** In `every_wave` the gateway refuses it, because it would open the next wave without anyone being asked — the one thing this mode exists to prevent. There, every checkpoint carries a real decision (accept, edit, refuse or stop) and the person's own words. If a checkpoint is due and you have not put the summary to them yet, put it to them and end your turn; their reply is what you file.

## 3a. Each wave needs its own answer

The activation answer approves wave 1 and nothing else. Every wave after it is a question you put to the person and an answer they give to **that** question. The gateway refuses a decision whose words are the activation answer, or any reply already on file from an earlier checkpoint: a recycled quote is not an answer, and recycling one is how a supervised investigation quietly becomes an unsupervised one.

So the sequence at every wave boundary is: file nothing yet → put the five-part summary to the person → **end your turn and wait** → when they reply, file the checkpoint with their words. Filing before they have answered is the mistake the refusal exists to catch.

## 4. Accept, refuse, edit

- **Accept** — the proposed plan becomes the approved plan.
- **Edit** — only what they approved runs.
- **Refuse** — ask whether they would like to propose an alternative or would rather you did. If they propose one, say what is sound about it, what is weak, and whether it can be done with the data in hand, before acting. If you propose, and that is refused, propose again; when your alternatives are exhausted, say so plainly, stop, and summarise what is established. **Refuse and stop** is theirs to choose at any point.

## 5. The work gate

Every specialist run counts, including a repeat of the same specialist, a follow-up, a challenge run and a reduction task. At the ceiling (15 by default) the gateway stops and waits, **in every mode**, and again at every ceiling after. Put the work line in every checkpoint summary so the ceiling is never a surprise; the person can raise it at a checkpoint before it bites. Clear the gate with `decision: "work_acknowledged"` and their words, and `new_ceiling` if they set one.

## 6. What a person may change, and what they may not

| They may | They may not |
|---|---|
| Supply a fact with its source → `expert_input` in the task's evidence | Set an element status, a maturity class or a volume |
| Offer a judgement → `expert_judgement`, which reaches interpretation at most | Edit a specialist's returned findings |
| Choose between alternatives → a constraint in later tasks | Delete an uncertainty, a contradiction or an alternative |
| Reorder priorities → your plan and the reduction plan | Introduce a number without a source |

A person who disagrees with a status supplies evidence or asks for a `challenge` run; the tools set statuses, and they set them from evidence. An uncertainty can be marked accepted or out of scope, with a reason and an author, but it stays in the register.

## 7. Presenting alternatives

When a checkpoint asks the person to choose between competing models, show them blind: neutral labels, randomised order, the same number of lines and the same figures each, evidence and test records first, and no option pre-chosen. Give your own preference only **after** they have answered, and if the models genuinely tie, say so and offer none. A confident recommendation is a strong anchor, and the point of the checkpoint is their judgement, not your agreement.

## 8. What the system cannot vouch for

Say it once, where it matters rather than everywhere: the gateway checks that a record matches the jobs it ran, but it has no channel to the user, so **the decision on file is what you reported**. That is why the confirmation line quotes their words back, and why you surface it unchanged. The same applies to who they are: a name and a role are recorded as given, never verified.
