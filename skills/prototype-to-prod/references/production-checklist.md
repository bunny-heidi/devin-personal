# Production checklist

Use this in Step 4. Each item either ships handled or becomes a named follow-up with an owner. Items the prototype mocked or skipped are the ones that bite.

## 1. Contracts

- Real endpoints and schemas, not mocked responses. Request/response types match the API contract.
- Error shapes: every failure returns something the UI can actually render (status, message, code), not a thrown mock.
- Idempotency: retries and double-clicks can't create duplicates or double-charge.
- Partial failure: if the operation touches multiple things, define what "half done" means and how the UI shows it.

## 2. Who

- Permissions: the right ability/role checks server-side, not just hidden UI (see `src/helpers/casl/ability.ts`).
- Tiers and trials: Free vs Pro vs Enterprise vs managed. Feature gated by plan? Upgrade path shown only where a user can actually buy it (not Mac App Store, not managed Enterprise).
- Entitlements: seat/org/user-level grants wired to the real `ENTITLEMENT` source.
- Existing users: people mid-flow or with old data on ship day get a defined behavior, not a crash or a blank state.

## 3. Where

- Regions: feature behaves correctly in every Heidi region; no cross-region data reads or writes.
- Platforms: web and desktop, current and older desktop builds (feature must degrade, not break, on a three-month-old app).
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
