# Production checklist

Use this in Step 4. Each item either ships handled or becomes a named follow-up with an owner. Items the prototype mocked or skipped are the ones that bite.

## 1. Contracts

- Real endpoints and schemas, not mocked responses. Request/response types match the API contract.
- Error shapes: every failure returns something the UI can actually render (status, message, code), not a thrown mock.
- Idempotency: retries and double-clicks can't create duplicates or double-charge.
- Partial failure: if the operation touches multiple things, define what "half done" means and how the UI shows it.

## 2. Who — the audience matrix

Fill the matrix before building, using `references/heidi-checklist.md` §2–§5 for the full dimension detail. Each row is a dimension, each value either in scope (handled) or out of scope (defined behavior — hidden, upsell, no-op, or follow-up ticket). The dangerous bugs live in the *combinations*, so check at least the boundary cells: flag on × lowest in-scope tier, managed user × restricted role, trial ending mid-flow, oldest supported desktop build × feature on.

| Dimension | Values to consider | Typical v1 answer |
| --- | --- | --- |
| Feature flag | on, off, flag never evaluated (old client) | flag off → prior behavior, no residual UI |
| Plan tier (`SubscriptionTier`) | Free, Pro, Pro Plus, Clinician variants, Evidence, Enterprise | paid-only? what Free sees must be decided |
| Billing state | trialing, trial just ended, `PAST_DUE`, `CANCELED` | mid-flow transition handled, not stranded |
| Role | Admin, Clinician, Management, Support, View-Only | server-side ability check, not hidden UI |
| Account/team type | solo, team member/owner/admin, org global admin, managed/enterprise | managed users can't self-serve — what's their path? |
| Entitlement (`ENTITLEMENT`) | Evidence, Coder seat, Translator, Connectors, none | who grants it: org, team, or user |
| Platform | web, desktop (Mac/Win), Mac App Store build, managed desktop (3-month lag), on-prem, iOS/Android | each in scope or explicitly excluded with behavior |
| Region | AU, US, UK, EU, CA, CH, enterprise `-AZ1` | per-region differences or in-scope list |
| Clinical scope | touches clinical content (notes, transcripts, templates, EHR sync) or not | if yes → med-device review, per `heidi-checklist.md` §4 |

- Permissions: the right ability/role checks server-side, not just hidden UI (see `src/helpers/casl/ability.ts`).
- Upgrade paths only where a user can actually buy: not Mac App Store (`isAppleCompliantBuild()`), not managed Enterprise — their "buy" is contact-admin/contact-sales.
- Existing users: people mid-flow or with old data on ship day get a defined behavior, not a crash or a blank state.
- The classic misses: View-Only user reaches the feature via deep link; Support role sees admin-only UI; managed Enterprise member hits an upgrade wall they can't act on; trial expires between screen one and screen two; on-prem or managed-desktop users on an old build hit UI for a flag that can't evaluate.

## 3. Where

- Regions: feature behaves correctly in every in-scope Heidi region; no cross-region data reads or writes.
- Platforms: web and desktop, current and older desktop builds (feature must degrade, not break, on a three-month-old app). If mobile is out of scope for v1, say what a mobile user sees instead.
- Vendors/integrations: third-party dependencies degrade cleanly when down (EHR, Stripe, email, model providers).

## 4. States

- Loading, empty, error, success — all four, for every async surface.
- Data extremes: zero items, thousands of items, stale data after someone else edits.
- i18n and RTL; long strings; slow connection.
- Accessibility: keyboard path and screen reader labels on the new UI.

## 5. Data

- Migrations/backfills have a rollout order and a rollback story.
- Concurrency: two users (or two tabs) touching the same record — last-write-wins is a decision, not an accident.
- No patient or clinician data in logs, analytics or third parties — same rule as the spec stage.
- Sensitive fields stay in-region and respect existing data-residency guarantees.

## 6. Flag and rollout

- Feature flag defaults off; name and owner recorded in the PR.
- Rollout plan written in the PR description: internal → % → full, or per-tier/per-region order.
- Kill switch: flag off returns the product to prior behavior with no deploy.
- Desktop lag accounted for — flag on while an old build is still in the wild is safe.

## 7. Observability

- Metrics/events for the number the spec said should move — you can't prove it worked without it.
- Failure-state visibility: errors are logged somewhere an engineer can find them (no PII).
- Alerts or dashboards for the risky parts, proportional to blast radius.

## 8. PR hygiene

- Title/description link the spec; screenshots or recordings of the failure states, not just the happy path.
- Prototype PR closed unmerged and linked.
- No dead code, leftover `console.log`s, TODOs without tickets, or commented-out blocks.
- Changelog/release notes if user-facing; support docs flagged if the flow changed.
