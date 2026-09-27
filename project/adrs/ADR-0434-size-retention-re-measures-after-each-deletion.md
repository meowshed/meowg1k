---
id: ADR-0434
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2255, REQ-2636]
supersedes: []
---

# 0434. The size limit deletes and re-measures one session at a time

## Decision

The size limit deletes and re-measures one session at a time (from
https://github.com/retran/meowg1k/pull/128, high).

Once this is accepted, a sweep with a size limit deletes the oldest session
tree, measures the database, and repeats until it fits or every session has
been visited (from crates/meow-session/src/retention.rs:80 to 94, high). The
measure is the length of the database file plus its `-wal` and `-shm` files
(from crates/meow-store/src/lib.rs:127, high). Nothing runs `VACUUM` or sets
`auto_vacuum` (from a search for `vacuum` under crates/, high), and SQLite
keeps freed pages in the file rather than returning them, so a deletion
doesn't shrink the measure and the loop deletes every unprotected session once
the file is over the limit (reasoned from crates/meow-store/src/lib.rs:127,
medium). No caller passes a size limit today (from
crates/meow-cli/src/wire.rs:737, high).

## Why

Deleting is what changes the file size, so a plan made up front would be wrong
after the first deletion (from https://github.com/retran/meowg1k/pull/128,
high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: retention by age and count only | No database query inside a loop, and no dependence on how the file shrinks (reasoned from crates/meow-session/src/retention.rs:64, low) | REQ-2255 asks for a size limit, because a log without one ends as a gigabyte of SQLite (from docs/design/0.3.0-sessions.md section 8, high) |
| Plan the deletions up front | One pass, and the set to delete is known before anything goes (reasoned, low) | The plan is wrong after the first deletion changes the file size (from https://github.com/retran/meowg1k/pull/128, high) |

## What it costs

It is a loop with a database query in it, fine for a sweep and wrong for
anything on a run's path (from https://github.com/retran/meowg1k/pull/128,
high). Without a vacuum the loop can delete every unprotected session (reasoned
from crates/meow-store/src/lib.rs:127, medium).

## What would reverse it

- The store can report a session's own size, so the deletions needed can be
  computed before any is made (reasoned from
  crates/meow-store/src/lib.rs:127, low).

## Consequences

- The store exposes the database's size on disk, including the write-ahead log
  (from crates/meow-store/src/lib.rs:127, high).
- The sweep calls the delete step inside the size loop as well as after it, so
  a size sweep and an age or count sweep share one deletion path (from
  crates/meow-session/src/retention.rs:91 and 96, high).

## How I will know it was realised

1. A sweep over a database above the limit stops once the measured size is at
   or below it, and keeps the sessions it didn't need to delete; no test covers
   the size limit today (from a search for `max_bytes` under
   crates/meow-session/tests, high). That test would fail today unless the
   store reclaims freed pages (reasoned, medium).

## What this does not settle

- Whether the store vacuums after a sweep, and when (from a search for
  `vacuum` under crates/, high).
- Where the size limit is configured; ADR-0433 records that nothing configures
  it yet.
