---
id: BUG-0114
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A value set with `ctx.session.set` is gone when the session is resumed

`ctx.session.set` writes a `Note` with level `state` into the log and keeps the
value in memory. A run resumed with `--continue` starts with an empty map and
doesn't read the notes back, so `ctx.session.get` returns `None` for every key
the original run set.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare a command whose handler prints `ctx.session.get("n")` and then calls
   `ctx.session.set("n", 1)`.
2. Run it once. It prints `None`.
3. Run it again with `--continue`. It prints `None`, not `1`.

This follows from the code below.

## What the system does

`Log::open` builds `values` as an empty `BTreeMap` on both branches,
`crates/meow-cli/src/session.rs:57-76`. `rebuild`, `session.rs:97-117`, turns
the log into messages and skips `Note` events. `Log::set`,
`session.rs:127-137`, writes the value as `key=value` text, which nothing
parses back.

## What it should do, and why

No requirement covers what `ctx.session` keeps across a resume, which is a gap
for the requirements step. The comment on the `Session` port,
`crates/meow-star/src/port.rs:95-96`, says a stored value is "replayable with
the run that stored it", and ADR-0439 records that a resumed run can't read it.
A resumed run should see the values the session holds.

## Triage

The defect enters in `meow-cli`'s session adapter. No requirement is violated,
so the fix enters at the requirements step: decide whether session values
survive a resume and, if so, how they're recorded so they can be read back.
It's minor because `store` offers durable values and `ctx.session` is new.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`a_session_value_survives_continue`, running the reproduction above and
expecting `1` on the second run.
