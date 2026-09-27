---
id: ADR-0424
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2834, REQ-2804]
supersedes: []
---

# 0424. The port behind `ctx.out` takes one typed event

## Decision

The port behind `ctx.out` takes one typed event rather than offering a method
per call, and the engine's own events go down the same channel (from
https://github.com/retran/meowg1k/pull/124, high).

Once this is accepted, the `Events` trait has one method, `event`, taking a
`ViewEvent`, and the ten `ctx.out` calls are the variants of `Output` (from
crates/meow-star/src/port.rs:17-30 and crates/meow-core/src/view.rs:192-245,
high). The plain and JSON renderers and the terminal renderer each take the
same `ViewEvent` stream (from crates/meow-ui/tests/renderers.rs:162-175,
high).

## Why

A trait with ten methods would let a renderer quietly handle nine (from
https://github.com/retran/meowg1k/pull/124, high). One channel lets a renderer
see one vocabulary instead of two (from
https://github.com/retran/meowg1k/pull/124, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A trait with a method per `ctx.out` call | Each call gets its own signature, and a renderer overrides only the calls it cares about (reasoned from https://github.com/retran/meowg1k/pull/124, low) | A renderer could quietly handle nine of the ten (from https://github.com/retran/meowg1k/pull/124, high) |
| One channel for `ctx.out` and another for the engine's events | Keeps what a script said apart from what the engine reported, so a consumer can take one without the other (reasoned from crates/meow-star/tests/running.rs:33-38, where a test filters the engine's events out, low) | A renderer would see two vocabularies that have to be kept level (from https://github.com/retran/meowg1k/pull/124 and crates/meow-star/src/port.rs:19-23, high) |
| Do nothing: keep v0.2.x's `ctx.ui` with 22 layout builtins | A script decides its own layout (from docs/spec/tui.md:204-206, medium) | Presentation is decided in userland and can't be fixed centrally, and three rendering stacks each own the cursor (from docs/spec/tui.md:199-206, high) |

## What it costs

A renderer has to name every live event, including the ones it ignores: the
plain renderer carries explicit arms that drop `Schema`, `Progress` and the
deltas (from crates/meow-ui/src/plain.rs:57 and crates/meow-ui/src/plain.rs:74,
high). A test that wants only what a handler said filters the engine's events
out of the one stream (from crates/meow-star/tests/running.rs:33-38, high).

## What would reverse it

- A renderer that needs a reply from the port, such as a value returned from
  a `ctx.out` call, which a one-way `fn event(&self, ViewEvent)` can't give
  (reasoned from crates/meow-star/src/port.rs:28-30, low).

## Consequences

- A variant added to `LiveKind` or `Output` fails to compile until the plain
  and terminal renderers handle it, because they match those two enums without
  a wildcard arm, and the JSON renderer serialises any variant it is given
  (from crates/meow-ui/src/plain.rs:55-120, crates/meow-ui/src/json.rs:55-70
  and a search of crates/meow-ui/src for `_ =>`, medium).
- The engine's events reach a renderer through the same port, mapped by
  `Relay` in `meow-star` (from crates/meow-star/src/run.rs:605-616, high).

## How I will know it was realised

1. `every_out_call_reaches_every_renderer` in
   crates/meow-ui/tests/renderers.rs sends each of the ten `Output` variants
   through all three renderers (from crates/meow-ui/tests/renderers.rs:279-281,
   high).
2. `ctx_out_has_exactly_ten_calls` in crates/meow-star/tests/running.rs and
   crates/meow-ui/tests/renderers.rs pins the number of calls at ten (from
   crates/meow-star/tests/running.rs:775-777 and
   crates/meow-ui/tests/renderers.rs:266-268, high).

## What this does not settle

- Persisted kinds travel as `ViewEvent::Logged(EventKind)`, and the plain and
  terminal renderers drop every `Logged` kind they don't name, so the
  compile-time check above covers `LiveKind` and `Output` and not `EventKind`
  (from crates/meow-ui/src/plain.rs:141-143 and
  crates/meow-ui/src/tty.rs:397, high).
- Where the mapping from engine events to view events lives; ADR-0426 settles
  that (from docs/adrs/ADR-0426-relay-lives-in-meow-star.md, high).
