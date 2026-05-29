# Review instructions

## Re-review convergence
After the first review, suppress new Nits and post Important findings only.
Do not re-flag findings already addressed in a subsequent push — let the
auto-resolve mechanism handle them.

## What Important means here
Reserve 🔴 Important for findings that would break behavior, leak data, or
introduce a regression against the PRD's definition of done. Style and
naming are 🟡 Nit at most.

## PRD-aware
The PRD lives at `prds/<feature>/prd.md`; the runnable definition of done is
`prds/<feature>/run-prd-test.sh`. If a finding is outside the PRD's stated
scope, mark it 🟣 Pre-existing — the responder will reply with a follow-up
PRD recommendation rather than implement it.

## Cap the nits
Report at most three Nits per review. If you found more, summarize as
"plus N similar items" rather than posting them all.

## Don't re-flag what the deterministic gate owns
`scripts/local-checks.sh` (lint, format, typecheck, fast tests, custom lints) runs
before this review. Don't spend findings on lint/format/type issues or anything
those checks own — they're the computational floor; your job is the semantic/logic
review they can't do. **But** if a check was reached green by *silencing* — a new
suppression directive (`@ts-ignore`, `eslint-disable`, `as`/`!`, …), a weakened
config, or a skipped/deleted test — flag it 🔴 Important. A green gate reached by
silencing is a regression in disguise.

## Non-conversational
Replying to an inline comment does not prompt you to respond or update the PR.
The implementer acts on findings by pushing commits, not by replying. Comment
threads are for human-to-human discussion.
