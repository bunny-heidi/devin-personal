---
name: cleanup-comments
description: Clean up verbose, redundant, or unnecessary comments in the current PR diff. Especially strip Linear ticket IDs and strip every ponytail mention (label goes, constraint stays). Use when the user asks to clean up comments, tidy docstrings, trim comments before opening a PR, or invokes /cleanup-comments.
triggers: [user]
---

# Cleanup Comments

Trim and prune comments in the **current PR diff only**. Never touch lines outside the diff.

## Workflow

1. Get the diff against the base branch:

```bash
git diff "$(git merge-base HEAD origin/main 2>/dev/null || git merge-base HEAD main)"...HEAD
```

(Fall back to `git diff` for uncommitted changes if there is no branch diff yet.)

2. **First pass — tickets + ponytail.** Grep the diff for ticket IDs (`GRO-`, `ARC-`, `PLA-`, `MONEY-`, `DX-`, `SCR-`, etc.) and for **`ponytail` in any form** — case-insensitive, with or without the colon, inline comment or docstring prose:

```bash
rg -ni 'ponytail|GRO-|ARC-|PLA-|MONEY-|DX-|SCR-' $(git diff --name-only "$(git merge-base HEAD origin/main)"...HEAD)
```

Fix those before anything else.

3. For every other added/changed comment in the diff, apply the rules below.
4. Edit only the comment lines. Do not change code logic.
5. Re-read the touched files to confirm comments still read cleanly.
6. **Verify pass.** Re-run the step-2 grep. Zero ticket IDs and zero `ponytail` hits, or the cleanup is not done. Say so explicitly when reporting back.

## Rules

- **Examine every comment** in the diff. For each, ask: can this be expressed in fewer tokens? Strip articles, filler, and hedging — almost caveman style. Keep technical terms, symbols, and meaning exact; cut only fluff. `// This is the function that handles the user login flow` → `// handles login flow`.
- **Delete** comments that restate what the code already says (self-explanatory code needs no comment). E.g. `// increment counter` above `counter++`.
- **Shorten** lengthy comments / docstrings to 1–3 lines max. Keep only the non-obvious intent, trade-off, or constraint.
- **Remove Linear (or any) ticket references** everywhere — including inside `ponytail:`, `TODO`, `FIXME`, and docstrings. Patterns like `GRO-1234`, `ARC-161`, `(see PLA-2496)`, `MONEY-296`. Drop the ID; keep any surrounding explanation if it stands alone. Ticket lives in the branch / PR, not the code.
- **Ponytail mentions — strip the label, keep the constraint.** The `ponytail:` marker is authoring scaffolding; it does not ship in a PR. Delete the word `ponytail` wherever it appears (inline comment, docstring, any casing, colon or not), then keep the shortcut's substance — *what was deferred* + *ceiling* — as plain prose in the nearest docstring, collapsed to one line. Drop ticket IDs, hot-path laundry lists, and "delete in GRO-X" wording. If the shortcut is self-evident from the code, drop the line entirely instead of rehoming it.
- **Keep** only essential detail: the *why*, not the *what*. Preserve license headers, API contract docs, and `TODO` / `FIXME` markers (minus any ticket IDs in them).

## Examples

Before:
```ts
// This function takes a user ID and fetches the user from the database.
// It was added as part of GRO-4521 to support the new profile page.
// Returns null if the user is not found.
async function getUser(id: string) { ... }
```

After:
```ts
// Returns null if the user is not found.
async function getUser(id: string) { ... }
```

Before:
```ts
// Loop over the items array and push each item id into the ids array
items.forEach((item) => ids.push(item.id));
```

After (comment deleted — code is self-explanatory):
```ts
items.forEach((item) => ids.push(item.id));
```

Before (`ponytail:` + ticket):
```python
"""...

ponytail: shadow phase computes both paths, so the served request pays the
non-served path's cost too (an extra entitlement read + a duplicate team
gate) on the GET /user + chat-completion hot paths. Bounded to the shadow
window — deleted with the legacy branch (GRO-3647). Upgrade path if it
bites: sample the shadow, or gate it behind its own flag.
"""
```

After (label gone, ceiling kept as plain prose):
```python
"""...

Shadow runs both paths: extra entitlement read + team gate per request.
"""
```

Before (inline `ponytail:` above the code it qualifies):
```python
    """Best-effort TEAM evidence upsert; non-managed keep zero rows."""
    # ponytail: demotion managed→not leaves stale deny. Upgrade: revoke on demotion if parity flags it.
    if not self._team_needs_explicit_evidence_entitlement(...):
```

After (rehomed into the docstring, one line):
```python
    """Best-effort TEAM evidence upsert; non-managed keep zero rows.

    Demotion managed→not leaves a stale deny row (never revoked here).
    """
    if not self._team_needs_explicit_evidence_entitlement(...):
```
