# Code Review

PR review-cycle skills (Flow 2.5).

- **`address-feedback/`** — the responder. Triages an automated reviewer's findings on the
  feature PR (Clear / Ambiguous / Complex / Out-of-PRD-Scope), fixes-and-pushes the Clear ones
  diff-only, and replies to the rest. Headless; the dispatcher owns the round counter and STUCK
  escalation.

The reviewer half (self-hosted `claude-code-action` or managed Code Review, plus `REVIEW.md`)
is configured by `harness-init`, not shipped here.
