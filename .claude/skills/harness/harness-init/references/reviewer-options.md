# Reviewer options (Flow 2.5)

The PR review cycle is OPTIONAL but real. When set up, an automated reviewer
flags issues on the feature PR and `/address-feedback` responds, with a
deterministic round cap so the loop can't run away. Without it, every comment
lands on a human and the harness just opens PRs and waits for merge.

Present the three paths below, explain the tradeoffs, and let the user choose.
If they choose a reviewer, you create the artifacts for that path. If they
decline, the harness still works — it just has no automated review step.

## The three paths

### A. Self-hosted via `claude-code-action` (recommended default)

A GitHub Actions workflow running on the team's own API key. Posts findings on
PR open and on every push.

- **Setup:** drop `assets/workflows/claude-review.yml` into
  `.github/workflows/`, drop `REVIEW.md` at repo root, add an
  `ANTHROPIC_API_KEY` repo secret.
- **Cost:** often well under $1/PR with a small model for the first pass +
  Expert as context-narrowing + prompt caching on stable parts.
- **Requires:** a GitHub remote; ability to set a repo secret.
- **Best for:** most projects.

### B. Anthropic managed Code Review

Install the GitHub App, configure Review Behavior per-repo.

- **Setup:** install the app + configure in GitHub; drop `REVIEW.md` (the
  managed service consumes the same file).
- **Cost:** ~$15-25/PR; multi-agent parallel review with a verification step.
- **Best for:** teams that want it turnkey and have the budget.
- **Note:** at this cost, the round cap (default 5) can mean ~$125 worst case —
  mention the cost-cap follow-up (design Open Gap) if the user picks this.

### C. None (v1 build/test/PR loop only)

No reviewer, no `REVIEW.md`. The harness implements, passes the PRD runner,
opens a PR, and stops for a human. Perfectly valid starting point; the reviewer
can be added later with one workflow file and no other changes.

## What's shared across A and B

- **`REVIEW.md`** (canonical, `assets/REVIEW.md`) — the convergence rules. Both
  self-hosted and managed read it. Key rule: after the first review, suppress
  new nits and post Important findings only. This is the primary convergence
  mechanism; the round counter is just the safety net.
- **The responder** — `/address-feedback` triages each comment into Clear /
  Ambiguous / Complex / Out-of-PRD-Scope. The dispatcher increments the round
  counter before *every* invocation regardless of bucket, so a reply-only stall
  marches to STUCK — the correct escalation, since only a push converges a PR.
  The dispatcher enforces the cap, not the agent.
- **The reviewer is non-conversational.** It posts findings; the implementer
  signals back via commits, not comment replies. Comment threads are
  human↔human.

## Migration note

A and B share the same trigger surface (PR open, push, `@claude review`) and the
same `REVIEW.md`. Switching between them is one workflow-file change, nothing
else. So starting at C and moving to A later is cheap — reassure the user that
declining now isn't a one-way door.

## Heterogeneous reviewers (upgrade path, not a v1 option)

Some teams run CodeRabbit (style/conventions lane) + Claude (logic lane) in
parallel. They don't oscillate because they flag different classes of issue.
Worth mentioning only if the user already pays for CodeRabbit and asks. Do NOT
run two Claude reviewers in the same lane — that oscillates.

## What you generate per choice

| Choice | Files created |
|--------|---------------|
| A | `.github/workflows/claude-review.yml` + `REVIEW.md` + note to add `ANTHROPIC_API_KEY` secret |
| B | `REVIEW.md` + instructions to install the GitHub App |
| C | nothing |
