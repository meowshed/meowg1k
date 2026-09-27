---
id: ADR-1007
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1063, REQ-1064, REQ-1065, REQ-1066, REQ-1067, REQ-1068]
supersedes: []
---

# 1007. A sink error is handled the same way for every event kind

## Decision

When the sink returns an error, for every event kind alike, the engine stops
delivering to that sink, records a `Note` event naming the error and continues
the run. (from docs/spec/agent.md [R-AGENT-071], high)

## Why

In v0.2.x `emitEvent` discards callback errors while `makeCallback` propagates
them, so a bug in an event handler is fatal for text deltas and silent for tool
events. Rendering is not the work, so a broken sink can't destroy a run in
progress, and it can't fail silently either. (from docs/spec/agent.md Changes
from v0.2.x and [R-AGENT-071], high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: discard some sink errors and propagate others, as v0.2.x did | A handler's abort logic works on streamed text, because `makeCallback` propagates its error (from https://github.com/meowshed/meowg1k/pull/89#discussion_r2895914526, high) | A bug in an event handler is fatal for text deltas and silent for tool events (from docs/spec/agent.md Changes from v0.2.x, high) |
| Propagate every sink error and abort the run, as a review of v0.2.x suggested | A handler's abort logic in `on_event` takes effect (from https://github.com/meowshed/meowg1k/pull/89#discussion_r2895914526, high) | Rendering is not the work, so a broken sink must not destroy a run in progress (from docs/spec/agent.md [R-AGENT-071], high) |
| Discard every sink error | The run never stops on a rendering fault (reasoned from docs/spec/agent.md [R-AGENT-071], low) | A broken sink must not fail silently either, which is the half v0.2.x got wrong for tool events (from https://github.com/meowshed/meowg1k/pull/118 and docs/spec/agent.md [R-AGENT-071], high) |

## What it costs

A script's `on_event` callback can't stop a run by failing, because its error
only detaches the sink (reasoned from
https://github.com/meowshed/meowg1k/pull/89#discussion_r2895914526 and
crates/meow-agent/src/engine.rs:504-511, low). After the first error the sink
sees nothing more of the run, so a renderer that failed once shows no ending
(from crates/meow-agent/src/engine.rs:504-515, high).

## What would reverse it

- A sink turns out to be part of the work, for example the only writer of the
  session log, so that detaching it loses a record the run needs (reasoned from
  crates/meow-star/src/run.rs:742-768, low).

## Consequences

`Delivery` wraps the sink, and after the first error it drops every later event
and reports that it wants no deltas (from
crates/meow-agent/src/engine.rs:486-516, high). The engine records no `Note`
for the failure: `Delivery` stores the error in `broken` and nothing reads it,
so the run's outcome and its events don't say the sink broke (from
crates/meow-agent/src/engine.rs:494-515, high). No sink outside the tests
returns an error: `Relay` in the Starlark runtime and `Collect` and `Discard`
in the engine all return `Ok(())`, so this path isn't reached today (from
crates/meow-star/src/run.rs:676-781 and
crates/meow-agent/src/event.rs:137-160, high).

## How I will know it was realised

1. `a_broken_sink_does_not_destroy_the_run` in crates/meow-agent/tests/spec.rs
   passes: the run finishes with its text and the sink is called once (from
   crates/meow-agent/tests/spec.rs:633-663, high).
2. A test sees a `Note` naming the sink's error after the sink fails. No such
   test exists, and the code doesn't emit one (from
   crates/meow-agent/tests/spec.rs:633-663 and
   crates/meow-agent/src/engine.rs:494-515, high).

## What this does not settle

- Where the `Note` goes when the sink that failed is the one that persists
  events. The Starlark runtime's sink both renders and keeps session events, so
  the `Note` can't travel through it (from crates/meow-star/src/run.rs:742-768,
  medium).
