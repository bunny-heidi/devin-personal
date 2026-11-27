# Prototype rules

The prototype exists to answer "is this the right thing?" Make it cheap to run and impossible to mistake for production code.

## Do

- Branch from `main`. Open a **draft** PR titled `[Prototype] <feature> - not for merge` and add the `prototype` label.
- Start the PR description with: "This is a prototype. It will not be merged. An engineer will build the feature from the spec linked below."
- Mock every backend call with fake data in the prototype's own files. Show the failure states from the checklist with mocked errors, not real ones.
- Gate the prototype behind a dev-only flag or route so it can't reach real users. 🚩 Engineering to confirm the preferred mechanism.
- Build with Heidi design system components (`@heidi/design-system`). Check `.ai-context/design-system/component-mappings.md` before using any native HTML element, and use semantic color tokens so dark mode works.
- Build a state switcher: a dev-only control that jumps the prototype to any row of the spec's Failure map (save failed, trial ended, offline, no permission, empty, huge data). Every Failure map row with a decided answer needs a state in the switcher. Reviewers should spend more time in the switcher than on the happy path.
- Label each switcher state with its Failure map step and row, so reviewers can match the screen to the spec.
- Keep the code in as few new files as you can, and don't refactor existing code.

## Don't

- Call real save, update or delete endpoints, or change what the backend stores.
- Add or edit backend endpoints, API routes (`pages/api/**`, `app/api/**`) or the generated schema in `packages/schema`.
- Add analytics events, Datadog logging or third-party scripts.
- Use real patient data, even from a test account. Use made-up names and notes.
- Edit translation JSON under `src/locales/`.
- Add a sidebar or navigation entry. That needs a designer's decision first.
- Mark the PR ready for review, or remove the draft status.

## If the repo checks fail

Don't fix unrelated lint, type or test failures, and don't bypass git hooks. Note the failure in the spec under "What the prototype faked" and move on. The prototype doesn't need to pass CI to answer its question.
