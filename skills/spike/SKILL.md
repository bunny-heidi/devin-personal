---
name: spike
description: >-
  Run a time-boxed technical spike — clarify the decision, probe minimally,
  document tradeoffs, and deliver a recommendation. Use when the user asks for
  a spike, feasibility check, proof-of-concept, investigation, de-risk,
  "investigate whether we can", or invokes /spike or /timeboxed-spike.
triggers: [user]
---

# Spike

A spike **reduces uncertainty so the team can decide** — not ship production code.

**Deliverable:** decision doc with recommendation, tradeoffs, confidence, effort estimate, and next steps.  
**Code:** throwaway POC only. Polish, tests, and full error handling are out of scope unless they answer the spike question.

---

## Spike vs other modes

| Mode | Goal | When |
|------|------|------|
| **Explore** | Think, clarify problem, no commitment | Question still fuzzy |
| **Spike** | Answer a specific question with evidence | Decision blocked on unknown |
| **Propose / RFC** | Full change proposal + implementation plan | Spike says proceed |
| **Implement** | Ship production code | Proposal approved |

If the user wants to **build the feature**, stop — spike is wrong mode.  
If the question is still fuzzy, explore first, then spike.

---

## Non-negotiables

1. **Timebox fixed up front.** Ask if not given ("1 day"? "half a day"?). Default 0.5–2 days. At end of box, name the approach — even if answer is "not feasible yet, here's the blocker."
2. **Frame around crisp decision questions, not a topic.** "Can we use X for Y, or must we build it?" beats "look into Y." Vague topic → convert to 3–6 answerable questions and confirm.
3. **Validate the riskiest assumption first.** Rank by (impact if wrong × uncertainty). Spend box top-down. Don't polish easy parts.
4. **Measure, don't hand-wave.** Comparing approaches? Run head-to-head on small hard-case set. Report numbers (accuracy, latency, cost). "Probably fine" is not a finding; "P=1.0/R=0.95, ~2ms vs ~1000ms" is.
5. **Reuse before build.** Inventory existing services, hooks, patterns, licences. Most spikes end with "assemble existing parts." List reused pieces explicitly.
6. **Throwaway code labeled throwaway.** One place (branch/PR/`scripts/`), named clearly, marked disposable in write-up.

---

## Workflow

Copy this checklist and track progress:

```
Spike progress:
- [ ] 1. Frame — decision questions, success criteria, timebox, guardrails, out of scope
- [ ] 2. Inventory — what already exists to reuse
- [ ] 3. Rank assumptions — attack riskiest first
- [ ] 4. Probe — smallest experiment that produces evidence
- [ ] 5. Conclude — pass/fail criteria, recommendation, effort, risks
- [ ] 6. Hand off — doc + ticket comment / PR summary
```

### 1. Frame (before writing code)

Ask or infer:

1. **Decision needed** — what choice does this unblock? (approach A vs B, feasible yes/no, effort estimate)
2. **Decision questions** — 3–6, each answerable yes/no or with a number
3. **Success criteria** — 3–5 checkboxes, each pass/fail observable
4. **Time box** — fixed end date or duration
5. **Guardrails** — hard constraints answer must respect (e.g. "must not increase latency")
6. **Out of scope** — what spike will NOT do
7. **Stakeholder** — who reads the conclusion?

Bad framing: "explore Playwright"  
Good framing: "Can auth fixtures reach parity with the old runner in 2 days? Recommend proceed/pivot/kill."

If framing is weak, write the brief first (see [spike-brief-template.md](spike-brief-template.md)) and confirm with user before investigating.

### 2. Inventory existing infrastructure

Before proposing new code:

- Grep/read codebase for services, hooks, patterns, licences that already do part of this
- Write "Existing infrastructure (reuse, don't rebuild)" list — each item: what, where, what you'd reuse
- Read docs / APIs / existing code **before** writing new code

### 3. Rank assumptions by risk

- List assumptions the approach depends on
- Order by "how bad if wrong" × "how unsure am I"
- Attack top assumption first in the probe

### 4. Probe (investigate)

Pick **smallest vertical slice** that answers the question:

- One POC path, not three polished alternatives
- Reuse repo patterns — don't invent parallel infrastructure
- Document dead ends; they are spike value

Allowed:
- Read-only codebase search
- Throwaway branch / scratch files
- Minimal POC (happy path only)
- Manual smoke test proving or disproving hypothesis
- Head-to-head eval with raw outputs and numbers

Not allowed (unless user explicitly widens scope):
- Production-quality error handling
- Full test coverage
- i18n, analytics, observability, feature-flag cleanup
- Refactors unrelated to the question

If blocked (bad creds, missing licence) — flag as finding, keep going on rest of box.

Log findings as you go — bullets, not prose essays.

### 5. Conclude

Every spike ends with:

1. **Success criteria table** — each criterion: ✅ pass / ❌ fail / ⚠️ partial
2. **Findings by question** — one-line verdict + 1–3 evidence bullets per question
3. **Options considered** — 2–3 rows: pros, cons, rough effort
4. **Recommendation** — proceed / pivot / kill — one clear sentence + tradeoff named
5. **Verdict** — GREEN / AMBER / RED to build + any unblock needed
6. **Confidence** — high / medium / low + what would change your mind
7. **Remaining risks** — what production work must still validate
8. **Rough effort estimate** — assumptions stated, long poles named
9. **Proposed PR breakdown** — if approved, ordered list with gating deps
10. **Next steps if approved** — ordered list, not full task breakdown

Park descoped ideas as explicit fast-follows; don't silently drop them.

### 6. Hand off

Put conclusion where team will find it:

| Context | Where |
|---------|-------|
| Ticketed work | Comment on spike ticket (Linear, Jira, GitHub issue, etc.) |
| Spec / design work | Conclusion section in spec or design doc |
| UX / architecture spike | Design-decisions doc capturing *why*, not just *what* |
| Quick team spike | PR description or linked doc in ticket |

---

## Repo conventions (if applicable)

Check project for existing spike patterns before starting:

- **Branch:** `spike/<ticket>-<short-name>`
- **Ticket:** link spike issue; post conclusion back on ticket
- **Specs:** explicit success criteria + revert path if spike touches infra
- **Revertibility:** if spike adds infra, document how to tear it down quickly
- **Do not** merge spike POC as production without follow-up proposal + tasks

---

## Anti-patterns

- Scope creep — "while I'm here…"
- Unbounded exploration — reading forever with no question to close
- Gold-plating the POC
- Hand-waved comparison — "X is probably better" without numbers
- Building before checking reuse — proposing new infra without inventory
- Ending with "here's what I learned" and **no recommendation**
- Burying the answer — findings dump with no decision
- Hiding uncertainty behind "works on my machine"
- Skipping success criteria — makes conclusion unverifiable
- Blowing the box on one blocker — flag it, move on
- Turning spike into full implementation without explicit approval

---

## Templates

### Brief (before investigation)

For one-pager before probe starts, use [spike-brief-template.md](spike-brief-template.md).

### Full write-up (at conclusion)

Put in tracking doc (Linear/Notion/PR). Adapt headings, keep shape: questions → what exists → answers → decision → risks → cost → build plan → artifacts.

```markdown
## Spike conclusion: [name]

**Recommendation:** [Proceed with X / Pivot to Y / Do not pursue]
**Confidence:** [high | medium | low]
**Verdict:** [GREEN / AMBER / RED to build]

## Questions to answer
1. **[Dimension]** — [crisp decision question]? [any guardrail].
2. ...

## Success criteria
| Criterion | Result | Notes |
|-----------|--------|-------|
| ... | ✅ / ❌ / ⚠️ | ... |

## Existing infrastructure (reuse, don't rebuild)
- **[Thing]** — `path/to/it`. [what it is]; [what we reuse].

## Findings (by question)
### 1. [Question] — [one-line verdict]
- [evidence bullet, with numbers where relevant]

## Options
| Option | Pros | Cons | Effort |
|--------|------|------|--------|
| A | | | |
| B | | | |

## Recommendation
**[Decisive statement.]** [Why, tradeoff named.] [Descoped items as fast-follows.]

## Risks / Open questions
- **[Risk]** — [what's still unknown / what could bite].

## Rough effort estimate
[State assumptions. Name long poles.]

| Approach | Estimated effort | Confidence |
| -- | -- | -- |
| [recommended] | [BE/FE days, parallel] | Medium |
| [alt] | [delta] | Low-Med |

## Proposed PR breakdown (if approved)
1. **[PR]** — [scope]. [gating dependency if any].

## Code / artifacts
**Throwaway spike code (not production):** [link]
- `script.py` — [what it proves].

## Next steps (if approved)
1. ...
```
