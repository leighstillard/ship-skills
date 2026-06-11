---
name: ship
description: Take a finished change from local diff to merged PR. Use whenever the user says "ship it", "/ship", "get this merged", or has a completed change ready to go out. Drives the change to a goal state through non-negotiable quality gates.
---

# Ship

## Goal
The change is merged to the default branch, and every invariant below holds with evidence. You decide the path; the invariants are not yours to waive.

## Invariants
These are preconditions, not steps. If any is unmet, achieving it becomes your next action.

1. **Simplified before verified.** A `/simplify` pass has run over the final shape of the diff. Code changes after simplify don't require re-simplifying unless substantial.
2. **No PR exists without a VERIFIED verdict.** `/verify` has ended in VERIFIED, with concrete evidence (commands + output, end-to-end execution). NOT VERIFIED, partial, or "tests pass" does not satisfy this. Any code change after a VERIFIED verdict invalidates it — re-run `/verify` before relying on it again.
3. **No PR exists without a design verdict if the diff is user-visible.** `/design-check` has passed or correctly declared itself not applicable.
4. **The PR carries its evidence.** Opened via `/pr`, with the verify evidence (and design screenshots, if any) in the body.
5. **No merge without independent review addressed.** `/codex:review` has run against the open PR and every finding is fixed, or rebutted with reasoning on the thread. Non-trivial fixes re-trigger invariant 2.
6. **No merge while red or contested.** CI green, review comments addressed, conflicts resolved — `/babysit` owns this until merge completes.

## Operating rules
- The natural path is 1→6, but reordering is yours when it serves the goal (e.g. a babysit CI fix loops you back through invariants 2 and 5 — do that without being told).
- Report state transitions tersely: `[ship] verify: VERIFIED → opening PR`.
- Maintain a checklist of invariant status (TodoList if available) so an interrupted session resumes correctly.
- Escalate, don't waive: if an invariant can't be met without a decision above your pay grade (large refactor needed, broken main, disputed review, missing credentials), pause and ask the user. Declaring the goal met with an unmet invariant is the one prohibited move.
