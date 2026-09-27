---
id: ADR-0469
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2400, REQ-2843]
supersedes: []
---

# 0469. User commands are Starlark files, not compiled workflows

## Decision

Workflows moved from Go code to user-editable Starlark scripts under the
workspace (from https://github.com/retran/meowg1k/pull/80, high). The rewrite
keeps the choice: a workspace's commands are declared in Starlark under `.meow/`
and reached at the top level (from https://github.com/retran/meowg1k/pull/126,
high).

Once this is accepted, `meow` loads `.meow/` first and builds its command
line from what the workspace declares, so `meow greet world` runs a command a
workspace declared with `meow.command(meow.tool(...))` and `meow --help` lists
it (from https://github.com/retran/meowg1k/pull/126 and
crates/meow-cli/tests/surface.rs:105, high). This repository's own `review`,
`commit` and `ask` are declared that way in `.meow/meow.star` (from
.meow/meow.star:167, high). `meow run <name>` ignores its trailing arguments, so
only the direct form takes typed flags (from
https://github.com/retran/meowg1k/pull/126 and crates/meow-cli/src/wire.rs:416,
high).

## Why

Users can write custom commands without rebuilding the binary (from
https://github.com/retran/meowg1k/pull/80, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: workflows compiled into the binary, as the Go tree's `internal/flows/` and `internal/activities/` were | The compiler checks every workflow, and nothing is evaluated at start-up (reasoned from https://github.com/retran/meowg1k/pull/80, low) | Users can't change a command without rebuilding the binary (from https://github.com/retran/meowg1k/pull/80, high) |
| Commands as YAML tasks, as v0.2.0 had them | A data format needs no interpreter and can't run code (reasoned from https://github.com/retran/meowg1k/pull/80, low) | `.meow/meow.star` is a program, not a data format that grew expressions (from docs/philosophy.md section 6, medium) |

The code and the history name no third option.

## What it costs

A `.meow/` directory is executable code with tool access, so the first run in a
workspace has to show what it declares and ask before running it (from
docs/requirements/REQ-1222-untrusted-workspace-shows-declarations.md, high).
The command line can't be built until the workspace has loaded, so a broken
`.meow/` is reported before anything else and only `version`, `completions` and
`init` work without one (from https://github.com/retran/meowg1k/pull/126,
high). Shell completions cover the built-in commands only, because a user's
commands depend on the workspace the shell is in (from
crates/meow-cli/src/wire.rs:948, high).

## What would reverse it

A command that Starlark can't express, because a runtime module can't give it
what it needs, would have to go back into the binary (reasoned from
crates/meow-star/src/modules.rs, low). Evidence that users don't edit their
commands, only run the ones shipped, would remove the reason (reasoned from
https://github.com/retran/meowg1k/pull/80, low).

## Consequences

- A declared command's arguments become flags described by the same
  declaration that drives the model's schema (from
  https://github.com/retran/meowg1k/pull/126, high).
- The Starlark API is the product, and the Rust code exists to serve it (from
  CLAUDE.md principle `starlark_api_is_the_product`, high).
- The workspace root is the nearest directory holding `.meow/meow.star`, and
  nothing outside it is merged in (from
  crates/meow-star/tests/loading.rs:49, high).

## How I will know it was realised

`a_declared_command_needs_no_prefix` in `crates/meow-cli/tests/surface.rs`
passes: a workspace declares `greet`, and `meow greet world` prints
`hello, world` and `meow --help` lists it (from
crates/meow-cli/tests/surface.rs:105, high).
`discovery_takes_the_nearest_ancestor_and_merges_nothing` in
`crates/meow-star/tests/loading.rs` passes (from
crates/meow-star/tests/loading.rs:49, high).

## What this does not settle

- `meow run <name>` ignores its trailing arguments, and giving it the same
  parsing means building a second parser for one declaration (from
  https://github.com/retran/meowg1k/pull/126, high).
- Whether completions should include a workspace's commands (from
  crates/meow-cli/src/wire.rs:948, high).
