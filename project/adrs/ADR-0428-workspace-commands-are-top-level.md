---
id: ADR-0428
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2843]
supersedes: []
---

# 0428. A workspace's agents and tools are top-level commands

## Decision

A workspace's agents and tools are top-level commands with no prefix: `meow
review`, not `meow run review` (from https://github.com/meowshed/meowg1k/pull/126,
high). `run` exists for a name that would otherwise be ambiguous (from
https://github.com/meowshed/meowg1k/pull/126, high).

Once this is accepted, `meow greet world` reaches a declared tool directly and
its declared arguments are flags described by the same declaration (from
crates/meow-cli/tests/surface.rs `a_declared_command_needs_no_prefix` and
`a_declared_argument_becomes_a_described_flag`, high). Two things don't work
yet. `meow run <name>` drops its trailing arguments, because the binary calls
the command with an empty argument map (from crates/meow-cli/src/wire.rs:416,
high). A declared command named `pkg` isn't refused at load, because the
reserved list in `meow-star` omits the `pkg` group the binary adds (from
crates/meow-star/src/registry.rs:17 against crates/meow-cli/src/surface.rs:32,
high).

## Why

A user's agents and tools are what the tool is for, so they take the top level
and the built-ins are grouped under nouns to keep that namespace clear (from
docs/design/0.3.0-tui.md section 2.1, high). A command at the top level with
its own `--help` also answers "what can this workspace do": `meow --help` lists
every declared command (from
https://github.com/meowshed/meowg1k/issues/60#issuecomment-5750552217, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep v0.2.x's registration, which added every Starlark command to the root with no check (from `git show v0.2.1:cmd/starlark.go`, lines 67 to 75, medium) | Needs no reserved-name list, so nothing has to be kept in step with the built-ins (reasoned from the same lines, low) | A user command could shadow a built-in or be shadowed by one; the design refuses the collision at load in both directions instead (from docs/design/0.3.0-tui.md section 2.1, high) |
| A prefix command, `meow run review` | Built-ins and user commands share no namespace, so no name is reserved and no collision is possible (reasoned from crates/meow-star/src/registry.rs:17, low) | User commands are what the tool is for and belong at the top level (from docs/design/0.3.0-tui.md section 2.1, high); the prefix survives only for a name that would otherwise be ambiguous (from https://github.com/meowshed/meowg1k/pull/126, high) |
| `meow run <path.star or file.md>` runs an agent or script that isn't registered | Runs a file without declaring it in `.meow/` (from docs/design/0.3.0-tui.md section 2.2, high) | The command line gave `run` a declared name instead (from https://github.com/meowshed/meowg1k/pull/126, high); no source records why the path form was dropped |

## What it costs

`meow run <name>` ignores its trailing arguments, because giving it the same
argument parsing means building a second parser for the same declaration (from
https://github.com/meowshed/meowg1k/pull/126, high). Thirteen names are closed to
workspaces, and the list that closes them lives in `meow-star` apart from the
lists the binary builds its surface from, so the two can drift; they already
have, over `pkg` (from crates/meow-star/src/registry.rs:17 and
crates/meow-cli/src/surface.rs:19 and 32, high).

## What would reverse it

- A new built-in has to take a top-level name that existing workspaces
  already declare. The collision rule would then break those workspaces at
  load, and only a prefix avoids that (reasoned from
  crates/meow-star/tests/loading.rs `a_command_may_not_take_a_builtin_name`,
  low).

## Consequences

- The command surface is built after the workspace loads, because what the
  command line accepts depends on what the workspace declares, so a broken
  `.meow/` is reported before anything else except `version`, `completions`
  and `init` (from https://github.com/meowshed/meowg1k/pull/126, high).
- `meow-star` keeps a reserved list and refuses a command that collides with a
  built-in at load (from crates/meow-star/src/registry.rs:17 and
  crates/meow-star/tests/loading.rs `a_command_may_not_take_a_builtin_name`,
  high).
- A tool's declared arguments become clap flags built from the declaration
  that drives the model's schema, and an agent takes its task as trailing words
  (from crates/meow-cli/src/surface.rs, function `declared`, high).

## How I will know it was realised

1. crates/meow-cli/tests/surface.rs `a_declared_command_needs_no_prefix`
   passes: `meow greet world` exits 0 and prints the greeting, and `meow
   --help` lists `greet` (from that test, high).
2. crates/meow-star/tests/loading.rs `a_command_may_not_take_a_builtin_name`
   passes, and a command named after each entry in `GROUPS`, `pkg` included, is
   refused at load; no test covers `pkg` today (from
   crates/meow-cli/src/surface.rs:32, high).

## What this does not settle

- Which name is "ambiguous" enough to need `run`. Collisions with built-ins are
  refused at load, so no declared name can clash with one, and no source names
  the case `run` serves (reasoned from crates/meow-star/src/registry.rs:17,
  low).
- Whether `meow run` takes a file path, as docs/design/0.3.0-tui.md section 2.2
  says, or a declared name, as the code does (from
  crates/meow-cli/src/wire.rs:416, high).
- How `meow run <name>` parses the declared arguments (from
  https://github.com/meowshed/meowg1k/pull/126, high).
