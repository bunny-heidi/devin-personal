---
name: rewrite-pr-description
description: Rewrite/refresh an existing PR's description to reflect new changes pushed since it was opened, re-deriving each template section from the current branch diff while preserving the human's intent, checked boxes, and ticket. Use when the user says "rewrite the PR", "update the PR description", "refresh the PR body", "the PR is stale", "regenerate the PR description", or after pushing more commits to an open PR.
triggers: [user]
---

# Rewrite PR Description

Refresh **existing** PR's body to match what branch now does. Re-derive sections from current full diff, but don't clobber human's deliberate edits (ticket, ticked boxes, hand-written rationale).

## Steps

### 1. Find the PR + current state (parallel)

```bash
gh pr view --json number,title,url,body,state,isDraft,baseRefName
git rev-parse --abbrev-ref HEAD
git log --oneline origin/HEAD..HEAD
git diff --stat $(git merge-base origin/HEAD HEAD)..HEAD
```

- No PR for branch → tell user; suggest `open-draft-pr` skill instead. Don't create one here.
- Ensure local pushed so diff matches PR:

```bash
git push
```

### 2. Diff the body against reality

Compare **existing PR body** to **current branch diff**:
- Which "What Changed" bullets now stale, missing, or wrong?
- New files trigger guardrails not flagged before (On-Prem, new deps)?
- Dominant change type shifted (e.g. `fix` → `feat`) — title may need updating too.

### 3. Preserve, don't clobber

Carry over from old body verbatim:
- **Ticket / Linear URL** — keep existing; never replace real `SCR-XXXX` with placeholder.
- **Checked boxes** (`[x]`) — keep checked. Never untick a box the human ticked.
- **Hand-written rationale** in Key Decisions / Goal — keep human's wording; only append genuinely new decisions.
- Before/After screenshots, Loom links, custom sections.

Only rewrite sections the new diff actually changes.

### 4. Re-derive the changed sections

Use `.github/pull_request_template.md` as structure of record (in case template itself changed). Re-fill from **whole branch** diff (merge-base..HEAD):
- **What Changed** — describe the **end state**: what the PR adds / what now exists. Not a commit-by-commit or inter-diff narration. Keep concise even on large diffs — end-state only, no "moved X→Y", no "refactored Z", no commit history.
- **Risk & Impact → On-Prem Impact** — re-check guardrail file list (`src/utils/buildProfile.ts`, `isOnPrem`-gated providers, `tauri-config.js`, `src-tauri/tauri.conf*.json`, `src/utils/endpoints.ts`, `package.json` analytics deps, `next.config*`), update Breaking/Additive/N/A.
- **Dependencies** — update if new upstream PRs/infra appear.

### 5. Apply the update

```bash
gh pr edit --body "$(cat <<'EOF'
<refreshed body>
EOF
)"
```

Change type shifted → also update title (keep existing `SCR-XXXX`):

```bash
gh pr edit --title "type(SCR-XXXX): brief description"
```

Return PR URL and one-line summary of what sections changed.

## Rules

- **Edit, never recreate** — use `gh pr edit`, not `gh pr create`. Don't close/reopen.
- Never untick a human-checked box or overwrite a real ticket id with placeholder.
- Don't invent test claims — leave unverifiable boxes as human left them.
- Diff from whole branch (merge-base..HEAD), not just latest push.
- Don't force-push or commit; this skill only rewrites PR body/title.
- Description = end state / what's added. Never narrate inter-commit diffs in the body.
- On large rewrites, show user short before/after diff of the description.
