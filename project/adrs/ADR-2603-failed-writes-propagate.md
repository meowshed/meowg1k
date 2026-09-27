---
id: ADR-2603
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2625, REQ-2626]
supersedes: []
---

# 2603. A failed write propagates to the caller

## Decision

A failed write reaches the caller as an error, and the store never logs a
failure and continues [REQ-2625] [REQ-2626] (from docs/spec/store.md, high).

## Why

A log whose entire job is to be trusted later is worse than useless when it can
be silently incomplete (from docs/spec/store.md, high). v0.2.x's diagnostics
also went to `log.Printf`, which wrote straight through the terminal frame and
corrupted the display on the way out (from docs/design/0.3.0-sessions.md
section 2 and docs/design/0.3.0-architecture.md section 2, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: best-effort writes, as v0.2.x's `module_llm.go` does, logging ten distinct failures through `log.Printf` and continuing | A run keeps going when the log can't be written, so the user still gets the answer (reasoned from crates/meow-star/src/port.rs:107-113, which argues the same for the port, medium) | The session log can be silently incomplete (from docs/spec/store.md, Changes from v0.2.x, high), and the message corrupts the terminal on its way out (from docs/design/0.3.0-sessions.md section 2, high) |

The question has two answers at the store, report or swallow, and the source
weighs only these two. What a caller does with the error, for example failing
the run as docs/design/0.3.0-sessions.md section 3.1 proposes, belongs to the
caller and not to this decision (reasoned from docs/spec/store.md, Scope, low).

## What it costs

Every store call returns a `Result`, and every caller has to decide what a
failed write means for it (from crates/meow-store/src/error.rs and
crates/meow-store/src/lib.rs, high). A caller that can't act on the error has
to drop it explicitly, where best-effort writes would have dropped it for it
(reasoned from crates/meow-cli/src/session.rs:139-143, low).

## What would reverse it

A store call site where reporting the failure loses more than the record
would argue for swallowing it there. The `Session` port already makes that
argument for a handler's log: a log that can't be written is the case where
saying so loudly loses the run as well as the record (from
crates/meow-star/src/port.rs:107-112, high). That argument applies at the
port and not at the store, so it hasn't reversed this decision.

## Consequences

- Every write in `meow-store` returns `StoreError` through `?`, and the store
  has no logging dependency to log with: its dependencies are `blake3`,
  `rusqlite` and `thiserror` (from crates/meow-store/Cargo.toml:13-16, high).
- The store keeps its promise, and the layer above it doesn't. The `Session`
  port's `record` returns nothing, and its implementation in `meow-cli`
  discards the error with `let _ = sessions.append(&self.id, kind);`, so a
  failed session write leaves the log incomplete and nothing says so (from
  crates/meow-star/src/port.rs:107-113 and
  crates/meow-cli/src/session.rs:139-143,
  high).

## How I will know it was realised

1. `a_failed_write_is_returned_rather_than_logged` in
   crates/meow-store/tests/spec.rs passes: appending the same sequence number
   twice returns `StoreError::Sqlite` and leaves one event behind (from
   crates/meow-store/tests/spec.rs:205-222, high).
2. A session write that fails during a run reaches something the user sees.
   It doesn't today, because `Log::record` in crates/meow-cli/src/session.rs
   drops it (from crates/meow-cli/src/session.rs:139-143, high).

## What this does not settle

- What a run does when its session write fails: fail the run, as
  docs/design/0.3.0-sessions.md section 3.1 proposes, or keep going, as the
  `Session` port does (from docs/design/0.3.0-sessions.md section 3.1 and
  crates/meow-star/src/port.rs:107-113, high).
- How a failed write is shown to the user, which the event stream and
  ADR-2806 decide: a diagnostic goes into scrollback through the renderer
  (reasoned from CLAUDE.md, `no_stdout_behind_the_tui`, low).
