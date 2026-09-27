---
id: ADR-2206
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2231, REQ-2232, REQ-2233, REQ-2234]
supersedes: []
---

# 2206. A session's liveness is a heartbeat on its row

## Decision

While a run is in flight, its writer updates a heartbeat timestamp on the
session row at a fixed interval, and a session whose last event isn't `Finished`
and whose heartbeat is older than three intervals is treated as dead (from
docs/spec/session.md [R-SESSION-043], high).

Once this holds, the interval is `HEARTBEAT_SECS`, 10 seconds, the heartbeat is
the `heartbeat` column of the `sessions` table, and `Sessions::reap_if_dead`
appends `Finished` with `failed` and "process exited" to a session whose
heartbeat is more than 30 seconds old (from crates/meow-session/src/log.rs:12-18
and 300-325 and crates/meow-store/src/migrations.rs:40-47, high). What doesn't
hold yet: the binary writes the heartbeat once, when the session starts, and
nothing updates it during the run or calls `reap_if_dead` (from
crates/meow-session/src/log.rs:79 and `grep -rn "heartbeat\|reap_if_dead"
crates`, which finds no caller outside meow-session and its tests, high).

## Why

A heartbeat needs no platform code, and the window in which a dead session still
looks alive is bounded by the interval and tunable (from docs/spec/session.md,
high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| No liveness signal, as in v0.2.x (the do-nothing option) | No write during a run and nothing to tune (reasoned from docs/spec/session.md, low) | A session whose process died stayed `running` forever (from docs/spec/session.md, high) |
| A process identifier plus its start time | Chosen first for being portable (from docs/spec/session.md, high) | It isn't portable: reading another process's start time means procfs on Linux, sysctl on macOS and a Win32 call on Windows, the same per-platform work as an advisory lock for a weaker guarantee (from docs/spec/session.md, high) |
| An advisory lock | Gives a stronger guarantee than a process identifier plus start time (from docs/spec/session.md, high) | It needs per-platform work, where a heartbeat needs no platform code (from docs/spec/session.md, high) |
| A heartbeat recorded as an event in the log | Liveness replays with the rest of the run (reasoned from commit ed9810f, low) | The event kinds are closed and the log is append-only, so the two requirements couldn't both hold with a heartbeat event (from commit ed9810f, high) |

## What it costs

One small write per interval, on a run that is already writing events (from
docs/spec/session.md, high). A dead session looks alive for up to three
intervals, 30 seconds (from crates/meow-session/src/log.rs:12-18, high). A
live writer that can't beat for three intervals, such as one blocked in a long
tool call with no separate thread to beat, is reaped as dead (reasoned from
crates/meow-session/src/log.rs:305-325, low).

## What would reverse it

- A live run reaped as dead because its writer couldn't update the heartbeat
  in time, seen as a `Finished` with "process exited" on a session whose
  process was still running (reasoned from
  crates/meow-session/src/log.rs:305-325, low).

## Consequences

- The heartbeat isn't an event, because it is mutable and the log isn't (from
  docs/spec/session.md [R-SESSION-043], high).
- Reaping compares against a `now` the caller passes, so a test can reap
  without waiting (from crates/meow-session/src/log.rs:305 and
  crates/meow-session/tests/spec.rs:469-491, high).
- A session with no heartbeat recorded is never reaped, because
  `reap_if_dead` returns without acting when the column is empty (from
  crates/meow-session/src/log.rs:309-311, high).

## How I will know it was realised

1. `a_session_whose_writer_died_is_closed_on_the_next_open` in
   crates/meow-session/tests/spec.rs passes (from
   crates/meow-session/tests/spec.rs:469-491, high).
2. During a `meow` run longer than 30 seconds, the `heartbeat` column of its
   session row keeps advancing; no test checks this, and today it doesn't
   advance (reasoned from `grep -rn heartbeat crates`, low).

## What this does not settle

- Which operation counts as opening a session and calls the reap, which
  [R-SESSION-042] leaves to the caller (from docs/spec/session.md, medium).
- Whether the interval is configurable: it is a constant today (from
  crates/meow-session/src/log.rs:18, high).
