---
id: BUG-0133
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `meow run <name>` ignores the arguments after the name

`meow run` accepts trailing arguments, and its help calls them "Its
arguments", but it invokes the command with an empty argument map, so every
flag is silently dropped and defaults are used.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments; `cargo build -p meow-cli`, `HOME` set to
a scratch directory.

1. Create `.meow/meow.star`:

   ```python
   def greet(ctx):
       ctx.out.write("hello %s" % ctx.args.who)
       return "true"
   meow.command(meow.tool(name = "greet", about = "s", run = greet, args = {
       "who": meow.arg.string(about = "who", required = False, default = "nobody"),
   }))
   ```

2. Run `meow trust`, then `meow greet --who world`. It prints `hello world`.
3. Run `meow run greet --who world`. It prints `hello nobody` and exits 0.

## What the system does

`surface.rs` declares a `rest` argument with `trailing_var_arg`,
`crates/meow-cli/src/surface.rs:102-111`. `with_workspace` calls `invoke` with
`&Map::new()` for `run`, `crates/meow-cli/src/wire.rs:416-419`, and never reads
`rest`.

## What it should do, and why

No requirement covers `meow run`, which is a gap for the requirements step.
ADR-0469 records that only the direct form takes typed flags. `meow run` should
parse its trailing arguments against the command's declaration, or refuse
them, and not run with defaults the person didn't choose.

## Triage

The defect enters in `with_workspace`. It's minor because the direct form
works, but a script using `meow run` gets a run with other arguments than it
passed and a success exit code.

## Closed by

A test in `crates/meow-cli/tests/surface.rs`, named for example
`meow_run_passes_its_arguments`, running step 3 and expecting `hello world`.
