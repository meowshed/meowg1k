---
id: BUG-0020
artifact: bug
status: approved
severity: major
violates: REQ-2231
found: 2026-09-27
revised: 2026-09-27
issue:
---

# The session heartbeat is written only when the session starts

`Sessions::start` writes the heartbeat once. Nothing updates it while the run is
in flight, so a live run lasting longer than three intervals looks the same as
one whose process died.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `grep -rn heartbeat crates`. The production callers are
   `crates/meow-session/src/log.rs:79`, inside `Sessions::start`, and the
   `Sessions::heartbeat` method itself at `crates/meow-session/src/log.rs:295`,
   which only `crates/meow-session/tests/spec.rs:478` calls.

## What the system does

`crates/meow-session/src/log.rs:79` sets the heartbeat when the session row is
created, and no timer, task or engine event calls `Sessions::heartbeat` after
that.

## What it should do, and why

REQ-2231: "While a run is in flight its writer MUST update a heartbeat timestamp
on the session row at a fixed interval." REQ-2232 and REQ-2233 read liveness
from it.

## Triage

The defect enters in `meow-cli`'s session wiring, which owns the writer and
starts no heartbeat. It is major because the requirement's main behaviour
doesn't happen; it is harmless today only because nothing reads the heartbeat
(BUG-0021), and fixing that one alone would reap live runs.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`a_long_run_keeps_its_heartbeat_fresh`, that runs a command whose handler sleeps
past one interval and expects the heartbeat to have moved.
