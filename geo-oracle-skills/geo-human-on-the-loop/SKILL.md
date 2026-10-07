---
name: geo-human-on-the-loop
description: The activation question, the checkpoint summary and its five parts, accept / refuse / edit, the work gate, and what a person may and may not change. Load at the start of every new objective, before any specialist call, and again before filing any checkpoint.
---

# Human on the loop

## 0. How to ask — every question in this skill, without exception

Call **`ask_user_question`**, by that name. Both parameters are required:

| Parameter | Shape |
|---|---|
| `question` | the text. Put the whole summary in it; there is nowhere else for it to go |
| `options` | 0 to 5 mutually exclusive choices, plain strings. **A free-text box is always rendered beside them**, so the person can pick one, type something else, or both |

The question **blocks until it is answered and the turn survives**, so the investigation carries on in the same turn. You cannot perceive the wait: the answer simply appears, with nothing to tell you whether it took five seconds or an hour. Never reason about how long it took, and never say a question returned immediately.

Before asking, always: **update the ledger, including its Pending question block.** A question holds the turn open, and if nobody answers the turn is cut at its limit with no message. The ledger is what makes the work recoverable.

A worked example — the activation question, whose shape every other trigger copies:

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

**Three rules about options.** Five is the hard maximum, so design for three or four and let the free-text box carry the rest: accept / edit / refuse / stop is four, and "edit" is really free text anyway, so three plus the box usually reads better than five. Append the exact suffix `" (Recommended)"` to one option **only** where you are asking the person to sanity-check your judgement — a plan, a retry, a reduction wave. **Never at an alternatives checkpoint**: recommending is pre-selection in a thin disguise, and the point of that question is their judgement, not your agreement (section 7). And if more than five alternatives are in play, do not silently drop one — say in the question how many there are and offer the ones that differ most, or ask them to narrow it first.

## 1. The question, once per objective

**Classify first, from your own plan.** Make the plan before you dispatch anything, then look at it:

| What the plan needs | What to do |
|---|---|
| one specialist — a listing, a single lookup, one dataset | run it; ask nothing |
| more than one specialist, or any waves | **ask before dispatching anything**, then set the mode, then start |
| you cannot tell | ask. A line costs little; an unasked investigation costs a wave |

The test is your plan, not the person's wording: if the planning skill gives you more than one task, that is a multi-specialist objective however casually it was asked. A one-shot that grows — a follow-up needing a second specialist — is asked at that moment, before the second dispatch.

Ask it with `ask_user_question` as section 0 shows, and carry on in the same turn when they answer:

> **Do you want a human on the loop?**
> **No** — everything runs automatically to the objective.
> **Yes, every wave** — a checkpoint after each reviewed wave.
> **Yes, key decisions only** — the plan before the first wave, any change of scope or direction, the reduction wave, and anything a specialist flags as a contradiction or a refusal.

**Mint a new `investigation_id` for a new investigation**, with the date and time in it: `INV-dome-co2-20261006T0504`. The same name twice makes the second investigation inherit the first's run count, approved plan and any checkpoint left due, so a rerun of yesterday's question would start two-thirds through its work ceiling. The gateway refuses activation on an id that already carries work, and the refusal tells you to pick a new one.

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

`hotl_checkpoint(investigation_id, record, decision, verbatim, approved_plan)`. The record is the five parts **and every figure any specialist in the wave returned** — url, caption and the claim it supports. The gateway refuses a record that drops one: the final answer is written from the ledger, and a result several waves back is no longer in front of you, so a figure left out of the record is a figure the answer will not have. Put it in the ledger's Figures table in the same breath. `decision` is what the person said; `approved_plan` is the specialists they approved, **and only those will run**. An edit is binding: a specialist struck from the plan is refused by the gateway, so do not re-propose it in the same wave.

**A checkpoint record is filed after every wave in every mode**, and the next wave is refused until it is. Only the pause differs: in `every_wave` the record is filed **after** the person answers, carrying their words, and `automatic` is refused there; in `off` and `key_decisions` you file it with `decision: "automatic"` and carry straight on. That way an unattended run leaves exactly the same audit trail as a supervised one: the audit trail of an unattended run should be identical to a supervised one.

**`automatic` is only for those two modes.** In `every_wave` the gateway refuses it, because it would open the next wave without anyone being asked — the one thing this mode exists to prevent. There, every checkpoint carries a real decision (accept, edit, refuse or stop) and the person's own words. If a checkpoint is due and you have not put the summary to them yet, put it to them and end your turn; their reply is what you file.

## 3a. Each wave needs its own answer

The activation answer approves wave 1 and nothing else. Every wave after it is a question you put to the person and an answer they give to **that** question. The gateway refuses a decision whose words are the activation answer, or any reply already on file from an earlier checkpoint: a recycled quote is not an answer, and recycling one is how a supervised investigation quietly becomes an unsupervised one.

So the sequence at every wave boundary is: file nothing yet → **update the ledger, including its Pending question block** → put the five-part summary and the question through **`ask_user_question`** (section 0), with accept / refuse / stop as options and the free-text box carrying any edit → they answer in the same turn → file the checkpoint with their words and clear the pending block. Filing before they have answered is the mistake the refusal exists to catch.

**Do not end your turn to ask.** `ask_user_question` blocks and the turn survives it, so an answered checkpoint carries straight on and the whole investigation runs in one turn.

**If nobody answers, the turn is cut at its limit with no message.** That is the accepted cost of supervision: a supervised investigation is only as alive as its supervisor, which is the point of it. Nothing is lost provided the ledger was written first, which is why the pending block goes in before the question and not after. An investigation meant to run unattended is `off` mode, not `every_wave`.

## 3b. Key decisions only

The same machinery as every-wave, fired at four moments instead of at every boundary. **Every one of them is an `ask_user_question` call, answered in the same turn, exactly as in every-wave mode (section 0).** Never prose, never a turn ending, never a statement of what you intend followed by a stop.

| Trigger | When | What you ask |
|---|---|---|
| **The plan** | once, before the first wave | the waves you intend to run, the specialists in each, and what each should settle. Their answer approves it |
| **A change of scope or direction** | whenever the plan you had approved no longer fits what you found | what changed, what you now propose instead, and why |
| **The reduction wave** | when the risk specialist's plan has actionable items | the items, their owners and modes, and what each would settle. The gateway sees a published `reduction_plan` and refuses a wave filed as "no key decision here" that contains one |
| **A contradiction or a refusal** | when a specialist flags either | both sides of the contradiction, or what was refused and why, and what you propose to do. The gateway sees contradictions: a wave filed as "no key decision here" that contains one is refused |

The last two can fire **mid-wave**, not only at a boundary. Ask when you reach them; do not save them for the end of the wave, because by then you may have built on the thing in question.

**Between triggers, waves open without waiting.** File the record with `decision: "automatic"` — the gateway still requires it, so the audit trail matches a supervised run — and carry straight on. Do not ask at a wave boundary in this mode: the person chose not to be asked there, and asking anyway is the same failure as not asking in every-wave mode, from the other side.

**The plan trigger fires once.** Its answer approves the plan, not the whole investigation: a later change of scope is its own question.

## 3c. The confirmation line

Every `hotl_checkpoint` returns one, built from state you cannot set — the wave number, the specialists opening, the run count — and you surface it to the person **unchanged**. Each says which setting it acted under, so a wave that opened without asking cannot be mistaken for one that skipped them:

```
Wave 2: Approved as recorded — opening: structural_geology. 4 of 15 runs used.
        Your words on file: "yes, but skip the seismic rerun"
Wave 2: HOTL Selection — Key Decisions Only (no key decision here) — opening: structural_geology. 4 of 15 runs used.
Wave 2: HOTL Selection — Unsupervised — opening: subsurface_play. 4 of 15 runs used.
Wave 3: Work gate — 15 runs used, continuing to 30. This gate holds even unsupervised.
        Your words on file: "yes, keep going, raise it to 30"
Wave 3: Refused — opening nothing. 7 of 15 runs used.
Wave 3: Stopped at your request — nothing further opened. 7 of 15 runs used.
```

The second and third are the ones that matter most to read out. `automatic` is also the word the gateway refuses in every-wave mode, so a bare "Wave 2 automatic" reads, to someone who asked for key decisions, like you skipping them. The line says the setting instead.

**Two of the four triggers are checked, not trusted.** If any specialist in the wave returned a contradiction, or published a `reduction_plan`, the gateway refuses a record filed as "no key decision here" and tells you to ask. A change of scope and some refusals it cannot see, so those remain your judgement: if the plan you had approved no longer fits what you found, that is a trigger whether or not anything flags it.

## 4. Accept, refuse, edit

- **Accept** — the proposed plan becomes the approved plan.
- **Edit** — only what they approved runs.
- **Refuse** — ask whether they would like to propose an alternative or would rather you did. If they propose one, say what is sound about it, what is weak, and whether it can be done with the data in hand, before acting. If you propose, and that is refused, propose again; when your alternatives are exhausted, say so plainly, stop, and summarise what is established. **Refuse and stop** is theirs to choose at any point.

## 5. The work gate

**The ceiling is theirs, not yours.** It starts at 15 and the gateway refuses a different one unless the person's own words contain that number, in digits or spelled out. Do not decide that a long investigation deserves more rope: the work gate is the only gate that holds in every mode, so it is the one thing between an unattended run and unlimited spending, and choosing your own ceiling is choosing how much rope you get. If you think 15 is too low for the scope, say so when you ask the activation question and let them name a number — or let the gate fire at 15 and ask then, which costs one interruption.

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

When a checkpoint asks the person to choose between competing models, show them blind: neutral labels, randomised order, the same number of lines and the same figures each, evidence and test records first, and no option pre-chosen — in particular **no option carries `" (Recommended)"` here**, which is pre-selection in a thin disguise. With more than five alternatives, say how many there are and offer those that differ most, rather than dropping one silently to fit. Give your own preference only **after** they have answered, and if the models genuinely tie, say so and offer none. A confident recommendation is a strong anchor, and the point of the checkpoint is their judgement, not your agreement.

## 8. What the system cannot vouch for

Say it once, where it matters rather than everywhere: the gateway checks that a record matches the jobs it ran, but it has no channel to the user, so **the decision on file is what you reported**. That is why the confirmation line quotes their words back, and why you surface it unchanged. The same applies to who they are: a name and a role are recorded as given, never verified.
