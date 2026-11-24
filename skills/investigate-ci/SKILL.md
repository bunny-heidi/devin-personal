---
name: investigate-ci
description: >-
  Triage failing CI on a PR: attribute the failure to this PR or not, then fix,
  rebase, or do nothing. Use when the user pastes CI logs, reports a red check,
  asks why CI failed, or wants help investigating a PR / GitHub Actions failure.
---

# Investigate CI

Triage a CI failure before changing anything. Prefer attribution over guessing.

## Decision tree (must follow)

1. **Related to this PR?** → fix it.
2. **Unrelated, and main already fixed it?** → rebase onto the PR base.
3. **Unrelated, and not yet fixed on main?** → do nothing. Report and stop.

Never "helpfully" patch main's problems on this branch. Never open a side PR
unless the user asks.

## Workflow

Copy and track:

```
CI Triage:
- [ ] 1. Capture failure
- [ ] 2. Attribute: related to this PR?
- [ ] 3. Branch:
        related     → fix
        unrelated + fixed on main → rebase
        unrelated + not fixed on main → stop
- [ ] 4. Report outcome
```

### 1. Capture failure

Resolve the PR and the failing check:

```bash
gh pr view --json number,title,baseRefName,headRefName,statusCheckRollup
gh pr checks
```

Pull the failed job log (prefer the specific check the user named):

```bash
gh run list --branch "$(git branch --show-current)" --limit 5
gh run view <run-id> --log-failed
```

If the user pasted logs, use those — still confirm which check/job it is.

Ignore noise unless it is the actual exit cause: missing optional env
interpolation warnings, unrelated package warnings, advisory notes that did
not fail the step.

### 2. Attribute: is it related to this PR?

Compare the failure to this branch's diff vs the PR base (default `main`):

```bash
BASE=$(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || echo main)
git fetch origin "$BASE"
git diff --name-only "origin/$BASE"...HEAD
```

**Related** when any of these hold:

- The failing file/path is in the PR diff (or imported only by changed files).
- The error cites a symbol, string, message id, test, or config this PR added
  or edited.
- Reproducing locally on this branch fails the same way, and the same command
  passes on `origin/$BASE` (or the failure clearly needs this branch's change).

**Unrelated** when:

- The failing path/symbol is untouched by the PR and not a downstream of it.
- Same check is also red on `origin/$BASE` / recent main runs.
- Infra/flake: runner, registry auth, quota, network, flaky test with no
  connection to the diff.
- Monorepo lint/typecheck of a package this PR did not touch, with an error
  inside that package's pre-existing code.

When unsure, gather one more signal (main check status, or
`git log -S'<symbol>' origin/$BASE -- <file>`) — then decide. Do not fix on a
coin flip.

### 3a. Related → fix

- Reproduce the failing command locally when cheap (`pnpm`/`task`/`nx` as the
  repo already uses — never invent a different package manager).
- Fix the root cause in the PR's code. Smallest diff that makes the check green.
- Do **not** commit or push unless the user asked. Leave the fix in the working
  tree and summarize what changed.
- If the user asked to fix CI end-to-end, commit with a message that names the
  check/root cause, then push the feature branch (no force unless a rebase
  happened — then `--force-with-lease` only).

### 3b. Unrelated + fixed on main → rebase

Confirm main (or the PR base) already has the fix:

```bash
# same check on base
gh run list --branch "$BASE" --limit 5
# or the fixing commit / path on base
git log "origin/$BASE" --oneline -20 -- <failing-path>
git show "origin/$BASE:<failing-path>"   # when applicable
```

If base is green for that failure (or clearly contains the fix), rebase:

- Follow the `rebase-on-base` skill.
- Prefer rebase over merge.
- After rebase: `--force-with-lease` only if the user wants the branch updated
  remotely; otherwise stop after local rebase and report.

### 3c. Unrelated + not fixed on main → do nothing

Stop. Do not edit files, do not rebase, do not commit.

Tell the user:

- which check failed
- why it is **not** caused by this PR (1–2 concrete signals)
- that main does **not** appear to have a fix yet
- that you are leaving it alone per triage policy

Optional one-liner they can take elsewhere: link/run id + error headline.

## Report format

Keep it short:

```markdown
**Verdict:** related → fixed | unrelated → rebased onto <base> | unrelated → no action

- Check: <name>
- Cause: <one line>
- Evidence: <path / symbol / main status>
- Action taken: <what you did, or "none">
```

## Hard rules

- Attribution before mutation.
- Unrelated + unfixed on main = **zero** code changes.
- Never bypass hooks (`--no-verify`, `--no-gpg-sign`).
- Never force-push `main`/`master`/the base branch.
- Do not conflate with `babysitting-pr` (ongoing monitor/fix loop) — this
  skill is a single triage pass driven by the decision tree above.
