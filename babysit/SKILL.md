---
name: babysit
description: Shepherd an open PR to merge. Watches CI, fixes failures, addresses review comments, resolves merge conflicts, and enables auto-merge when green. Use after opening a PR, when the user says "babysit", "watch the PR", "get this merged", or as the final step of /ship.
---

# Babysit

Goal: the PR merges, or you produce a precise statement of what's blocking it and why you stopped.

## Loop

Poll with `gh pr checks <number>` and `gh pr view <number> --json state,reviewDecision,mergeable,comments`. Between polls, wait (`sleep 60–120`) rather than spinning.

On each iteration, handle whichever applies:

### CI failed
1. `gh run view <run-id> --log-failed` — read the actual failure, not the job name.
2. Classify:
   - **Real failure caused by this diff** → fix it, commit, push. Re-verify locally first if the fix is non-trivial.
   - **Flaky** (known flaky test, infra timeout, rate limit) → retry once: `gh run rerun <run-id> --failed`. If it flakes twice, treat as real or escalate to the user — do not retry in a loop.
   - **Broken on main** (failure reproduces on the base branch) → report to user; do not "fix" main's problem inside this PR.
3. **Loop guard**: if the same check has failed 3 times after 3 distinct fix attempts, STOP and summarise the attempts for the user. Repeated retry loops burn tokens and hide a real problem.

### Review comments arrived
- Address substantive comments with code changes; reply on the thread explaining what changed.
- For comments you disagree with, reply with reasoning — don't silently ignore, don't silently capitulate on things that matter. Flag genuine disputes to the user rather than arguing past two exchanges.
- Nitpicks (naming, style): just do them unless they conflict with repo convention.
- Never resolve a reviewer's thread yourself unless the repo convention is author-resolves.

### Merge conflict
- `git fetch && git rebase origin/<default-branch>` (or merge if the repo's convention is merge commits — check history).
- Resolve conservatively: when in doubt about intent on the other side of the conflict, read the conflicting commit's message and diff before choosing.
- Re-run the test suite after any conflict resolution before pushing.

### All green + approved
- Enable auto-merge per repo convention: `gh pr merge <number> --auto --squash` (check whether the repo squashes, merges, or rebases — look at recent merged PRs).
- Confirm merge completed; report the merge commit.

## Exit conditions
Stop and report (do not keep polling indefinitely) when:
- Merged ✅
- Blocked on human review with no comments to action
- Loop guard tripped
- A failure needs a decision you shouldn't make alone (API change, data migration, security-relevant)

## Gotchas
- Pushing a fix dismisses stale approvals in many repos — batch comment fixes into one push where possible.
- `gh pr checks` exit code is non-zero while checks are pending, not just on failure — read the output, don't branch on exit code.
- After a rebase, force-push with `--force-with-lease`, never bare `--force`.
- If required checks include external systems (deploy previews, security scans) that you cannot retry via `gh`, report rather than waiting forever; note the check name.
