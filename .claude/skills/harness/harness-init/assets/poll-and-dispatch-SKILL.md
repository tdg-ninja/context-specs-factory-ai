---
name: poll-and-dispatch
description: One tick of the harness outer loop. Reads disk state, dispatches one skill per branch. Run on an interval via /loop.
---

# poll-and-dispatch

Run `./scripts/poll-and-dispatch.sh` from the repo root and report what it did.

That is the entire skill. The intelligence is in the script — this exists only
so `/loop` has a slash-command-shaped target. Do not do any feature work in this
session; the script shells out to fresh `claude -p` subprocesses for all real
work, which is what keeps this outer session's context from bloating across
ticks.
