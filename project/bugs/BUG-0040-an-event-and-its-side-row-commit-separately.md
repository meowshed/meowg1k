---
id: BUG-0040
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A `Usage` or `Compaction` event and its side-table row commit separately

`Sessions::append` inserts the event and then, in a second statement, the row in
the `usage` or `compactions` side table. Each commits on its own, so a crash
between them leaves the event without its row, and everything that reads the
side table disagrees with the log.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Open a store whose `usage` table has a trigger that raises an error on
   insert, standing in for a crash after the first statement.
2. Call `Sessions::append` with an `EventKind::Usage`.

The call fails, and the event row is committed without its usage row. This was
traced through the code and not run.

## What the system does

`crates/meow-session/src/log.rs:141-147` appends the event, and
`crates/meow-session/src/log.rs:150-170` then calls `put_usage` or
`put_compaction`. The store runs each statement in autocommit mode
(`crates/meow-store/src/rows.rs:74-111`), with no transaction around the pair.

## What it should do, and why

No requirement says an event and its side row commit together; this is a gap in
the requirements and routes to the requirements step. ADR-0405 says each event
is durable when the call returns, which holds for the event and not for its
row. The side tables are read on every rebuild
(`crates/meow-session/src/log.rs:149`), so a missing compaction row means a
rebuild ignores the compaction, and a missing usage row undercounts the
session's spend.

## Triage

The defect enters in `meow-session`'s `append`. It is minor: it needs a crash
or a storage error in a window of one statement.

## Closed by

A test in `crates/meow-session/tests/`, named for example
`an_event_and_its_side_row_commit_together`, that fails the side insert and
expects neither row present.
