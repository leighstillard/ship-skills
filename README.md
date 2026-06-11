# ship-skills

A set of five **agent skills** that take a finished code change from a local diff all the way to a merged pull request — through quality gates that don't get waived under pressure.

They're written for [Claude Code](https://claude.com/claude-code) and any agent that supports the `SKILL.md` format (a folder with a `name`/`description` frontmatter), but the workflow they encode is tool-agnostic.

## The skills

| Skill | What it does |
|-------|--------------|
| **ship** | The orchestrator. Drives a completed change to *merged on the default branch* through a set of non-negotiable invariants (simplified → verified → design-checked → PR'd → reviewed → green). Decides the path; won't declare done with an unmet gate. |
| **verify** | Proves the change actually works. Core principle: **tests passing ≠ feature working**. Drives the change end-to-end against the real entry point, sweeps edge cases, and ends in a hard `VERIFIED` / `NOT VERIFIED` verdict backed by command output. |
| **design-check** | Visual/UX review for anything a human looks at (UI, CLI output, emails, rendered docs). Renders the change (not the source), reviews against a consistency/states/accessibility/taste checklist, and exits immediately if no user-visible surface was touched. |
| **pr** | Commit, push, open the PR. Branch hygiene, deliberate staging, a quality commit message, and a PR body that carries the verify evidence. |
| **babysit** | Shepherds the open PR to merge. Watches CI, reads real failure logs, fixes or classifies (real / flaky / broken-on-main), handles review comments and merge conflicts, and enables auto-merge when green — with a loop guard so it doesn't burn tokens retrying forever. |

`ship` is the entry point; the other four are the gates it runs (and each is independently useful on its own).

## Independent review & your inference providers

One of `ship`'s gates is a **second-model review**: before merge it runs `/codex:review` (the [Codex](https://github.com/openai/codex) plugin) against the open PR and requires every finding to be fixed or rebutted on the thread. The point is to get a *different* model's eyes on the change, not the one that wrote it — a cheap, effective check against blind spots.

That gate is best-effort: if the Codex plugin isn't installed, `ship` proceeds without it (and you lose the independent-review safety net). If you want it, install the Codex CLI + its Claude Code plugin so `/codex:review` resolves.

More generally, these skills get sharper the more inference options you give your agent. **Ask your agent what review tools and providers it actually has available — and how to make the best of them.** For example:

> What inference providers and code-review tools do I have wired up (Codex, other model CLIs, MCP reviewers, CI bots)? Given those, how should the `ship` skills' verify and review gates use them to get a genuinely independent second opinion on my changes?

The skills are written to *use* an independent reviewer when one exists; pointing them at the strongest one you have access to is where most of the value is.

## Installation

These are folders containing a `SKILL.md`. Installing them means dropping each folder into your agent's skills directory.

### Ask your agent to do it (easiest)

Paste this to your coding agent:

> Clone `https://github.com/leighstillard/ship-skills` and install the five skills into my personal skills directory: copy each top-level skill folder (`ship`, `verify`, `design-check`, `pr`, `babysit`) into `~/.claude/skills/`. Don't overwrite anything without telling me first.

For a **single project** instead of globally, ask it to copy the folders into `<project>/.claude/skills/` instead.

### Do it yourself

```bash
git clone https://github.com/leighstillard/ship-skills
cd ship-skills

# Personal (all projects):
mkdir -p ~/.claude/skills
cp -r ship verify design-check pr babysit ~/.claude/skills/

# — or — per-project:
mkdir -p /path/to/project/.claude/skills
cp -r ship verify design-check pr babysit /path/to/project/.claude/skills/
```

Restart your agent / start a new session so it picks up the new skills.

## Usage

Once installed, invoke them by name. In Claude Code:

```
/ship          # the whole pipeline: verify → design-check → pr → babysit → merged
/verify        # just prove it works
/design-check  # just the UX/visual pass
/pr            # just commit + push + open PR
/babysit       # just shepherd an open PR to merge
```

Or just say what you want in plain language — e.g. *"ship it"*, *"is this actually working?"*, *"watch the PR and get it merged"* — and the matching skill's `description` will trigger it.

## License

MIT — use, fork, and adapt freely.
