# Production checklist

Use this in Step 4. Each item either ships handled or becomes a named follow-up with an owner. Items the prototype mocked or skipped are the ones that bite.

## 1. Contracts

- Real endpoints and schemas, not mocked responses. Request/response types match the API contract.
- Error shapes: every failure returns something the UI can actually render (status, message, code), not a thrown mock.
- Idempotency: retries and double-clicks can't create duplicates or double-charge.
- Partial failure: if the operation touches multiple things, define what "half done" means and how the UI shows it.

## 2. Who — the scope of release

Answer it in a couple of sentences, not a matrix: **who does v1 ship to, and what does everyone else see?** It looks different for every feature. Example:

> "v1 ships to PLG clinician and practice tiers, flag-gated, web and desktop, AU and US first. Free tier sees the upsell. Managed and on-prem are out — feature hidden, follow-up ticket for the managed path. Mobile is a separate PR."

Check the statement covers:

- the motion + tier: PLG (free / clinician / practice / legacy), SLG, `is_managed`
- role, platform, region, flag state where they matter
- for every out-of-scope segment, a defined behavior — hidden, upsell (only where they can actually buy), no-op, or a named follow-up

Classic misses: a managed user hits an upgrade wall they can't act on; a free-tier deep-link lands in a paid feature; trial expires mid-flow; a legacy-tier user the spec forgot exists; an old managed-desktop build gets UI for a flag it can't evaluate.

If the feature touches clinical content, med-device review applies (see `references/heidi-checklist.md` §4).

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
