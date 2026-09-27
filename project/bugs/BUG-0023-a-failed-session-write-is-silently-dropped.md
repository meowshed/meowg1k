---
id: BUG-0023
artifact: bug
status: approved
severity: major
violates: REQ-2626
found: 2026-09-27
revised: 2026-09-27
issue: 196
---

# A failed session write is dropped without a word

The CLI's session port discards the result of every `Sessions::append`. When the
store refuses a write, because the disk is full or the database is locked past
its busy timeout, the event is lost, the run carries on, and nothing tells the
user the log is incomplete.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run a workspace command whose handler calls `ctx.session.set` and runs an
   agent.
2. While it runs, make `.meow/.data/meow.db` read-only, or hold an exclusive
   lock on it from another connection for longer than the busy timeout.

The run finishes normally and the session log lacks the events written after
step 2, with no diagnostic. This was traced through the code and not run.

## What the system does

`crates/meow-cli/src/session.rs:139-143` is `let _ = sessions.append(&self.id,
kind);`, and it also skips the write when the mutex is poisoned. The default
`Session::record` in `crates/meow-star/src/port.rs:107-115` returns nothing, and
its comment defends not reporting the failure.

## What it should do, and why

REQ-2626: "The store MUST NOT log a failure and continue." Here the failure
isn't even logged. The vision's fifth goal says every turn, tool call, policy
decision and token lands in an append-only log, and a log that silently misses
events can't be audited. No requirement says whether the run fails or continues
with a diagnostic; that choice routes to the requirements step.

## Triage

The defect enters in the `Session` port, whose `record` has no way to return an
error, and in `meow-cli`'s implementation of it. It is major: the log's main
promise fails under a storage fault, and the fault is hidden. It isn't rated
critical because it needs a storage failure to happen.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`a_session_write_that_fails_is_reported`, that drives the port against a store
that refuses writes and expects either a failed run or a diagnostic event,
whichever the new requirement chooses.
