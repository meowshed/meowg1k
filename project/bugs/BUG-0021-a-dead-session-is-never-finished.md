---
id: BUG-0021
artifact: bug
status: approved
severity: major
violates: REQ-2234
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A session whose process died is never finished

`Sessions::reap_if_dead` appends `Finished { stop: failed, reason: "process
exited" }` for a session with a stale heartbeat, and only its test calls it.
Opening such a session through `meow session list`, `show`, `export` or
`--continue` leaves it unfinished and reported as running.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `grep -rn reap_if_dead crates`. The callers are in
   `crates/meow-session/tests/spec.rs:479-490` only.
2. Start a workspace command that runs long, kill the process with `SIGKILL`,
   and run `meow session list`. The session shows no `failed` finish. This
   step wasn't run; it follows from step 1.

## What the system does

`crates/meow-session/src/log.rs:305-325` implements the reaping, and no code in
`meow-cli` calls it when it opens, lists or resumes a session.

## What it should do, and why

REQ-2234: "Opening a session whose last event is not `Finished` and whose
recording process is no longer alive MUST append `Finished { stop: failed,
reason: "process exited" }`." REQ-2235 adds that it MUST NOT be left reported as
running.

## Triage

The defect enters in `meow-cli`'s session commands, which open sessions without
the check. It is major because the requirement's main behaviour doesn't happen.
It depends on BUG-0020: until the heartbeat is kept fresh, calling
`reap_if_dead` would also finish live runs, so the two fixes land together.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`listing_a_session_whose_writer_died_finishes_it`, that seeds a session with a
stale heartbeat and no `Finished`, runs `meow session list`, and expects a
`failed` finish with reason `process exited`.
