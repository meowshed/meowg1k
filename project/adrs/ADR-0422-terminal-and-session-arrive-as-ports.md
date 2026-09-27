---
id: ADR-0422
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2445, REQ-2446]
supersedes: []
---

# 0422. The terminal, the session log and search arrive in `meow-star` as ports

## Decision

`out`, `ask`, `stdin` and `session` are ports rather than types (from
https://github.com/meowshed/meowg1k/pull/123, high). `search` is a port too,
because `meow-index` lives beside the Starlark runtime rather than under it
(from https://github.com/meowshed/meowg1k/pull/132, high).

Once this is accepted, `meow-star` declares five traits, `Events`, `Ask`,
`Stdin`, `Session` and `Search`, and `meow-cli` implements each one against the
real terminal, the session store and the index (from
crates/meow-star/src/port.rs:28-147, crates/meow-cli/src/ask.rs:106,
crates/meow-cli/src/ask.rs:232, crates/meow-cli/src/session.rs:118,
crates/meow-cli/src/render.rs:57 and crates/meow-cli/src/index.rs:172, high).
A run with no terminal or no index still works through the do-nothing ports in
`port::quiet` and through `Unindexed`, where `search.code` is what fails (from
crates/meow-star/src/port.rs:289-340, crates/meow-cli/src/index.rs:402 and
https://github.com/meowshed/meowg1k/pull/132, high).

## Why

The terminal and the session log live in crates that depend on `meow-star`, so
they arrive as traits (from https://github.com/meowshed/meowg1k/pull/123, high).
A capability is reached with `load` and not off the handler context, so the
index can't be a context member either (from
https://github.com/meowshed/meowg1k/pull/132, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Concrete types for the terminal and the session log | No trait to implement and no dynamic dispatch between a handler and the terminal (reasoned from crates/meow-star/src/port.rs:28-97, low) | They live in crates that depend on `meow-star` (from https://github.com/meowshed/meowg1k/pull/123, high) |
| Do nothing: let the Starlark runtime import the adapters directly, as v0.2.x did | One import and no seam, which is how `internal/core/starlark` reached `internal/adapters/gateway` in v0.2.x (from docs/design/0.3.0-architecture.md:138-139, high) | The architecture forbids reproducing that broken boundary (from docs/design/0.3.0-architecture.md:138-139, high), and the handler context would need a terminal to be tested (from https://github.com/meowshed/meowg1k/pull/123, high) |
| Make search a member of the handler context | A handler would reach the index without a `load` line (reasoned from crates/meow-star/src/port.rs:142-147, low) | `R-STAR-021` says a capability is reached with `load` and not off the handler context (from https://github.com/meowshed/meowg1k/pull/132, high) |

## What it costs

Every frontend that runs a handler implements five traits, and a test that
drives one writes its own fakes: `running.rs` carries `Recorder`, `Willing`,
`Piped` and `FakeIndex`, and `approval.rs` carries three more (from
crates/meow-star/tests/running.rs:33-201 and
crates/meow-star/tests/approval.rs:30-86, high). The `Search` port reports a
failure as a `String`, so a caller can't match on what went wrong (from
crates/meow-star/src/port.rs:154-217, high). `Session::record` returns nothing,
so a failed write to the log never reaches the handler, and `meow-cli` drops
the error (from crates/meow-star/src/port.rs:113 and
crates/meow-cli/src/session.rs:141, high).

## What would reverse it

- The terminal, the session log or the index moving into a crate that
  `meow-star` depends on, which removes the dependency direction that forces a
  trait (reasoned from https://github.com/meowshed/meowg1k/pull/123 and
  crates/meow-star/Cargo.toml, low).

## Consequences

A test drives a handler without a terminal, which is most of what made the run
phase testable at all (from https://github.com/meowshed/meowg1k/pull/123, high).
`meow-star` depends on no terminal, store or index crate: its dependencies are
`meow-agent`, `meow-core`, `meow-llm` and `meow-policy` (from
crates/meow-star/Cargo.toml, high).

## How I will know it was realised

1. `the_context_has_six_members_and_cancelled` and `every_context_member_works`
   in crates/meow-star/tests/running.rs run a handler against fake ports and
   assert on what it wrote (from crates/meow-star/tests/running.rs:375-447,
   high).
2. `a_capability_is_loaded_rather_than_a_context_member` in the same file
   reaches a capability through `load` (from
   crates/meow-star/tests/running.rs:449-451, high).
3. `cargo tree -p meow-star -e normal --depth 1` lists `meow-agent`,
   `meow-core`, `meow-llm` and `meow-policy` and no other workspace crate
   (from running that command on 2026-09-27, high).

## What this does not settle

- What the port behind `ctx.out` carries; ADR-0424 settles that it takes one
  typed event (from docs/adrs/ADR-0424-ctx-out-port-takes-one-typed-event.md,
  high).
- How `ctx.ask` behaves without a terminal; ADR-0121 settles that (from
  docs/adrs/ADR-0121-ask-fails-without-a-terminal.md, high).
