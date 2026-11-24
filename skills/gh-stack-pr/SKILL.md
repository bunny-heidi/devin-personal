---
name: gh-stack-pr
description: Create and manage GitHub stacked PRs with the gh stack CLI extension (gh stack link, init, submit, view, merge). Use when the user asks for stacked PRs, gh stacking, stack a PR on another PR, or when a PR was created with --base pointing at another feature branch but is not in GitHub's stack UI.
triggers: [user]
---

# GitHub PR Stacking (`gh stack`)

Use **GitHub native stacks** via the `gh stack` extension — not manual `--base feature-branch` alone.

## Prerequisite

```bash
gh extension list | rg 'gh stack'
```

Missing → `gh extension install github/gh-stack`

## Real stack vs fake stack

| | Fake | Real |
|---|---|---|
| How | `gh pr create --base other-feature-branch` | `gh stack link` / `gh stack submit` |
| GitHub UI | Parent PR in base dropdown only | Stack panel, stack number, merge-as-stack |
| Fix | Run `gh stack link` on open PRs | Already correct |

**Rule:** After any PR targets another feature branch, run `gh stack link` (or `gh stack submit`) so GitHub registers the stack.

## Stack order

Arguments are **bottom → top** (trunk-adjacent first, dependents last):

```
main
└ PR #100  foundation
  └ PR #101  builds on #100
    └ PR #102  builds on #101
```

Merge order: bottom first. Top PRs rebase/update as lower layers land.

---

## Workflow A — Link existing open PRs (most common)

Use when PRs already exist but aren't in the stack UI.

```bash
# PR numbers or URLs, bottom to top
gh stack link 15828 15886

# Or branch names (auto-pushed, PRs looked up or created)
gh stack link feat/base feat/consumer
```

Optional local tracking:

```bash
gh stack init --base main feat/base feat/consumer
gh stack view --short
```

Return: stack number from link output, PR URLs, `gh stack view --short` tree.

---

## Workflow B — New stack from scratch

1. **Plan slices** — independent layers off trunk, or true dependency chain. Prefer [`split-to-prs`](../../.cursor/skills-cursor/split-to-prs/SKILL.md) for split planning.

2. **Init branches** (creates/adopts stack locally):

```bash
gh stack init --base main feat/layer-1 feat/layer-2
# Or single layer to start:
gh stack init --base main feat/layer-1
```

3. **Work on top branch** — commit on each layer before adding the next.

4. **Add another layer**:

```bash
gh stack add feat/layer-3   # branches from current top
```

5. **Push + create/update stack on GitHub**:

```bash
gh stack submit --auto        # non-interactive, draft titles
# Or interactive editor (default in TTY):
gh stack submit
```

6. **Fill PR bodies** — use [`open-draft-pr`](open-draft-pr/SKILL.md) patterns. Top PR **Dependencies** must cite parent PR numbers.

---

## Workflow C — Append to existing stack

```bash
# 15890 = existing stack number from GitHub UI / link output
gh stack link 15890 15899

# Or grow locally then submit
gh stack add feat/new-top
git commit ...
gh stack submit --auto
```

Numeric first arg = stack number when it matches an existing stack; otherwise treated as PR number.

---

## Verify

```bash
gh stack view
gh stack view --short
gh pr view <top-pr> --json baseRefName,url
```

Expect: each layer's base = parent branch; stack number in link/submit output; stack UI on GitHub.

---

## Merge

```bash
gh stack merge          # interactive
gh stack merge <pr>     # merge stack from a PR in the chain
```

Do not merge top PR alone via plain `gh pr merge` if user wants stack semantics — prefer `gh stack merge`.

---

## Sync / rebase after trunk moves

```bash
gh stack sync
gh stack rebase
```

---

## Agent rules

- **Never** treat `--base feature-branch` as done — always `gh stack link` or `gh stack submit`.
- **Never** `git commit`, `git push`, or `git push --force` without explicit user consent.
- **Bottom-to-top** order in every `link` / `init` argument list.
- One concern per layer — same guidance as split-to-prs.
- PR already in a stack → `gh stack view` / `gh pr view`; don't duplicate.
- After linking, tell user stack number + merge order (bottom first).
- Complements `open-draft-pr` (single PR body) and `split-to-prs` (how to slice work).

## Examples

**Fix manual stack (this session's pattern):**

```bash
gh stack link 15828 15886
gh stack init --base main \
  feature/gro-3825-cancellation-reason-enum-subscription-scoped-persistence \
  feature/retire-user-redeemed-discounts-on-billing-account
gh stack view --short
```

**Fresh two-layer stack:**

```bash
gh stack init --base main feat/auth feat/api
# ... commits on each branch ...
gh stack submit --auto
```
