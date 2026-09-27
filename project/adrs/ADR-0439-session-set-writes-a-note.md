---
id: ADR-0439
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2203]
supersedes: []
---

# 0439. `ctx.session.set` writes a `Note`, not a new event kind

## Decision

`ctx.session.set` writes a `Note` with level `state`, and `get` reads from
memory within the run (from https://github.com/meowshed/meowg1k/pull/129, high).

Once this is accepted, `ctx.session.set` stores the value in the run's memory
and appends a `Note` with level `state` and the message `key=value`, and
`ctx.session.get` answers from that memory (from
crates/meow-cli/src/session.rs:123, high). A resumed run can't read a value the
earlier run set: the rebuild ignores notes and the memory starts empty (from
crates/meow-cli/src/session.rs:97 and crates/meow-cli/src/session.rs:75, high).

## Why

The log has ten kinds and none of them is a key-value pair, and making the value
replayable needs an eleventh kind, which is a schema migration and a
specification amendment (from https://github.com/meowshed/meowg1k/pull/129, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| An eleventh event kind for a key-value pair | A rebuild could read the value back (from https://github.com/meowshed/meowg1k/pull/129, high) | It needs a schema migration and a specification amendment (from https://github.com/meowshed/meowg1k/pull/129, high) |
| Typed rows keyed by name beside the log, as the design describes | A resumed session would see what the original left behind (from docs/design/0.3.0-sessions.md section 7, high) | A row outside the log is state the log doesn't define, and REQ-2230 allows such a copy only if the log can rebuild it (reasoned from docs/requirements/REQ-2230-state-copy-may-be-denormalised.md, low) |
| Do nothing: keep the value in memory and write nothing to the log | No note per `set`, so the log holds only what the run did (reasoned from crates/meow-cli/src/session.rs:127, low) | A value a handler stored should be replayable with the run that stored it, not live only in the process (from crates/meow-cli/src/session.rs:131, high) |

## What it costs

A rebuild can't read the value back (from
https://github.com/meowshed/meowg1k/pull/129, high). The design promises that a
resumed session sees what the original left behind, and this doesn't deliver it
(from docs/design/0.3.0-sessions.md section 7, high). The value is written as
text, so a reader of the log gets `key=value` with the value in its display form
and not its type (from crates/meow-cli/src/session.rs:133, high).

## What would reverse it

A command that needs a value from an earlier run of the same session, read
through `ctx.session.get` after `--continue`, would need the eleventh kind or a
rebuild that reads the notes back (reasoned from
crates/meow-cli/src/session.rs:97, low). An amendment adding a key-value kind
to REQ-2203 would reverse it.

## Consequences

- The event kinds stay the ten REQ-2203 lists (from
  crates/meow-core/src/event.rs:155, high).
- `ctx.session.get` returns only what this run set (from
  crates/meow-cli/src/session.rs:123, high).
- A failed write of the note is ignored: `record` drops the error from
  `Sessions::append`, although the design says a failed session write fails the
  run (from crates/meow-cli/src/session.rs:140 and
  docs/design/0.3.0-sessions.md section 3.1, high).

## How I will know it was realised

`every_context_member_works` in `crates/meow-star/tests/running.rs` passes: a
handler sets `plan` and reads it back within the run (from
crates/meow-star/tests/running.rs:405, high). No test checks the `Note` with
level `state` in a written session (reasoned from
crates/meow-cli/src/session.rs:127, low).

## What this does not settle

- How a resumed run would get the values back: through an eleventh kind or by
  reading the notes (from https://github.com/meowshed/meowg1k/pull/129, high).
- Whether a failed session write should fail the run, as the design says, or be
  dropped, as `record` does now (from crates/meow-cli/src/session.rs:140, high).
