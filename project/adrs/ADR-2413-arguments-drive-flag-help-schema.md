---
id: ADR-2413
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2506, REQ-2511]
supersedes: []
---

# 2413. `meow.arg` with real types replaces `meow.param`

## Decision

`meow.param` becomes `meow.arg` with real types, and one argument declaration
drives the command-line flag, the help text and the model schema (from
docs/spec/starlark.md, Changes from v0.2.x [R-STAR-061], high).

## Why

One declaration producing the flag, the help and the JSON Schema means the three
can't drift (from docs/spec/starlark.md [R-STAR-061], high). In v0.2.x the
tool's `Params` fed the model schema while each command carried a separate
`FlagDef` copy that fed the flags (from
v0.2.1:internal/core/starlark/registry.go:34-64 and
v0.2.1:cmd/starlark.go:81-110, medium).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Do nothing: keep the v0.2.x `meow.param` | Existing workspaces keep working, and one constructor takes the type as a string (from v0.2.1:internal/core/starlark/registry.go:16-30, high). | The flag came from a `FlagDef` copy and the schema from the `Param`, so the two could drift (from v0.2.1:internal/core/starlark/registry.go:34-64, medium), and an absent parameter was filled with its type's zero value (from docs/design/0.3.0-starlark-api.md, section 6.1, high). |
| Store each argument as a finished JSON Schema | The model's schema is ready without a conversion (reasoned from https://github.com/meowshed/meowg1k/pull/121, low). | The flag, the help line and the schema have to come from the same node or they drift, so an argument is stored as its declaration (from https://github.com/meowshed/meowg1k/pull/121, high). |

Neither the code, the history nor the design documents name a further option.

## What it costs

- Every v0.2.x script that calls `meow.param` breaks, deliberately (from
  docs/design/0.3.0-starlark-api.md, section 11, high).
- The `pattern` a string argument accepts is `*` and literals, not a regular
  expression, because a full engine would be another dependency and a crafted
  input could make a declaration run long (from
  https://github.com/meowshed/meowg1k/pull/121, high).

## What would reverse it

If one argument has to look different on the command line and in the model's
schema, such as a flag the model must not see, one shared node can't express it
(reasoned from crates/meow-star/src/args.rs:11-18, low).

## Consequences

- An argument is one JSON Schema node with `x-meow` annotations, and the flag,
  the help and the schema are three readers of it (from
  crates/meow-star/src/args.rs:4-18, high).
- A missing required argument is never invented: the tool isn't invoked, and the
  engine returns a correction to the model (from
  docs/design/0.3.0-starlark-api.md, section 6.1, high).

## How I will know it was realised

The tests `one_declaration_produces_the_flag_the_help_and_the_schema` and
`every_argument_type_builds_with_its_constraints` in
crates/meow-star/tests/arguments.rs pass (high).

## What this does not settle

- Positional arguments, which REQ-2512 and REQ-2513 cover (from
  docs/spec/starlark.md [R-STAR-062], high).
- Enforcing a constraint on both paths, which REQ-2514 covers (from
  docs/spec/starlark.md [R-STAR-063], high).
