---
id: ADR-0604
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2534]
supersedes: []
---

# 0604. `env.require` fails at load, naming the variable

## Decision

`env.require(name)` fails while `.meow/` is evaluated, with the variable's name,
when the variable is unset (from docs/design/0.3.0-starlark-api.md section 4,
high).

Once this is accepted, it works: `env` is one of the two modules callable
during declaration, and `require` returns an error saying "`NAME` is not set in
the environment" (from crates/meow-star/src/modules.rs:119-125 and
docs/requirements/REQ-2524-declaration-callable-load-and-env.md, high).

## Why

A load-time failure with the name is better than an empty key and a 401 forty
seconds into the first run (from docs/design/0.3.0-starlark-api.md section 4,
high). "Missing credentials" without a name is a support ticket (from
crates/meow-star/src/modules.rs:121, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: offer only `env.get`, which returns `None` or a default | A workspace that declares three providers loads when only one key is set, and the missing one fails only when used (from https://github.com/retran/meowg1k/pull/126, high) | Where the key is always needed, the failure arrives at the first request, far from the line that caused it (from docs/design/0.3.0-starlark-api.md section 4, high) |
| Keep keys out of `.meow/` and read them from the global credential store | Keys belong to the machine and not the repository (from docs/design/0.3.0-starlark-api.md section 4.1, high) | This is the normal path and stays so; `env.require` is the escape hatch for a key that must come from the environment (from docs/design/0.3.0-starlark-api.md section 4.1, high) |

## What it costs

A workspace that calls `env.require` doesn't load at all without the variable,
including for commands that never reach that provider, such as `meow check`
(reasoned from crates/meow-star/src/modules.rs:123, low).

## What would reverse it

- Declarations start loading lazily, per command, so an unused provider's key
  is never read (reasoned from
  docs/requirements/REQ-2523-declaration-file-has-no-effects.md, low).

## Consequences

- `meow init` and `.meow/meow.star` use `env.get` for the provider key, so they
  load without one and the credential chain decides (from
  crates/meow-cli/src/wire.rs:984 and .meow/meow.star:7, high).

## How I will know it was realised

1. A test in crates/meow-star/tests/loading.rs loads `env.require("UNSET")` and
   gets a load error naming `UNSET`. No such test exists yet; the nearest is
   the one that allows `@std//env` during declaration (from
   crates/meow-star/tests/loading.rs:354, high).

## What this does not settle

- Whether an empty variable counts as set: `std::env::var` returns it, so
  `require` accepts `""` (from crates/meow-star/src/modules.rs:124, high).
