---
id: ADR-2405
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2466, REQ-2467]
supersedes: []
---

# 2405. The store holds values and not text

## Decision

`store.get` returns the value `store.put` was given, of the same type, for any
value a handler can build (from docs/spec/starlark.md [R-STAR-027], high).

## Why

A store that took strings would make every handler encode on the way in and
decode on the way out, and the two halves would be written in different places
and drift (from docs/spec/starlark.md, Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| A store that takes strings | The stored bytes are whatever the handler wrote, so no encoding change in meowg1k can make an old row unreadable (reasoned from crates/meow-cli/src/keep.rs `Durable::get`, low). | Every handler encodes on the way in and decodes on the way out, and the two halves drift (from docs/spec/starlark.md, Decisions, high). |
| Fall back to an in-memory map when the workspace database won't open | A handler keeps working within one run (from https://github.com/retran/meowg1k/pull/143, high). | A handler would write a value, read it back within the run, and find it gone next time with nothing explaining why; every call fails saying so instead (from https://github.com/retran/meowg1k/pull/143, high). |
| Do nothing: keep state in `ctx.session` | No new module (reasoned from crates/meow-star/src/port.rs `Keep`, low). | `ctx.session` is the place for what one run decided and the wrong place for what every run should remember, because collecting sessions removes it (from crates/meow-star/src/port.rs `Keep`, high). |

## What it costs

The encoding becomes the `store` module's problem (from docs/spec/starlark.md,
Decisions, high). The encoding is JSON, so a value JSON can't carry changes
type: a tuple put in comes back as a `list` (from `target/debug/meow` built on
2026-09-27, and crates/meow-star/src/modules.rs `store.put`, which calls
`to_json_value`, high). A row written by a future version that encodes
differently fails to decode, and `get` names the key it couldn't read (from
https://github.com/retran/meowg1k/pull/143 and crates/meow-cli/src/keep.rs
`Durable::get`, high).

## What would reverse it

A handler that needs to store bytes JSON can't hold, or to share the table with
a program outside meowg1k that expects its own format, would favour storing
text (reasoned from crates/meow-cli/src/keep.rs, low).

## Consequences

A handler tells "absent" from "stored `None`" by passing `default` to `get`
(from https://github.com/retran/meowg1k/pull/143, high). The table belongs to
the workspace and not to a session, so `meow session gc` leaves it alone (from
crates/meow-star/src/port.rs `Keep`, high).

## How I will know it was realised

`a_value_survives_the_store_unchanged` and
`an_absent_key_gives_the_default_the_caller_named` in
crates/meow-star/tests/running.rs pass (from those tests, high). The first
covers `int`, `string`, `bool`, `list`, `dict` and `None`, and no test covers a
`tuple` or a `float` (from that test, high).

## What this does not settle

Migrating a stored value when the encoding changes: there is nothing yet to
migrate from (from https://github.com/retran/meowg1k/pull/143, high). Which
Starlark types count as "any value a handler can build": a tuple doesn't keep
its type today, which [R-STAR-027] doesn't allow (from the probe above, high).
