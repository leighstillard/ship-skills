---
name: design-check
description: Visual and UX review pass for any change that touched UI — frontend components, HTML, CSS, templates, dashboards, CLI output formatting, or anything a human looks at. Use whenever a diff includes UI files, when the user asks for a design review, or as part of /ship. If the diff touched nothing user-visible, state that and exit immediately.
---

# Design Check

## Step 0: Applicability
Inspect the diff. If nothing user-visible changed (no frontend files, templates, styles, CLI output, emails, rendered docs), say "No UI surface touched — design check not applicable" and stop. Do not invent work.

## Step 1: Render it
You cannot review design from source code alone.
- Web UI → run the dev server and screenshot the changed views (Playwright/puppeteer if headless). Capture at least one narrow viewport (~380px) and one desktop width.
- CLI output → run the command and capture actual terminal output, including a case with long values and a case with empty results.
- Generated documents/emails → render to the final format and look at it.

## Step 2: Review against this checklist
**Consistency**
- Matches the existing design system / component library of the repo. New one-off styles need justification.
- Spacing, type scale, and colour come from existing tokens or variables, not hardcoded values.

**States**
- Loading, empty, error, and overflow states all exist and look intentional.
- Long text: does it truncate, wrap, or break the layout?
- Interactive elements have hover/focus/disabled states.

**Accessibility**
- Sufficient colour contrast; information not conveyed by colour alone.
- Focus order and keyboard operability for anything interactive.
- Images/icons that convey meaning have accessible labels.

**Taste**
- Avoid AI-default clichés: gratuitous purple gradients, Inter-by-default, emoji-as-decoration, cards inside cards.
- Alignment is deliberate. Nothing is "almost" aligned.
- Density appropriate to the audience (data tools can be dense; marketing pages breathe).

## Step 3: Fix or flag
Fix mechanical issues directly (a missing empty state, hardcoded colour, broken responsive layout). Flag judgement calls for the user with a screenshot and a one-line recommendation each — don't redesign unilaterally.

## Step 4: Verdict
- **PASS** — with screenshots as evidence.
- **PASS WITH NOTES** — list flagged judgement calls.
- **FAIL** — list blocking issues; loop back after fixes.

## Gotchas
- The most common failure is skipping rendering and reviewing the JSX/CSS as text. Render first, always.
- Check the diff's *effect on neighbours*: a width change in a shared component breaks pages you didn't touch.
- Dark mode: if the repo supports it, screenshot both modes.
- If a `frontend-design` skill is installed, defer to it for aesthetic standards and use this skill for the process/checklist.
