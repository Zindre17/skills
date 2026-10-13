---
name: refine
description: A relentless interview that stress-tests a plan, design, task or decision before any of it gets built, asked in rounds of up to four questions per AskUserQuestion form with the full reasoning carried inside the form itself, writing the glossary entries and ADRs as the decisions land. Use this whenever the user says refine, grill me, grill this, grill with docs, stress-test, pressure-test, poke holes, interrogate, sharpen, or asks to be interviewed or questioned about an idea before implementing it. Also use before starting any task the user calls unrefined, half-baked, or vague, and whenever they hand over a plan and ask what is wrong with it.
---

Interview the user relentlessly until you reach a shared understanding. Map the work as a
**design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are
already settled: the questions you can ask _now_ without guessing at answers you have not
heard yet. A question whose answer depends on another question still open belongs to a
_later_ round. Each round of answers reshapes the tree and pushes the frontier outward.

The session is done when the frontier is empty: every branch visited, nothing left silently
assumed.

## First, load the domain model discipline

Before composing anything, invoke the `domain-modeling` skill with the Skill tool — it ships in
a plugin, so in the available-skills list it is usually named
`mattpocock-skills:domain-modeling`; use whichever form is listed. Then read
the project's `CONTEXT.md` (or `CONTEXT-MAP.md`) and its existing ADRs, because those are the
decisions you are forbidden to re-litigate and the words you are required to use.

If neither form is in the available-skills list, stop before the first round and tell the user
that /refine needs the `mattpocock-skills` plugin:
`/plugin install mattpocock-skills@claude-plugins-official`. Do not reconstruct its rules from
memory; a half-remembered glossary discipline is the fuzzy language it exists to prevent.

It has to be loaded _first_, not offered at the end, because its work is inseparable from
composing a question. A question written in fuzzy language produces a fuzzy answer that both
of you will later remember differently, and the glossary is what stops that. Loading it
afterwards would mean tidying up vocabulary that had already done its damage.

That skill will also have you write `CONTEXT.md` entries **the moment a term crystallises**,
rather than in a lump at the end. Keep that habit here, and extend it to ADRs: the answer to a
question often _is_ the ADR, and writing it while the reasoning is still in front of you is
the difference between a real record and a reconstruction.

Where the two skills meet, these rules decide:

- **An ADR that meets the bar gets written, not offered.** `domain-modeling` says to offer
  one; here, a decision that is hard to reverse, surprising without context and the result of
  a real trade-off gets its ADR as soon as it is settled, without spending a question on
  permission. The bar itself is unchanged. Only the sweep at the end offers instead of writing.

- **A challenge to the user's language becomes a question, not a remark.** `domain-modeling`
  says to call out a conflicting term immediately; here, immediately means _in the form_ —
  either as its own question with the two candidate meanings as options, or in the `question`
  field of the next one. It never becomes a line printed above the form.
- **Writing files is not printing.** `CONTEXT.md`, ADRs and the transcript all get written
  mid-session; none of that goes on screen. Mention the paths once, at the end.

## The question goes in the form, not on the screen

Ask through `AskUserQuestion`, **one round at a time**: every question on the frontier, each
carrying everything needed to answer it, four to a form (the tool rejects more). Print
nothing above it — no preamble listing what you are about to ask, no numbered round, no "see
the options below".

The reason is the user's experience of the session: they are reading and thinking, and the
worst thing you can do to them is split one question across two places so they have to
scroll up to answer what is in front of them. A question is well-formed here when the screen
showing it contains everything needed to decide, and nothing above it matters.

This costs you the chance to compose a beautiful markdown round. Take the trade. The round
still gets written down — into the transcript, after the fact — so nothing is lost except
the scrolling.

### Ask the whole round at once

Compose the whole frontier, then send it: four questions to a call, since the tool rejects a
fifth (`maxItems: 4`). The form shows them one at a time, each on its own screen, so batching
costs the user nothing in readability; what it saves is the gap between questions. A short
wait before a round is fine — long enough to look away — but a string of few-second pauses
between single questions is too short to do anything with and too long to feel smooth. Pay
the thinking cost once per round, not once per question.

The price is that an answer can no longer retire a question the user has not seen yet. So
only batch questions that are genuinely independent: if one answer could make another question
moot or change its options, the dependent one waits for the next round. When more than four
independent questions are ready, compose every one of them before sending anything, then send
the first four and the rest in the next call straight after, with no thinking, writing or
lookups between the two forms: they are one round split across forms, and the gap between them
should be the tool's round trip and nothing else. Put the questions whose answers unlock the
most in the first form. Recompute the frontier after the whole round.

Between rounds, do the bookkeeping — transcript, `CONTEXT.md`, ADRs — and any lookups the next
round needs before sending it, so the user sees one pause per round rather than several.

### Budget the three channels

The form looks narrow because most of its room is in fields that are easy to leave thin.
Spend them:

- **`header`** — `Q3 Exit`, `Q1 Undo`. Number plus one or two words. Twelve characters is a
  hard limit, so count them: `Q5 Checkpoint` is already too long, `Q5 Exit test` fits.
- **`question`** — the tension, in **at most 50 words**. Not "which undo model?" but why the
  obvious answer is not obviously right. This is the only field the user cannot avoid reading,
  so it carries the stakes — and it is also the field that pushes the options off the screen,
  so every clause has to earn its line.
- **`label`** — a few words. A name for the option, not an explanation of it.
- **`description`** — **one line, twelve words or fewer.** What choosing this commits them to,
  compressed to a clause. Four options each given two sentences here costs twenty lines before
  the panel even starts, which is how a question stops fitting on a screen.
- **`preview`** — the real estate, and where all the prose belongs. A markdown panel,
  monospace, shown beside the options and **re-rendered as the user arrows between them**.

**Each question must fit one screen without scrolling, because that is the entire point
of asking this way.** In a batch every question gets its own screen, so the budget is per
question, not per round. Budget it: assume about 24 usable lines. Question ≤ 6 lines, four
options at ≤ 2 lines each, preview ≤ 16 lines. If it does not fit, the fix is never to shrink
the preview — cut an option, or split the question in two.

A preview is read on its own, with the other options' previews invisible, so write each one
self-contained. A good one is roughly:

```
WHAT THIS MEANS

Two or three lines of concrete consequence -- and where the options
differ in something showable, show it: the event payload, the schema
line, the directory tree, the two versions of the sentence.

WHY I'D PICK THIS  (or: WHY I WOULDN'T)

The actual argument. Name what it costs, not just what it buys, and
say which other option becomes right if a stated assumption is wrong.
```

Keep lines under about 52 characters and the whole panel under 16 lines — it is a panel, not a
document, and a monospace block that wraps is harder to read than a short one that does not.

### Constraints worth knowing before you compose

- **Previews are single-select only.** So do not use `multiSelect` here; the previews are
  worth more than the ability to tick two boxes. When choices genuinely compose, either make
  the combination its own option ("both, declared wins") or split it into two questions.
- **Four options maximum.** A fifth means either the weakest one belongs in a preview as
  "also considered, and why not", or the question is really two questions.
- **Never write your own "Other" or "None".** The form always offers Other with free text, so
  writing one wastes a quarter of the space.
- **Recommended option first**, with `(Recommended)` on the end of its label, so accepting
  your recommendation is one keystroke. Recommend something on every question: a question
  you cannot answer yourself is usually a question you have not thought hard enough about,
  and the user can always overrule you.

### When the answer is thinner than the question

The free-text "Other" and any note attached to a choice are the highest-signal thing you will
get all session — they are the user telling you the question was wrong. Read them as edits to
the tree, not as footnotes to an answer.

If a load-bearing answer comes back bare and you are not certain what it commits you to, put
the read-back in a `question` field of the _next_ round — "taking Q1 to mean the log shows
cancelled events but the counters ignore them" — rather than spending a whole question on
it. If an answer contradicts something already settled, say so and re-open the earlier
decision; the tree is allowed to change shape, a quietly inconsistent design is not. When you
re-open a decision you have already written up, go and fix the ADR or glossary entry in the
same breath — writing the docs as you go means a reversal can strand a document that states
the opposite of what you now believe, and a confidently wrong ADR is worse than no ADR.

## Finding facts is your job, never the user's

When a frontier question needs a fact from the environment — what the code already does, what
a library supports, what the data actually looks like — go and find it. Dispatch sub-agents in
parallel at the start of the session and before each round. A running exploration is just an
unsettled prerequisite: if it will land within a minute or so, wait for it and send a fuller
round; if it will take longer, send the questions that don't depend on it now and hold the
downstream ones for the next round.

Never ask the user something you could have looked up. It spends their attention on retrieval,
and their attention is the scarce thing in this session. The _decisions_ are theirs.

## The transcript

Keep a running transcript file, appended after each round's answers arrive. Write it where
the repo already keeps this kind of thing — `docs/adr/`, `docs/decisions/`, `tasks/` — and
otherwise
`docs/refinements/<YYYY-MM-DD>-<slug>.md`. Pick without asking; mention the path once, at the
end.

Write each entry **from the call you actually made**: the question text, the options with
their previews, then the chosen option and any free text, verbatim. Rendering the transcript
from the payload rather than composing it separately is what keeps the record and the
question identical — two separate authorings of the same question drift within a sentence or
two of each other, and then the record is quietly wrong.

This file is why nothing is lost by keeping the screen clean. It also survives the context
being compacted out from under you mid-session, and most of the value of a grilling is in the
alternatives that were rejected and the reason they lost — precisely what a list of final
answers throws away.

## If the form is unavailable

In a non-interactive session `AskUserQuestion` may not exist. Then, and only then, print the
round as markdown: numbered questions, full bodies, options spelled out, your recommendation
and its reason under each. Say once that you are falling back to text.

## Ending

When the frontier is empty, write the settled design: every decision, the answer, one line of
why, and — separately and explicitly — what was deliberately left open and what would trigger
deciding it. An honest list of open questions is worth more than a document pretending the
tree was finished.

By this point the glossary entries and ADRs should already exist, written as each decision
crystallised. So the ending is not a handoff, it is a sweep: name the files you wrote, and
check the settled list for anything that met `domain-modeling`'s bar for an ADR — hard to
reverse, surprising without context, the result of a real trade-off — and did not get one.
Offer those; do not write them unasked. A decision that fails the bar does not need a document
just because it was hard-won.

Do not start implementing until the user confirms you have reached a shared understanding.
The point of the session is the understanding, and acting early is how you find out you did
not have it.
