---
id: ADR-2203
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2228, REQ-2229, REQ-2230, REQ-2234, REQ-2235]
supersedes: []
---

# 2203. The event log defines a session's state

## Decision

A session's state is `running` when its last event isn't `Finished`, and
otherwise the stop reason that last `Finished` event carries; a denormalised
copy is allowed only if it can be rebuilt from the log and the log wins on any
disagreement (from docs/spec/session.md [R-SESSION-041], high). A session whose
recording process is no longer alive gets a `Finished` event with the stop
reason `failed` when it is opened (from docs/spec/session.md [R-SESSION-042],
high).

Once this holds, `Sessions::state` reads only the last event, and the cached
copy written on start, finish and reap is never consulted for the answer (from
crates/meow-session/src/log.rs:269-290, high). What doesn't hold yet: nothing
in the binary calls `Sessions::reap_if_dead` when it opens a session, so a
session whose process died still reads as `running` (from `grep -rn
reap_if_dead crates`, which finds only the definition and one test, high).

## Why

In v0.2.x, `status` was a mutable column that could disagree with the events
beneath it, and a session whose process died stayed `running` forever (from
docs/spec/session.md, high). Reading the state from the `Finished` event leaves
no separate flag that can disagree with the last event (from
docs/design/0.3.0-sessions.md section 5, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A mutable `status` column, as in v0.2.x (the do-nothing option) | Listing many sessions reads one column, with no read of the log (from docs/spec/session.md [R-SESSION-041], medium) | The column could disagree with the events beneath it, and a session whose process died stayed `running` forever (from docs/spec/session.md, high) |
| Derive the state from the log with no stored copy at all | No second copy exists to go stale (reasoned from docs/spec/session.md [R-SESSION-041], low) | Listing a thousand sessions would read a thousand events, which is why the requirement allows a rebuildable copy (from docs/spec/session.md [R-SESSION-041], high) |
| Leave a dead session `running` until someone finishes it by hand | Opening a session never writes to it (reasoned from crates/meow-session/src/log.rs:300-325, low) | A session that claims to be running forever is worse than one that admits it failed (from crates/meow-session/src/log.rs:302-304, high) |

## What it costs

Reading the state needs the last event decoded, and the cached copy is written
alongside every start, resume, finish and reap, so each lifecycle change is two
writes (from crates/meow-session/src/log.rs:77-133 and 322-323, high). Opening
a dead session appends to its log, so opening isn't read-only (from
docs/spec/session.md [R-SESSION-042], high).

## What would reverse it

- Listing sessions measured too slow because the state is read from the log
  per session, with the cached copy unable to help because it can't be trusted
  (reasoned from crates/meow-cli/src/session.rs:173-186, low).

## Consequences

- `meow session list` shows the state from the log, falling back to `running`
  when the read fails (from crates/meow-cli/src/session.rs:173-174, high).
- Resuming a session with a run still open fails, and finishing one with no
  open run fails, because both check the state from the log first (from
  crates/meow-session/src/log.rs:94-124, high).
- Deciding that the recording process is no longer alive needs a liveness
  signal, which ADR-2206 chooses (from docs/spec/session.md, high).

## How I will know it was realised

1. `the_state_is_whatever_the_log_says` and
   `the_cached_state_never_overrides_the_log` in
   crates/meow-session/tests/spec.rs pass (from
   crates/meow-session/tests/spec.rs:425-467, high).
2. `a_session_whose_writer_died_is_closed_on_the_next_open` in the same file
   passes (from crates/meow-session/tests/spec.rs:469-491, high).
3. Killing a running `meow` process and then running `meow session list` more
   than three heartbeat intervals later shows the session as `failed`; this
   fails today because no command reaps (reasoned from `grep -rn reap_if_dead
   crates`, low).

## What this does not settle

- How liveness is detected, which ADR-2206 settles (from docs/spec/session.md,
  high).
- Which command opens a session in the sense of [R-SESSION-042], that is,
  where the binary calls the reap: the specification doesn't say (from
  docs/spec/session.md, medium).
