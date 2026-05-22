# Config options — `.harness/env`

The harness has a small, deliberately tiny config surface. All of it lives in
`.harness/env` (sourced by the dispatcher) or in the `/loop` invocation. When
you walk the user through config, explain each knob's tradeoff and recommend the
default; only change a default if the user has a concrete reason.

## The knobs

### `MAX_WORKTREES` (default `1`)

How many features the harness works in parallel.

- **`1` (recommended default).** Single worktree, FIFO. One feature at a time.
  No disk multiplication, no duplicate `node_modules`/`target`, simplest mental
  model. Right for essentially every single-developer project.
- **`2`+.** Bounded parallelism. Each concurrent feature gets its own worktree
  (`${WORKTREE_BASE}-${feature}`), so disk and rebuild cost multiply. Flip this
  only when you have several independent features queued AND surplus disk/CI.

In-flight work always continues regardless of this value — it caps *new intake*,
not continuation. Lowering it mid-flight just stops new claims; existing
features finish.

### `WATCH_PATTERN` (default `prd/<your-slug>/*`)

Which PRD branches this harness claims. `<your-slug>` derives from
`git config user.email` (the part before `@`).

| Mode | Pattern | Behavior | When |
|------|---------|----------|------|
| **Per-dev (default)** | `prd/<my-slug>/*` | Each dev's harness claims only their own PRDs, on their own laptop, their own API quota. | Default. Best attribution, cost, and observability. |
| **Shared pool** | `prd/*/*` | All harnesses race for everything; atomic rename keeps it safe. | A team deliberately sharing a work pool. |

Note the glob: `*` matches one path component. Per-dev is `prd/alice/*`; shared
is `prd/*/*` (author component + feature component).

### Loop interval (in the `/loop` invocation, not `.harness/env`)

`/loop 5m /poll-and-dispatch` — the `5m` is the tick cadence. 5 minutes is a
sane default. Shorter = more responsive, more API ticks (most are cheap
no-ops); longer = lazier. This is a UX knob, not a correctness one.

## What does NOT go in config

- **The claim mechanism** — it's the atomic rename, hardcoded. Not configurable.
- **Completion signals** — sentinel files, hardcoded in the dispatcher.
- **The pipeline order** — that's the `if/elif` chain in the script, edited
  directly (see dispatcher-explained.md), not a config value.

## The `.harness/` directory

Holds runtime state, all re-derivable, none of it secret:
- `env` — the config above (committed, or kept local — see below).
- `last-main-sha` — last-seen `origin/main` for expert-update debounce.
- `feedback-rounds-<f>`, `local-check-attempts-<f>` — per-feature counters.

**Counters must be gitignored.** They are per-node runtime state, not project
artifacts. harness-init adds `.harness/feedback-rounds-*`,
`.harness/local-check-attempts-*`, and `.harness/last-main-sha` to
`.gitignore`. Whether `.harness/env` is committed is a choice: commit it to
share defaults across a team's checkouts; keep it local (gitignored) if each
dev tunes their own. Recommend committing `env` and ignoring the counters.

## Multi-developer reminder

Per-dev is the default for a reason (attribution, cost isolation, "my harness is
my assistant" mental model). Don't steer a user to shared-pool unless they
explicitly describe a shared-queue workflow. When a team truly outgrows local
harnesses, the move is to transition to a server/CI harness — not to run many
local harnesses in shared-pool mode forever.
