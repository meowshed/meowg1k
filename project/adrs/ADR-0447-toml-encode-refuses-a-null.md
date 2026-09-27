---
id: ADR-0447
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2432, REQ-2529]
supersedes: []
---

# 0447. `toml.encode` refuses a null rather than dropping the key

## Decision

`toml.encode` refuses a null rather than dropping the key (from
https://github.com/retran/meowg1k/pull/141, high).

Once accepted, a handler that encodes a null to TOML gets an error it can act
on, not a file missing a key (from crates/meow-star/src/modules.rs:258-262,
high). REQ-2432 was amended on 2026-09-27 to require round-tripping only
values a format can represent. What still doesn't hold: no test covers the
refusal (from crates/meow-star/tests, medium).

## Why

TOML has no null, and silently losing a key is worse than an error a handler can
act on (from https://github.com/retran/meowg1k/pull/141, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: drop a key whose value is null, as v0.2.x's `toml.stringify` did by handing the value to `BurntSushi/toml` | Any dict a handler builds encodes, including one read from JSON with a null in it (reasoned from v0.2.1:internal/core/starlark/module_toml.go:46, low) | A key disappears silently (from https://github.com/retran/meowg1k/pull/141, high) |
| Write a null as an empty string | The key survives with a value every TOML reader accepts (reasoned from crates/meow-star/src/modules.rs:258, low) | The source doesn't weigh it as a default; the code leaves that choice to the handler, which "can encode `""` or omit the key itself" (from crates/meow-star/src/modules.rs:258, high) |

A third option isn't named by the code, the history or the design documents,
so the table stops at two.

## What it costs

A handler that reads JSON or YAML carrying a null and writes it as TOML fails
at the call, and has to remove or replace each null itself before
`toml.encode` accepts the value (from crates/meow-star/src/modules.rs:260,
high; the handler's work reasoned from the same line, low). A null nested in a
list fails the same way, since the conversion is recursive (from
crates/meow-star/src/modules.rs:274, high).

## What would reverse it

TOML gaining a null value, or the target serialiser offering a representation
of one that round-trips back to `None` through `toml.parse`, which would give
the key somewhere to go (reasoned from crates/meow-star/src/modules.rs:258,
low).

## Consequences

- `toml.encode({"a": None})` fails with "TOML has no null, so a key with no
  value cannot be encoded" (from crates/meow-star/src/modules.rs:260, high).
- `toml.encode` is the one encoder of the three that refuses a value the others
  produce, and REQ-2432 now carries that exception in its wording (from
  docs/requirements/REQ-2432-encode-accepts-other-formats-values.md, high).

## How I will know it was realised

A test calling `toml.encode` on a dict holding `None` fails with the "TOML has
no null" message. No such test exists yet in
`crates/meow-star/tests/running.rs`; the encoder tests there cover a top level
that isn't a table and the cross-format round trip, not a null (from grep of
crates/meow-star/tests for "no null", high).

## What this does not settle

- REQ-2432 now reads "any value the other two produce that its own format can
  represent", so the refusal is within it; `R-STAR-016` in `docs/spec/` still
  says each `encode` accepts any value the other two produce, without the
  exception (from docs/spec/starlark.md `R-STAR-016`, high).
- Whether a refused top level that isn't a table, which TOML also can't
  represent, falls under the same exception; the code refuses it for the same
  reason (from crates/meow-star/src/modules.rs:211, high).
