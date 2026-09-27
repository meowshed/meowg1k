---
id: ADR-0452
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2458, REQ-2546]
supersedes: []
---

# 0452. `search.code` is `index.query` with a floor of zero

## Decision

`index.query` is `search.code` with the knobs, and `code` is `query` with a
floor of zero (from https://github.com/retran/meowg1k/pull/142, high).

With this in place, the binary's `Searcher::code` calls `Searcher::query` with
`min_score` 0.0, and `index.query` takes `min_score` as an integer or a float
(from crates/meow-cli/src/index.rs:172-174 and
crates/meow-star/src/modules.rs:916-925, high). The sharing is inside each
implementation and not in the port: `Search` declares `code` and `query` as
two required methods, and `search.code` calls `code` (from
crates/meow-star/src/port.rs:154-171 and crates/meow-star/src/modules.rs:811,
high).

## Why

With one implementation the two can't drift (from
https://github.com/retran/meowg1k/pull/142, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Two implementations | Each call can be tuned on its own (reasoned from crates/meow-star/src/port.rs:147-171, low) | They can drift (from https://github.com/retran/meowg1k/pull/142, high) |
| Do nothing: `search.code` only, with no floor | One call to learn (reasoned from crates/meow-star/src/port.rs:156-160, low) | [R-STAR-024] asks `query` to take the floor below which a hit is not worth returning (from docs/spec/starlark.md [R-STAR-024], high) |

The source names no third option.

## What it costs

Every `Search` implementor has to write the delegation from `code` to `query`
itself, because the port does not give `code` a default body (from
crates/meow-star/src/port.rs:154 and crates/meow-cli/src/index.rs:172-174,
high).

## What would reverse it

This is reversed if `search.code` needs a behaviour `query` doesn't have, for
example a default floor above zero, so that the two stop being one call with
different arguments (reasoned from crates/meow-star/src/port.rs:156-160, low).

## Consequences

- A handler moves from `search.code` to `index.query` by adding arguments, and
  gets the same hits at a floor of zero (from
  crates/meow-star/src/port.rs:156-160, high).
- `min_score = 0` and `min_score = 0.4` both work (from
  https://github.com/retran/meowg1k/pull/142, high).

## How I will know it was realised

`query_carries_the_floor_and_code_is_the_same_call_without_one` in
`crates/meow-star/tests/running.rs` passes (from
crates/meow-star/tests/running.rs:2593, high). That test drives `FakeIndex`,
whose own `code` delegates to `query`, so it shows the Starlark side passes the
floor and not that the binary's `Searcher` delegates (from
crates/meow-star/tests/running.rs:201-203, high).

## What this does not settle

- Whether the delegation should move into the port as a default method, so an
  implementor can't make `code` and `query` differ; today nothing stops one
  from doing so (reasoned from crates/meow-star/src/port.rs:154-171, low).
