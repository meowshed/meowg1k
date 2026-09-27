---
id: BUG-0103
artifact: bug
status: approved
severity: major
violates: [REQ-2480, REQ-2481]
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A workspace command named `pkg` or `help` panics the binary

`RESERVED` in `meow-star` omits `pkg` and `help`, so a workspace may declare a
command with either name. The binary then panics while building its command
line, for every subcommand, including `meow check`.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments; debug build from
`cargo build -p meow-cli`.

1. Create `.meow/meow.star`:

   ```python
   def _go(ctx):
       ctx.out.note("user pkg ran")
       return "true"
   meow.command(meow.tool(name = "pkg", about = "user pkg", run = _go))
   ```

2. Run `meow check`.

## What the system does

It panics with ``Command meow: command name `pkg` is duplicated`` from clap's
debug assertions and exits 101. `meow pkg list` and `meow run pkg` panic the
same way. Renaming the command to `help` gives the same panic naming `help`.
`RESERVED` at `crates/meow-star/src/registry.rs:17-31` lists `auth`, `check`,
`completions`, `doctor`, `index`, `init`, `models`, `policy`, `providers`,
`run`, `session`, `trust` and `version`; the binary also owns `pkg`,
`crates/meow-cli/src/surface.rs:292`, and clap adds `help`. A release build
skips clap's debug assertions, so there the collision isn't refused at all.

## What it should do, and why

REQ-2480: "A command whose name collides with a built-in MUST fail at load time
naming the collision." REQ-2481: it "MUST NOT shadow the built-in." Loading
should fail with the collision named, and `meow check` should report it.

## Triage

The defect enters in the reserved list, which is written by hand apart from the
command table in `surface.rs`, so the two drifted when `meow pkg` arrived. It's
major because the requirement's refusal doesn't happen: the debug binary
crashes and the release binary is left to clap's behaviour. The fix is to
derive the reserved names from `surface::build(None)` or to test that they
match.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`every_built_in_name_is_reserved`, that declares a command under each built-in
subcommand name, `help` included, and expects `meow check` to fail naming it.
