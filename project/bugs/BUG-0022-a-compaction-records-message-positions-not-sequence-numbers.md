---
id: BUG-0022
artifact: bug
status: approved
severity: major
violates: REQ-2209
found: 2026-09-27
revised: 2026-09-27
issue: 195
---

# A compaction records message positions, not the sequence numbers it supersedes

The engine reports a compaction as a range of positions in its in-memory message
list, and the relay writes that range into the session log as if it were a range
of event sequence numbers. The recorded range names the wrong events, and it is
written through `append`, which skips the overlap check `Sessions::compact`
makes.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run an agent with a small context window and compaction on, against recorded
   replies long enough to compact twice.
2. Read the two `Compaction` events in the session log.

The first names positions such as `1..=6` in the message list, where the events
it summarised sit at other sequence numbers, because the log also holds
`Started`, `ToolCall`, `Usage` and `Policy` events that aren't messages. The
second, taken after the first summary was spliced in, starts again from
position 1 and overlaps the first. This was traced through the code and not run.

## What the system does

`crates/meow-agent/src/event.rs:72-79` defines `supersedes` as "the messages
this replaces, by position", and `crates/meow-agent/src/engine.rs:216-221`
sends that range and then splices the list, so later positions shift. The relay
at `crates/meow-star/src/run.rs:757-768` converts it to `start..=end - 1` and
calls `keep`, which reaches `Sessions::append`
(`crates/meow-session/src/log.rs:141-172`). `Sessions::compact` at
`crates/meow-session/src/log.rs:181-207`, which refuses an overlap, isn't
called.

## What it should do, and why

REQ-2209: "A `Compaction` event MUST name the inclusive sequence range it
supersedes." REQ-2214: "A `Compaction` event MUST NOT supersede a range that an
earlier `Compaction` event already supersedes." A rebuild that skips the
recorded range drops the wrong events, so what a resumed model sees differs from
what it saw.

## Triage

The defect enters at the boundary between `meow-agent`, which knows messages,
and the relay in `meow-star`, which knows the log. It is major because the
requirement's main behaviour doesn't happen and the log's compaction record
can't be trusted for a rebuild. The fix is for the relay to track which sequence
number each message came from and to record through `Sessions::compact`.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`a_compaction_names_the_sequence_numbers_it_replaced`, that compacts twice and
expects each `Compaction` event to cover exactly the events it summarised, with
no overlap.
