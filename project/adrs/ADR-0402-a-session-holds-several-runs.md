---
id: ADR-0402
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2205, REQ-2206, REQ-2207, REQ-2208]
supersedes: []
---

# 0402. A session holds one or more runs, each opened and closed

## Decision

A session holds one or more runs, each opened by `Started` and closed by
`Finished` (from https://github.com/meowshed/meowg1k/pull/110, high).

Once this is accepted, `Sessions::start` appends the first `Started`,
`Sessions::finish` closes the open run and refuses when none is open, and
`Sessions::resume` appends a new `Started` to an ended session and refuses
while a run is still open (from crates/meow-session/src/log.rs:53-134, high).
Nothing in this decision is left unbuilt (reasoned from the same file, low).

## Why

`R-SESSION-005` required `Finished` to be the last event of a session, while
`R-SESSION-050` had resume append after it, and the two couldn't both hold (from
https://github.com/meowshed/meowg1k/pull/110, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: `Finished` is the last event of a session | A session has one ending, so its state is a property of the whole session (reasoned from docs/design/0.3.0-sessions.md, "the state _is_ the `Finished` event", low) | Resume appends after it, so the requirement and resume contradict each other (from https://github.com/meowshed/meowg1k/pull/110, high) |
| Resume by rewriting or removing the old `Finished` | `Finished` stays the last event and a session keeps one ending (reasoned from REQ-2208, low) | The log is append-only, and resuming has to continue a session without rewriting the event that ended it (from docs/requirements/REQ-2208-resume-appends-started-event.md, high) |
| Resume into a new session | Each session keeps exactly one run (reasoned from REQ-2237, low) | Resuming has to append to the same session and continue its sequence (from crates/meow-session/src/log.rs:90-93 and docs/requirements/REQ-2237-resume-creates-no-new-session.md, high) |

## What it costs

A session's state is read from its last event, so it shows only the latest
run's ending, and an earlier run's ending stays in the log without showing in
the state (from crates/meow-session/src/log.rs:274-290, high). Every writer
also has to keep runs strictly alternating, so `finish` and `resume` each check
the current state and fail with `SessionError::Lifecycle` when it is wrong
(from crates/meow-session/src/log.rs:94-100 and
crates/meow-session/src/log.rs:117-123, high).

## What would reverse it

- Resume stops being a way to continue a session: REQ-2236 and REQ-2237 are
  withdrawn or resume starts a new session (reasoned from
  crates/meow-session/src/log.rs:88-110, low).

## Consequences

- `Started` carries the task and the agent for each run, so each run in one
  session records what it was asked (from
  crates/meow-session/src/log.rs:67-75 and
  crates/meow-session/src/log.rs:101-107, high).
- A second `finish` without a new `Started` is an error, and a second
  `resume` while a run is open is an error (from
  crates/meow-session/tests/spec.rs:128-179, high).

## How I will know it was realised

1. `a_run_opens_and_closes_exactly_once` in crates/meow-session/tests/spec.rs
   passes: the events are `Started`, `Finished`, and a second finish fails
   (from crates/meow-session/tests/spec.rs:128-149, high).
2. `resuming_appends_a_new_run_and_leaves_the_old_ending_alone` in the same
   file passes: the events are `Started`, `Finished`, `Started` and the state
   is `running` (from crates/meow-session/tests/spec.rs:152-179, high).

## What this does not settle

- Which prompt a resumed run uses: ADR-0437 settles that (from
  docs/adrs/ADR-0437-a-resumed-run-takes-the-current-prompt.md, high).
- How a session that a crashed process left open gets its closing `Finished`:
  the design says the next open appends one with a failed stop reason, and
  this decision doesn't choose that (from docs/design/0.3.0-sessions.md,
  lines 127-128, high).
