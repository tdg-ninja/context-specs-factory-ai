# Code Review

PR review-cycle skills.

- **`address-feedback/`** — the responder. Triages an automated reviewer's findings on the
  feature PR (Clear / Ambiguous / Complex / Out-of-PRD-Scope), fixes-and-pushes the Clear ones
  diff-only, and replies to the rest. Headless; the dispatcher owns the round counter and STUCK
  escalation.

The reviewer half (self-hosted `claude-code-action` or managed Code Review, plus `REVIEW.md`)
is configured by `harness-init`, not shipped here.

**Convergence.** When the reviewer has no Important findings left, `REVIEW.md`
tells it to post a comment containing `HARNESS_REVIEW_CLEAN`. The dispatcher sees that
marker and hands the PR to the human for `/evaluate-pr` (Evaluate phase) — `address-feedback`
is unchanged by this; it only ever acts on the reviewer's findings.
