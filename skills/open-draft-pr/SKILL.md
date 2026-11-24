---
name: open-draft-pr
description: Open a draft PR for the current branch and fill the description from the repo's .github/pull_request_template.md, inferring each section from the actual branch diff and commits. Use when the user says "open a draft PR", "raise a draft PR", "create a PR for this branch", "draft PR with the template", or similar.
triggers: [user]
---

# Open Draft PR

Open **draft** PR for current branch. Write body by filling repo's PR template from real branch diff — not placeholder text.

## Steps

### 1. Gather state (run in parallel)

```bash
git rev-parse --abbrev-ref HEAD                 # current branch
git status --porcelain                          # uncommitted work?
git log --oneline origin/HEAD..HEAD             # commits on this branch
git diff --stat $(git merge-base origin/HEAD HEAD)..HEAD   # files touched
gh repo view --json defaultBranchRef -q .defaultBranchRef.name  # base branch
```

- `origin/HEAD` unset → fall back to `main` (or ask).
- Uncommitted changes → tell user, ask whether to commit first. **Never commit without explicit consent.**

### 2. Read the template

Read `.github/pull_request_template.md` from repo root. Absent → fall back to minimal Summary / Changes / Testing body and say so.

### 3. Push the branch

```bash
git push -u origin HEAD
```

Skip if already tracking remote and up to date.

### 4. Fill the template from the diff

Infer every section from actual commits + diff. Replace **all** placeholders — no `[like this]` or `XXXX` left.

- **Ticket:** extract `SCR-XXXX` from branch name or commit messages; build Linear URL. None found → leave link, flag for user.
- **Goal / What Changed / Key Decisions:** derive from commits and diff, concise bullets.
- **Dependencies:** "None" unless diff/commits reference upstream PRs or infra.
- **Risk & Impact → On-Prem Impact:** repo has guardrails. Diff touches any of `src/utils/buildProfile.ts`, `isOnPrem`-gated providers, `tauri-config.js`, `src-tauri/tauri.conf*.json`, `src/utils/endpoints.ts`, `package.json` analytics deps, or `next.config*` → mark **Breaking/Additive**, reference `.ai-context/onprem-guardrails.md`. Else `N/A`.
- **Testing / Checklist:** leave checkboxes unchecked (human ticks them); pre-check only "self-review" if clearly done. Don't fake test claims.

### 5. Title

Format: `type(SCR-XXXX): brief description` (e.g. `feat(SCR-1234): add patient export`). Pick `type` from dominant change (`feat`/`fix`/`chore`/`refactor`/`docs`/`test`). Use ticket id from step 4.

### 6. Create the draft PR

Pass body via heredoc so formatting survives:

```bash
gh pr create --draft --base <default-branch> --title "<title>" --body "$(cat <<'EOF'
<filled template>
EOF
)"
```

Return PR URL.

## Rules

- **Draft always** — never open ready-for-review PR from this skill.
- Never `git commit` or `git push --force` without explicit consent.
- Don't invent a ticket number — can't find `SCR-XXXX` → leave placeholder, tell user.
- Don't tick testing/checklist boxes you can't verify.
- Fill from **whole branch** diff (merge-base..HEAD), not just last commit.
- PR already exists for branch → report its URL, don't duplicate (`gh pr view --json url,state`).
