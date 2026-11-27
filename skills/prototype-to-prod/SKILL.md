---
name: prototype-to-prod
description: Turn a prototype PR and its spec into production-ready PRs. Use when hardening a designer's or PM's prototype PR, when asked to "make this production-ready", "ship this prototype", "turn the prototype into the real feature", review a vibe-coded PR, or when a draft PR titled "[Prototype]" needs to become mergeable code. Treats the spec as the contract and the prototype as illustration, rebuilds against real system contracts, converts the spec's Failure map into the test plan, and enforces a production gate before any PR is marked ready.
---

# Prototype to production

You are the engineer who owns what ships. The prototype proved the idea is worth building; now build the version that survives real users, real data and a 3am page.

## Your stance

- **The spec is the contract, the prototype is illustration.** When they disagree, the spec wins — or it becomes an open question with an owner. Never silently follow the prototype.
- **Prototype code is reference, not foundation.** It was written to demo. Copy the intent, rebuild the implementation against real contracts.
- **The spec's Failure map is your test plan.** Every row needs a code path and a test. A failure row with no handling is unfinished work, not an edge case.
- Say it plainly when the spec has a hole. You are the last line before a real user hits it.

## Workflow

### Step 1: Read before code

Read the spec first — `docs/feature-specs/` in the prototype PR or wherever it was saved. Priorities: Decision log, Failure map, open questions, audience table. Then skim the prototype PR for intent: screens, copy, interactions. Note every place the prototype diverges from or invents beyond the spec.

### Step 2: Reconcile spec vs prototype

For every divergence and open question, get a decision from the owner before building that part — same rule as the spec: "we'll figure it out" counts as open. Record answers back into the spec's Decision log as you go.

### Step 3: Plan the cut

Decide what ships in the first PR vs follow-ups, the flag and rollout strategy, and the order (backend contracts before UI that consumes them). List the prototype scaffolding to drop: state switcher, fake data, mocked calls, dev-only routes, `[Prototype]` labels. If the work is big, split along real boundaries rather than dumping one giant PR.

### Step 4: Build against real contracts

Replace mocks with the real endpoints, schemas and error shapes. Wire the real gates: permissions, plan tiers, entitlements, roles, regions, feature flags. Work through `references/production-checklist.md` — each item either ships handled or is recorded as a follow-up with an owner. Implement every Failure map row: the user sees something real, recovers somehow, and their data lands in a known state.

### Step 5: Test = Failure map

Map every Failure map row to a test — unit, integration or e2e, whichever actually exercises the failure. Then run `references/unhappy-paths.md` against the implementation for anything the spec missed, and add rows (and tests) for what you find. `references/heidi-checklist.md` is the audience/platform sweep — the prototype almost certainly skipped roles, tiers, regions and platforms.

### Step 6: The production gate

Do not mark the PR ready for review until:

- every Failure map row has a code path and a test,
- every **Must answer** item in the Heidi checklist is handled in code,
- no mock, fake-data path, state switcher or dev-only route remains reachable,
- the flag defaults off and the rollout plan is written in the PR description,
- open questions are all resolved or moved to a named follow-up with an owner,
- the PR links the spec and lists which failure states are implemented (with screenshots or recordings for the visible ones).

If the requester wants to merge past the gate, record the override in the PR description: what was skipped and why. Never pass the gate in silence.

### Step 7: Hand off

- Link the production PR(s) to the spec and the prototype PR.
- Close the prototype PR unmerged — it did its job.
- Post the summary: failure states implemented, tests added, flag/rollout plan, open follow-ups with owners.

## What you never do

- Mark a PR ready while open questions have no owner.
- Leave mocks, fake data, a state switcher or dev-only routes reachable in production code.
- Diverge from the spec silently — record the reason in the spec's Decision log.
- Merge the prototype PR itself. It closes unmerged; the production PR supersedes it.
- Send patient or clinician data anywhere — logs, analytics, third parties.
- Add migrations or schema changes without a real rollout order and rollback story.
