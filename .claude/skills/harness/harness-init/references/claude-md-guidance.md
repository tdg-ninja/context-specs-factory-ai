# Generating CLAUDE.md

`CLAUDE.md` is the agent contract — loaded into every agent's context on every
session, in every worktree. It is what makes the harness portable: any
harness (Claude Code, a server runner, a future tool) reads it and learns the
project's onboarding, the Expert, the skill chain, the branch lifecycle, and the
non-bypassable verification layers.

harness-init generates it from `assets/CLAUDE.md.template` plus a codebase scan
(project-discovery.md). Most of the template is canonical and stays verbatim;
only the bracketed parts are filled from the scan.

## Keep it tight

Everything in `CLAUDE.md` is paid for in tokens on *every* invocation. Setup
mechanics, multi-developer coordination, and hackable seams belong in
`SETUP.md`, not here. If you're tempted to explain *how to set up* the harness
in `CLAUDE.md`, stop — that's `SETUP.md`'s job. `CLAUDE.md` tells an agent how
to *operate* in this repo, not how it was built.

## What to fill from the scan

| Bracket in template | Fill from |
|---------------------|-----------|
| `[Project Name]` | repo name / `package.json` `name` / README title |
| project config refs in the verification section | the linters/formatters/typecheckers discovered (project-discovery.md) — e.g. "Pre-commit hooks: eslint + prettier via husky" |
| any project-specific onboarding pointer | only if the project has a genuinely load-bearing doc an agent must read first |

## What stays verbatim (do not editorialize)

- The **Onboarding** list (Expert, PRDs, specs, designInvariants).
- The **Expert skill** section — its role as procedural memory, the "consult
  before non-trivial work" rule, "reflects what's committed to main."
- The **skill chain** table — the state machine. This is canonical; do not
  reorder or reword the trigger conditions.
- The **branch lifecycle** block.
- The **non-bypassable verification** section's four layers.

## The Expert caveat for fresh projects

The Expert (`.claude/skills/expert/`) does not exist on a brand-new project — it
is born from the first merge to main (the bootstrap case of `/expert-update`).
`CLAUDE.md` still references it, because by the time the harness completes its
first cycle the Expert will exist. Don't strip the Expert section just because
the directory isn't there yet — but DO tell the user, during setup, that the
Expert will be empty until their first feature merges, and that's expected.

## designInvariants.md

The template's onboarding list points at `designInvariants.md`. If the consumer
project doesn't have it yet, offer to copy the canonical one in (it's part of
what makes "the tiebreaker when prose and behavior disagree" real). If the user
declines, drop that line from the onboarding list rather than leaving a dangling
reference.

## Present it as a diff

Show the user the filled `CLAUDE.md` and point out exactly which lines you
changed from the template and why (the scan finding behind each). Let them edit
before you commit. This is their contract; they should own every line.
