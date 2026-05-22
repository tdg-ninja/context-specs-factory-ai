# The dispatcher, explained

When you drop `assets/poll-and-dispatch.sh` into the project, walk the user
through it using this file. The goal: the user understands every section well
enough to decide whether to tweak it. Never present it as a black box.

## What it is

`scripts/poll-and-dispatch.sh` — ~150 lines of bash, `git`, and `gh`. No LLM in
the decision path. It runs once per tick. The `if/elif` chain is the state
machine; the artifacts on disk are the state.

## The six load-bearing properties

Explain these as the "why it's safe" of the script. Each maps to an invariant.

1. **One transition per tick per branch.** The `if/elif` fires at most one
   branch. Next tick, the artifact this tick wrote satisfies the condition and
   the *next* `elif` fires. Forward, one step at a time. (Inv 5)
2. **Artifacts are the only state.** A sentinel existing IS that phase being
   done. No checkpoint file. Crash recovery is free — next tick re-derives from
   disk. (Inv 1 + 4)
3. **Wipe + HEAD-check before every advance.** `git reset --hard && git clean
   -fd` discards a crashed skill's uncommitted mess; the HEAD guard skips any
   worktree whose branch doesn't match the feature being advanced, so a
   manually-checked-out branch is never clobbered. Runs only in per-feature
   worktrees. (Inv 3 + 6)
4. **Dispatcher never invokes an LLM directly.** It only spawns `claude -p`
   subprocesses or shells to `git`/`gh`. Each subprocess is a fresh process and
   a clean context window. (Inv 5)
5. **Local checks are optional and bounded.** If `scripts/local-checks.sh`
   exists, the PR is gated on it with a two-strike retry (auto-fix, then focused
   LLM fix, then stop). No script = gate skipped. (Inv 8)
6. **`MAX_WORKTREES` is the one concurrency knob.** Default 1 = FIFO single
   worktree. Claims are lazy: PRDs over capacity stay in `prd/<slug>/*` as the
   visible queue. In-flight work always continues regardless of cap changes.
   (Inv 2 + 3)

## Section-by-section map

| Lines (approx) | Section | Tweakable? |
|----------------|---------|------------|
| top | `flock -n` self-lock | No — serializes ticks; load-bearing. |
| config block | `SLUG`, `WATCH`, `WORKTREE_BASE`, `MAX_WORKTREES` | Via `.harness/env`, not by editing here. |
| `has_prd()` | Invariant-2 ownership filter | No — defines what "harness-owned" means. |
| step 1 | re-attach in-flight worktrees | The `bootstrap_worktree` hook call is the local addition (see below). |
| step 2 | lazy claim up to capacity | No — atomic rename is the claim lock. |
| step 3 | advance each feature one step | **This is where you add/remove pipeline steps** — one `elif` per step. |
| step 4 | cleanup merged/closed PRs | Safe to extend (e.g., notify on cleanup). |
| step 5 | post-merge `/learn` (Path A — memory, ground truth) | Debounce/idempotency live here. |
| step 6 | episodic `/capture-lesson` (Path B — learn from STUCK) | Fires per `stuck-<f>` sentinel, once each. |

## The bootstrap hook — project-owned worktree provisioning

A bare worktree has no `node_modules`, no `.env`, no generated code — so slice
signals and the PRD runner fail for reasons unrelated to the feature. So
immediately after every `git worktree add`, the dispatcher runs
`scripts/bootstrap-worktree.sh "$wt"` if that script exists and is executable.
The hook itself is canonical (the design doc's dispatcher includes it); what's
**project-owned** is the `bootstrap-worktree.sh` it calls — harness-init generates
that per project (see `worktree-bootstrap.md`). If the project has no bootstrap
script, the hook is a no-op.

Make sure the user knows: this hook is why the per-feature worktrees the harness
spins up are actually runnable.

## The two memory paths (steps 5 + 6)

The dispatcher feeds **two** memory write-paths, both human-merged via PR:

- **Step 5 — `/learn` (Path A).** Fires once per new `origin/main` sha (idempotent
  via `git ls-remote origin learn/<sha>`). Writes Expert shards, invariants,
  AGENTS.md pointers, and candidate lints, and **curates** `lessons.md`. Ground
  truth only — it never writes from a branch in flight.
- **Step 6 — `/capture-lesson` (Path B).** A terminal STUCK in step 3 drops a
  `.harness/stuck-<f>` sentinel; step 6 captures one episodic lesson from the
  struggle (idempotent via `lesson-captured-<f>`). Path B only *appends*; Path A
  *curates*. CLOSED-unmerged PRs are deliberately not a capture trigger.

## Common tweaks to offer

- **Add a pipeline step** (e.g. `/security-review` between validate and
  implement): insert one `elif` in step 3 with its own sentinel check. Nothing
  else changes.
- **STUCK caps** (now built in): `IMPLEMENT_CAP` (default 3) bounds PRD-runner
  retries before STUCK; `FEEDBACK_CAP` (default 5) bounds reviewer rounds. Both
  are env-overridable in `.harness/env`. Raise for hard features, lower to fail
  fast. Hitting either drops a `stuck-<f>` sentinel that feeds step 6.
- **Debounce window** for `/learn`: currently fires whenever `origin/main` sha
  changed. A team merging many times per minute may want a time-debounce.

## What NOT to let the user do

- Add `&` to any `claude -p` call (breaks synchronous one-step discipline).
- Add an LLM call to the decision logic (breaks Inv 5, makes it non-reproducible
  and a cost surface).
- Remove the wipe or the HEAD guard (breaks crash recovery and Inv 6).
- Replace the atomic-rename claim with a marker file (breaks Inv 2 + 7).
