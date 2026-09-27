---
id: ADR-0405
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2623, REQ-2624]
supersedes: []
---

# 0405. Each event commits on its own, and no transaction spans a turn

## Decision

`append_event` commits on its own, and another connection sees the event at once
(from https://github.com/retran/meowg1k/pull/113, high).

Once this is accepted, both store append paths, `append_event` and
`append_with_payload`, run one `INSERT` in autocommit mode, so each event is
durable when the call returns (from crates/meow-store/src/session.rs:55-75 and
crates/meow-store/src/rows.rs:74-111, high). Nothing in this decision is left
unbuilt (reasoned from the same files, low).

## Why

One transaction per turn held the single write lock across tool execution, which
blocks every other session and loses a long tool's result on a crash (from
https://github.com/retran/meowg1k/pull/111, high). The log is append-only, so a
half-written turn is a true record of how far the run got (from
https://github.com/retran/meowg1k/pull/113, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep the first `R-STORE-020`, "All writes belonging to one agent turn MUST be committed in a single transaction" (from commit e39cd2f, docs/spec/store.md, high) | A crash loses at most the turn in flight, and no reader sees a half-written turn (from docs/design/0.3.0-sessions.md at commit ad3f907, section 3.1, high) | It holds the single write lock across tool execution, blocking every other session and losing a long tool's result on a crash (from https://github.com/retran/meowg1k/pull/111, high) |
| Buffer a turn's events in memory and commit them together once its tools finish | The write lock is held only for the commit, and the turn still lands whole (reasoned from docs/design/0.3.0-sessions.md at commit ad3f907, "Writes are batched per turn", low) | A crash during a long tool still loses that tool's result, which the requirement names as a reason against a turn-wide write (from docs/requirements/REQ-2624-no-transaction-across-tool-execution.md, high) |

The history names one option besides the decision, and the buffered variant
is a reading of the same first design; no third was recorded (from commit
e39cd2f and https://github.com/retran/meowg1k/pull/111, medium).

## What it costs

A turn is not atomic: a reader can see a tool call whose result has not been
written yet, and a crash leaves the log ending mid-turn (from
docs/requirements/REQ-2624-no-transaction-across-tool-execution.md, high).
Every event pays its own commit; with write-ahead logging and
`synchronous = NORMAL` a power failure loses at most the last transaction,
which costs a rerun of one turn (from crates/meow-store/src/lib.rs:79-84,
high). An event and its typed side-table row, usage or compaction, are written
by separate statements, so a crash between them leaves the event without its
row (from crates/meow-session/src/log.rs:141-172, medium).

## What would reverse it

- Tool execution moves out of the write path so that no turn-wide transaction
  would hold the write lock while a tool runs, which removes the reason given
  for this decision (reasoned from
  https://github.com/retran/meowg1k/pull/111, low).

## Consequences

- Another connection opened on the same database sees an event as soon as
  `append_event` returns (from crates/meow-store/tests/spec.rs:191-203, high).
- A failed write reaches the caller and leaves nothing behind, because there is
  no enclosing transaction to half-commit (from
  crates/meow-store/tests/spec.rs:205-222, high).

## How I will know it was realised

1. `an_event_is_visible_to_another_connection_at_once` in
   crates/meow-store/tests/spec.rs passes: a second `Store::open` on the same
   file counts the event the first handle just appended (from
   crates/meow-store/tests/spec.rs:191-203, high).

## What this does not settle

- Whether an event and its side-table row should commit together: they don't
  today, and no requirement says they must (from
  crates/meow-session/src/log.rs:141-172, medium).
- How a session left mid-turn by a crash is closed: the session log's
  lifecycle handles that, not the store (from docs/design/0.3.0-sessions.md,
  lines 127-128, medium).
