---
name: explain-like-im-new
description: Explain any term, concept, error, or piece of code as if the user has never encountered it before — zero assumed background. Use when the user says "ELI5", "explain like I'm 5", "I have no idea what that means", "what even is X", "explain it simply", "I've never seen this before", or otherwise signals they're lost or new to a topic.
triggers: [user]
---

# Explain Like I'm New

User want concept from absolute zero. Assume **no prior knowledge** — no untaught jargon, no "as you know", no skipped steps. Build understanding ground up.

## Core rules

1. **Assume zero background.** Term needed to explain thing? Define that term first. Never use jargon word before introduced.
2. **One idea at a time.** Short sentences. Build up, don't dump.
3. **Analogy first, then the real thing.** Anchor new concept to everyday thing user knows (mail, kitchens, libraries, traffic, lego). Then map analogy back to real concept piece by piece.
4. **Concrete over abstract.** Tiny specific example, not general definition.
5. **No hand-waving.** Tempted to say "it just works" or "magic happens here"? Stop, explain actual mechanism simply.
6. **Name the thing precisely at the end.** After they get it, give correct technical term so they can search/talk later.

## Structure to follow

Use this shape (adapt length to concept):

1. **One-liner** — what it is, plain words, no jargon. (1 sentence)
2. **Analogy** — familiar real-world thing it's like.
3. **How it actually works** — real concept, mapped back to analogy, step by step.
4. **Tiny example** — smallest concrete instance (code snippet, real value, scenario).
5. **Why you'd care** — when/why it matters in practice.
6. **The proper name(s)** — exact terminology + close cousins, for lookup.
7. **Check** — one quick question or "does that part land before we go deeper?" so they can stop you.

## Calibration

- **Default depth: curious beginner.** Smart person, just new to *this*. Not literally a 5-year-old unless asked.
- They say "more ELI5" / "still lost" → simpler analogy, shorter sentences, strip all remaining jargon.
- They say "ok I get the gist, go deeper" / "I'm not that new" → skip analogy, raise precision, add edge cases.
- Always offer next rung: "Want the slightly more technical version?"

## Do not

- Don't lecture. Stop early, let them ask.
- Don't stack three analogies — pick single best.
- Don't define a term using harder terms.
- Don't apologize for simplifying. Just simplify.

## Example

**User:** "What even is a 'race condition'? No idea."

**Good response shape:**

> **One-liner:** Two things happening at the same time, where the result depends on which one finishes first — and you can't reliably predict the winner.
>
> **Analogy:** Two people editing the same shared shopping list at once. Both read "milk, eggs". You add "bread", they add "butter", you both save. Whoever saves last wipes out the other's change. The list is now wrong, and *which* item survives is luck.
>
> **How it works:** A computer often does many tasks in overlapping slices. If two tasks read and then write the same piece of data, the final value depends on the exact timing — and timing changes run to run. So the bug shows up sometimes and vanishes when you look for it.
>
> **Tiny example:** Counter starts at 0. Two tasks both read 0, both add 1, both write 1. You expected 2, you got 1.
>
> **Why you care:** These bugs are rare, random, and brutal to reproduce — the #1 reason "it works on my machine" but breaks in production.
>
> **Proper name:** "race condition" (cousins: "data race", "concurrency bug").
>
> Want me to show how a "lock" fixes this?
