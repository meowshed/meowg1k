---
id: ADR-0453
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2464, REQ-2465]
supersedes: []
---

# 0453. `store` refuses every call when the database won't open

## Decision

When the workspace database won't open, every `store` call fails saying so;
`quiet::Unopened` carries the reason and refuses, and `quiet::Ephemeral` exists
for tests and is never what the binary chooses (from
https://github.com/meowshed/meowg1k/pull/143, high).

With this in place, the binary opens `Durable` and, when that fails, wires
`Unopened` with the message "the workspace store will not open" and the error
(from crates/meow-cli/src/wire.rs:1124-1133, high). `get`, `put`, `delete` and
`keys` on `Unopened` each return that message (from
crates/meow-star/src/port.rs:440-455, high). `Ephemeral` appears only in the
test harnesses (from crates/meow-star/tests/running.rs:328 and
crates/meow-star/tests/approval.rs:226, high).

## Why

A handler would write a value, read it back within the run and find it gone next
time, with nothing anywhere explaining why (from
https://github.com/meowshed/meowg1k/pull/143, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Fall back to an in-memory map | It was the obvious shape, and a handler's run keeps going (from https://github.com/meowshed/meowg1k/pull/143, high) | The value disappears on the next run with nothing explaining why (from https://github.com/meowshed/meowg1k/pull/143, high) |
| Refuse each `store` call with the reason (chosen) | The handler sees why at the call that needed the store, the way `quiet::NoIndex` reports a missing index (from https://github.com/meowshed/meowg1k/pull/143, high) | Chosen |

Doing nothing was not an option, because before #143 there was no `store`
module at all (from https://github.com/meowshed/meowg1k/pull/143, high). The
source names no third option.

## What it costs

A handler that uses `store` fails when the database won't open, even if the run
could have finished without the value (reasoned from
crates/meow-star/src/port.rs:440-455, low).

## What would reverse it

This is reversed if [R-STAR-026] is amended so that a value need not survive
the run that wrote it, because then an in-memory store would meet the
requirement (reasoned from docs/spec/starlark.md [R-STAR-026], low).

## Consequences

- A failure to open the database shows at the first `store` call, with its
  cause in the message (from crates/meow-cli/src/wire.rs:1131-1133, high).
- Commands that don't use `store` still run (from
  crates/meow-cli/src/wire.rs:1124-1133, medium).

## How I will know it was realised

`the_store_outlives_the_run_that_wrote_it` and
`session_gc_does_not_take_the_store_with_it` in
`crates/meow-cli/tests/surface.rs` pass, showing the binary wires the durable
store (from crates/meow-cli/tests/surface.rs:702 and 736, high). No test yet
drives the refusal: a test that makes the database unopenable and expects a
`store.get` to fail naming the reason would show it (from grep for `Unopened`
over crates, high).

## What this does not settle

- The doc comment on `Ephemeral` still says it is "for tests and for a run
  with no workspace database", which reads as a use the binary might choose
  (from crates/meow-star/src/port.rs:392-396, high).
- A value stored by a future version that encodes differently fails to decode,
  and there is no migration for it (from
  https://github.com/meowshed/meowg1k/pull/143, high).
