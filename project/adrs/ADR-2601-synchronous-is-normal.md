---
id: ADR-2601
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2602]
supersedes: []
---

# 2601. `synchronous` is `NORMAL`

## Decision

The store sets `synchronous` to `NORMAL` [REQ-2602] (from docs/spec/store.md,
high).

## Why

With write-ahead logging, `NORMAL` loses at most the last transaction on a power
failure, which costs a rerun of one turn (from docs/spec/store.md, high). The
write path is on the agent's critical path, and `NORMAL` with write-ahead
logging is much faster than `FULL` (from `git show
f8e58ea:docs/spec/store.md`, Open questions, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| `FULL` | Keeps the last transaction through a power failure (from docs/spec/store.md, Decisions, high) | It buys that back at a cost paid on the agent's critical path, on every write (from docs/spec/store.md, Decisions, high) |
| Do nothing: leave `synchronous` at the SQLite default | One pragma fewer to set and test (reasoned from crates/meow-store/src/lib.rs:80-85, low) | The default for a database SQLite opens is `FULL`, so doing nothing is the `FULL` row with the choice left implicit (from <https://www.sqlite.org/pragma.html#pragma_synchronous>, medium) |

A third level, `OFF`, isn't named anywhere in the spec, the design documents or
the history; the source weighs only `NORMAL` against `FULL` (from `git show
f8e58ea:docs/spec/store.md`, Open questions, high).

## What it costs

A power failure can lose the last transaction, which costs a rerun of one turn
(from docs/spec/store.md, high). Each event commits on its own, so the last
transaction is one event, and a crash drops at most that event from the log
(reasoned from docs/spec/store.md [R-STORE-020], low).

## What would reverse it

A lost final event that a rerun can't recover would reverse it, because the
recommendation rests on a lost final turn being recoverable by rerunning (from
`git show f8e58ea:docs/spec/store.md`, Open questions, high). A measurement
showing `FULL` adds no noticeable time to a turn would remove the reason
`FULL` lost (reasoned from docs/spec/store.md, Decisions, low).

## Consequences

`Store::open_at` sets `journal_mode` to `WAL`, `synchronous` to `NORMAL` and
`foreign_keys` on, on every connection it opens, and the comment beside the
pragmas repeats the reason (from crates/meow-store/src/lib.rs:79-85, high).
The open question was one of three the plan named as blocking the earliest
milestones, and settling it unblocked M1 (from
docs/design/0.3.0-plan.md:404-407, high).

## How I will know it was realised

1. `PRAGMA synchronous` on a connection `Store::open` returned reads `1`, the
   value for `NORMAL`. No test checks it yet:
   `the_connection_pragmas_are_set` in crates/meow-store/tests/spec.rs cites
   [REQ-2602] and asserts only the foreign key refusal and the write-ahead log
   file (from crates/meow-store/tests/spec.rs:36-57, high).

## What this does not settle

- The busy timeout, which [R-STORE-023] sets at a floor of five seconds and the
  code sets at ten (from crates/meow-store/src/lib.rs:34-39, high).
- Whether a crash mid-run marks the session `failed` on the next open, which is
  the session spec's liveness rule and not the store's (reasoned from
  docs/design/0.3.0-sessions.md section 3.1, low).
