# <Feature name>

- **Requester:** <name>
- **Date:** <yyyy-mm-dd>
- **Prototype PR:** <link> (draft, not for merge)
- **Status:** Draft / Ready for engineering

## Summary

Two or three sentences: who has the problem, what we're changing, and why now.

## Problem

Describe the user, what they do today, and what goes wrong for them.

## Goals and non-goals

**Goals**
- <goal>

**Out of scope for v1**
- <thing we're choosing not to handle>

## How we'll know it worked

The number we expect to move, and by roughly how much.

## Happy path

1. <user action or system event>
2. <next step>

## Failure map

The core of the spec. One row per failure, grouped by happy-path step. Every step needs at least one row, or a note saying why nothing can go wrong there.

| Step | What goes wrong | What the user sees | How they recover | What happens to their data | Status |
| --- | --- | --- | --- | --- | --- |
| 4 | Save fails | "We couldn't save that. Try again." with a retry button | Retry | Nothing saved | Decided |
| 4 | Trial ended before the save | <behavior> | <recovery> | <data> | Open (owner) |

## Pre-mortem

The top complaints we expect three months after launch, and what the spec does about each.

- <complaint>: <covered by Failure map row, or open question>

## Personas

Where the flow broke for each persona, and the Failure map row that covers it.

| Persona | Where it broke | Row |
| --- | --- | --- |
| Managed enterprise clinician, Windows desktop, 3 months behind, UK | | |
| Pro trial user, trial ends mid-flow | | |
| Team member removed or downgraded mid-flow | | |
| Clinician mid-recording | | |
| Arabic UI on a slow connection | | |

## Who sees it

One row per audience that behaves differently. Mark a row "Hidden" when that group doesn't see the feature.

| Account type | Role | Tier / trial state | Region | Platform | What they see |
| --- | --- | --- | --- | --- | --- |
| Individual | n/a | Free | All | Web, desktop | <behavior> |
| Team member | Clinician | Pro trial | AU, UK | Web | <behavior> |
| Managed enterprise | Any | Enterprise | All | Desktop | Hidden |

## Scenarios

Each scenario gets an ID so the engineer can turn it into a test. Write at least two unhappy-path scenarios for every happy-path one, drawn from the Failure map.

**Happy paths**
- **S1.** Given <situation>, when <action>, then <result>.

**Unhappy paths**
- **S2.** Given <situation>, when <the save fails>, then <result>.

## Heidi checks

Record the answer for each section of the checklist: `Decided`, `Out of scope for v1`, `Not applicable (reason)`, or `Open question (owner)`.

| Area | Answer | Notes |
| --- | --- | --- |
| Account types and roles | | |
| Tier, trial and entitlements | | |
| Existing users | | |
| Region and data residency | | |
| Languages and right-to-left | | |
| Medical device scope | | |
| Patient data | | |
| Web / desktop | | |
| Mac App Store (billing) | | |
| On-prem | | |
| Managed desktop | | |
| Mobile parity | | |
| Interruption tier (T0 to T5) | | |
| Failure and edge states | | |
| Rollout and flag | | |
| Measurement | | |

## Copy

Draft text for every customer-facing string, marked for `microcopy` review.

| Where | Draft text |
| --- | --- |
| <button, error, empty state> | <text> |

## Rollout

- **Flag and first audience:**
- **Without the flag, users see:**
- **If it breaks, we switch off:**
- **Data already created when we switch off:**

## What the prototype faked

Everything the prototype mocked, hardcoded or skipped. The engineer builds these for real.

- <for example: team list is hardcoded; save always succeeds after 1 second>

## Open questions

| Question | Owner | Needed by |
| --- | --- | --- |
| | | |

## Decision log

Overrides and judgment calls, including any skipped gate items and the reason.

- <yyyy-mm-dd>: <decision> (<who>)
