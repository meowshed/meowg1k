---
id: ADR-0610
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2536]
supersedes: []
---

# 0610. An explicit `None` for an optional declaration argument means absent

## Decision

A `meow` declaration builtin reads an explicit `None`, given for an optional
keyword argument whose default is absent, as if the argument were left out
(from https://github.com/retran/meowg1k/pull/126, high).

Once this is accepted, it works for `meow.provider`'s `api_key` and `base_url`
and `meow.model`'s `kind` through `NoneOr` with a `None` default, and for
`temperature`, the agent's nested fields and an argument's `default`, `min` and
`max` through `present`, which drops a `None` (from
crates/meow-star/src/declare.rs:74-76, :158-159 and :189-277, high). An
argument with a default of its own, such as `required = True`, still refuses
`None` as the wrong type (from crates/meow-star/src/declare.rs:421, medium).

## Why

Starlark can't leave a keyword argument out conditionally, so
`api_key = get("ANTHROPIC_API_KEY")` passes `None` when the variable is unset,
and that is the line `meow init` writes; it failed with a type error until
this changed (from https://github.com/retran/meowg1k/pull/126 and
crates/meow-cli/src/wire.rs:984-989, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: an explicit `None` is a type error | A mistyped value can't pass silently as absent (reasoned from crates/meow-star/src/declare.rs:156-160, low) | The line `meow init` generates failed to load whenever the key was unset (from https://github.com/retran/meowg1k/pull/126, high) |
| Leave the builtins strict and make every declaration file write an `if` around the argument | The builtin's signature says exactly what it takes (reasoned from https://github.com/retran/meowg1k/pull/126, low) | Every declaration file grows an `if` for each optional value it reads from the environment (from https://github.com/retran/meowg1k/pull/126, high) |

## What it costs

A declaration that meant to pass a value and passed `None` by mistake loads as
if it had left the argument out, with no error at the line (reasoned from
crates/meow-star/src/declare.rs:74-76, low).

## What would reverse it

- Starlark gains a way to omit a keyword argument conditionally (reasoned from
  https://github.com/retran/meowg1k/pull/126, low).

## Consequences

- An unset key reaches the credential chain, which then tries the global store
  and the provider's variable in turn (from
  docs/requirements/REQ-1200-credential-resolution-order.md, medium).

## How I will know it was realised

1. `.meow/meow.star` loads with `ANTHROPIC_API_KEY` unset, since it passes
   `get("ANTHROPIC_API_KEY")` as `api_key` (from .meow/meow.star:17-21, high).
   No test asserts this directly; a test in crates/meow-star/tests/loading.rs
   that declares `api_key = None` and loads would.

## What this does not settle

- Whether arguments with a default of their own, such as `required`, should
  accept `None` too (from crates/meow-star/src/declare.rs:421, high).
