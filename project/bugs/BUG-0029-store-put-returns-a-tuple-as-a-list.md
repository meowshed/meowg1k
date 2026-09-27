---
id: BUG-0029
artifact: bug
status: approved
severity: minor
violates: REQ-2466
found: 2026-09-27
revised: 2026-09-27
issue: 202
---

# `store.put` of a tuple comes back from `store.get` as a list

A tuple stored with `store.put` is read back as a list, because the value goes
through JSON on the way in.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Create a workspace whose command handler runs `put("t", (1, 2))` and writes
   `repr(get("t"))` and `type(get("t"))`, with `put` and `get` loaded from
   `@std//store`.
2. Run `meow trust`, then the command.

It prints `store=[1, 2] list`. It should print `(1, 2) tuple`. This was run
against `target/debug/meow`.

## What the system does

`store.put` at `crates/meow-star/src/modules.rs:1183-1192` converts the value
with `to_json_value`, which has no tuple, so a tuple becomes a JSON array and
`store.get` rebuilds it as a list.

## What it should do, and why

REQ-2466: "`store.get` MUST return the value that `store.put` was given, of the
same type, for any value a handler can build."

## Triage

The defect enters in the `@std//store` module's encoding. It is minor: the
elements survive, and only the type changes. The fix is to encode the Starlark
type beside the value, or to refuse a tuple at `put` with an error, which would
need REQ-2466 narrowed.

## Closed by

A test in `crates/meow-star/tests/running.rs`, named for example
`store_round_trips_a_tuple`, that expects `type(get("t")) == "tuple"`.
