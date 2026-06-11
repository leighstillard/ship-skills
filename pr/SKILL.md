---
name: pr
description: Commit, push, and open a pull request for the current change. Use when the user says "open a PR", "ship this", "commit and push", or as the PR step of /ship. Handles branch hygiene, commit message quality, and PR description.
---

# PR

## Step 1: Pre-flight
- `git status` and `git diff` — review everything that would be committed. Confirm no secrets, no debug prints, no unrelated files, no `.env`, no large binaries.
- Confirm you are NOT on the default branch. If you are, create a branch: short, kebab-case, scoped (`fix-yamaha-diagram-dedupe`, not `leigh-changes-3`).
- `git pull --rebase origin <default-branch>` on the feature branch if it's stale; resolve conflicts before committing more work on top.

## Step 2: Commit
- Stage deliberately (`git add <paths>`), not `git add -A`, unless you have verified every untracked file belongs.
- Commit message: imperative summary line ≤72 chars stating *what and why*, body only if the why isn't obvious from the diff. Follow the repo's existing convention (check `git log --oneline -15` — if the repo uses conventional commits, use them).
- One logical change per commit where practical. Don't fragment artificially.

## Step 3: Push and open PR
```bash
git push -u origin <branch>
gh pr create --title "<summary>" --body "<body>"
```

PR body structure (adapt to any repo PR template — check `.github/PULL_REQUEST_TEMPLATE.md` first):
- **What** — one or two sentences.
- **Why** — link the issue/ticket (Linear ID if one exists in context).
- **How verified** — paste the evidence from the verify step: commands run, behaviours confirmed. This section is mandatory; "tests pass" alone is insufficient.
- **Risk / rollback** — anything reviewers should watch; how to revert if it goes wrong.
- Screenshots if the design-check step produced any.

## Step 4: Post-create
- Apply labels and reviewers per repo convention if discoverable.
- Output the PR URL.

## Gotchas
- `gh` must be authenticated; if `gh auth status` fails, stop and tell the user rather than fiddling with credentials.
- Never force-push to a branch with existing review activity without explicit instruction.
- If the diff is large (>~400 lines changed), say so and ask whether to split before opening — large PRs rot in review.
- Do not amend or rewrite commits that are already pushed unless asked.
- If CI config changed in this diff, call it out explicitly in the PR body — reviewers consistently miss it.
