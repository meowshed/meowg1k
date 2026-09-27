---
id: ADR-0605
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2535]
supersedes: []
---

# 0605. `crypto`, `template` and `math` are left out of the module table

## Decision

The `@std//` table carries no `crypto`, `template` or `math` module (from
docs/design/0.3.0-starlark-api.md section 10, high).

Once this is accepted, it holds: the table is `csv`, `env`, `fs`, `git`,
`http`, `index`, `json`, `path`, `re`, `search`, `shell`, `store`, `text`,
`time`, `toml`, `xml` and `yaml` (from crates/meow-star/src/modules.rs:35-38,
high).

## Why

No automation use case justified `crypto`; Starlark's `%` operator and string
methods cover what `template` did; and `math` belongs in a `.star` library, not
in the runtime (from docs/design/0.3.0-starlark-api.md section 10, high). A
module nobody calls is a surface nobody checked (from
https://github.com/retran/meowg1k/pull/135, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep all three, as v0.2.x's `ctx.crypto` and its siblings did | A v0.2.x script that hashed, templated or computed ports without a rewrite (reasoned from docs/design/0.3.0-starlark-api.md section 2, low) | None of them had a use case in automation that Starlark or a library doesn't cover (from docs/design/0.3.0-starlark-api.md section 10, high) |
| Keep `math` as a runtime module and drop the other two | Numeric helpers are there without a `load` of a workspace file (reasoned from docs/design/0.3.0-starlark-api.md section 10, low) | Pure computation can be written in Starlark, so it belongs in a `.star` library (from docs/design/0.3.0-starlark-api.md section 10, high) |

## What it costs

A handler that needs a digest runs a program through `shell`, such as
`sha256sum`, and a workspace that needs numeric helpers writes or loads its own
`.star` library (reasoned from crates/meow-star/src/capability.rs, low).

## What would reverse it

- A command in `.meow/` or a shipped agent needs a digest or a template that
  Starlark's string methods can't produce (reasoned from
  docs/design/0.3.0-starlark-api.md section 10, low).

## Consequences

- `load("@std//crypto", ...)` fails as a name the table doesn't hold, with the
  closest name suggested (reasoned from
  docs/requirements/REQ-2527-errors-suggest-closest-name.md, low).

## How I will know it was realised

1. `NAMES` in crates/meow-star/src/modules.rs contains none of the three, and
   `Modules::build` inserts none of them (from
   crates/meow-star/src/modules.rs:35-60, high).

## What this does not settle

- Whether `ui` as a widget kit stays out: the same section drops it in favour
  of `ctx.out`, and that is a separate choice (from
  docs/design/0.3.0-starlark-api.md section 10, high).
