---
name: rebase-on-base
description: >-
  Rebase the current branch onto its PR base branch (or main/default) and
  resolve merge conflicts. Use when the user asks to rebase on main, update
  from base, fix merge conflicts, make a PR mergeable, or sync with the
  target branch.
triggers: [user]
---

# Rebase on Base

Rebase the current branch onto the PR's **actual base branch** (not always
`main`), resolve conflicts, and force-push safely.

Prefer **rebase** over merge unless the user explicitly asks to merge.

## Hard rules

- Never `git rebase -i` / `--interactive` (no TTY).
- Never `--no-verify` / `--no-gpg-sign`.
- Never `git push --force`. Use `git push --force-with-lease` only after a
  successful rebase, and only of the **current feature branch** — never
  `main`/`master`/the base branch.
- Never discard the user's uncommitted work. Stash or stop.
- If conflict resolution is ambiguous (both sides look intentional, product
  decision, or unclear intent), **stop and ask** — do not guess.
- After resolving, leave the tree buildable: no leftover conflict markers.

## Workflow

Copy and track:

```
Rebase Progress:
- [ ] 1. Preconditions
- [ ] 2. Resolve base branch
- [ ] 3. Fetch
- [ ] 4. Rebase
- [ ] 5. Resolve conflicts (loop)
- [ ] 6. Verify
- [ ] 7. Push
- [ ] 8. Report
```

### 1. Preconditions (run in parallel)

```bash
git rev-parse --abbrev-ref HEAD
git status --porcelain
git branch -vv
gh pr view --json number,url,baseRefName,headRefName,mergeable,mergeStateStatus 2>/dev/null || true
```

- Detached HEAD or on `main`/`master`/the base itself → **stop**. Tell the user.
- Dirty worktree:
  - If the user said to include/stash changes → `git stash push -u -m "rebase-on-base"` and remember to pop after.
  - Otherwise → **stop** and ask (commit, stash, or discard). Do not proceed dirty.
- No open PR is fine — fall through to base resolution below.

### 2. Resolve base branch

Priority order:

1. User-named branch (`rebase onto develop`) → use that.
2. Open PR for this branch → `baseRefName` from `gh pr view`.
3. Else repo default:

   ```bash
   gh repo view --json defaultBranchRef -q .defaultBranchRef.name
   ```

4. Else `main`.

Call this `$BASE`. Confirm briefly to the user before rebasing if it is **not**
`main`/`master` (e.g. `develop`, release branch).

### 3. Fetch

```bash
git fetch origin "$BASE"
```

Also fetch the PR head remote if this is a fork PR being updated locally
(`git fetch origin` is enough when head lives on `origin`).

### 4. Rebase

```bash
git rebase "origin/$BASE"
```

- Clean success → skip to **Verify**.
- Conflicts → continue to conflict loop.
- Other failure → `git rebase --abort` if rebase is in progress, report the
  error, stop.

### 5. Conflict loop

While rebase is stopped on conflicts:

```bash
git status
# unmerged paths listed as "both modified" / "deleted by us" / etc.
```

For **each** conflicted file:

1. Read the file (and `git show :2:path` / `git show :3:path` when markers are
   messy). In a rebase: `:2` = onto (base), `:3` = our commit being replayed.
2. Resolve by combining intent:
   - Keep **both** when independent (imports, unrelated hunks).
   - Prefer the **feature-branch** change when the conflict is the PR's
     purpose.
   - Prefer **base** when base deleted/renamed a file the branch still edits —
     port the branch's intent onto the new location/API if possible.
   - Generated / lockfile / schema dumps → regenerate when a project command
     exists (`pnpm i`, `pnpm run schema:dev`, etc.); do not hand-merge blobs.
3. Strip all `<<<<<<<` / `=======` / `>>>>>>>` markers.
4. Stage: `git add <path>` (or `git rm` if the correct resolution is delete).

Then:

```bash
git rebase --continue
```

- Editor opens for the commit message → non-interactive:

  ```bash
  GIT_EDITOR=true git rebase --continue
  ```

- More conflicts → repeat this section.
- Want out → `git rebase --abort`, restore stash if any, stop.

**Do not** `git rebase --skip` unless the commit is empty **and** that is
clearly correct (e.g. change already on base). Prefer asking.

### 6. Verify

```bash
git status
# must be clean, not mid-rebase
rg -n '^(<<<<<<<|=======|>>>>>>>)' . || true   # must find nothing in tracked code
```

If the repo has a cheap sanity check for touched packages, run it (typecheck /
unit test scoped to changed files). Fix breakages caused by the resolution
before pushing.

If you stashed in step 1: `git stash pop` and resolve any stash conflicts the
same way.

### 7. Push

History rewrote → needs force-with-lease:

```bash
git push --force-with-lease
```

If the branch has no upstream yet:

```bash
git push -u --force-with-lease origin HEAD
```

`--force-with-lease` rejected → someone else pushed; **stop**, fetch, and ask
the user. Do not escalate to `--force`.

### 8. Report

Tell the user:

- Base used (`$BASE`)
- Whether rebase was clean or how many conflicted files
- Notable resolution choices (one line each)
- Push result + PR URL if any (`gh pr view --json url -q .url`)

## Conflict heuristics (short)

| Situation | Default |
|-----------|---------|
| Import / include list both added lines | Keep both, dedupe |
| Same line, feature vs formatting-only on base | Keep feature intent |
| File deleted on base, edited on branch | Port edit to replacement path; else ask |
| `package-lock` / `pnpm-lock.yaml` | Re-install to regenerate; don't hand-merge |
| Snapshots / generated schema | Prefer regenerate via project script |
| Truly unclear product conflict | Ask user |

## Abort / recovery

```bash
git rebase --abort          # during rebase
git stash pop               # if you stashed
```

Mid-rebase and user says stop → abort, do not leave the repo in a conflicted state.

## Out of scope

- Merging the PR
- Interactive rebase / squash / edit commits (unless user separately asks)
- Updating someone else's branch without a local checkout
