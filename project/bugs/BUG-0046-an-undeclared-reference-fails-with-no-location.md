---
id: BUG-0046
artifact: bug
status: approved
severity: minor
violates: REQ-2526
found: 2026-09-27
revised: 2026-09-27
issue: 219
---

# A reference to an undeclared name fails with no file, line or column

The checks that run after every file is loaded report a missing agent, model,
tool or command target as `StarError::Unknown`, which holds a kind, a name and a
suggestion and no location. The registry records where each reference was made
and discards it.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Create a workspace whose `.meow/meow.star` declares a tool, commands it, and
   then calls `meow.command("nothere")`.
2. Run `meow check` with `HOME` pointed at a scratch directory.

It prints ``tool or agent `nothere` is not declared`` and exits 7, with no file
or line. A runtime-module refusal in the same file prints a traceback with
`meow.star:2:1`. This was run against `target/debug/meow`.

## What the system does

`StarError::Unknown` at `crates/meow-star/src/error.rs:57-66` has no location
field. `crates/meow-star/src/registry.rs:423-433` iterates `(name, origin)` for
agent references and discards the origin with `let _ = origin;`, and the model
and command checks around it build the same error.

## What it should do, and why

REQ-2526: "Every load-time and run-time error MUST carry the file, line, and
column of the Starlark expression that caused it." ADR-0415 records the
decision.

## Triage

The defect enters in `meow-star`'s registry, whose deferred checks run after
evaluation has ended and so have no traceback to attach. It is minor: the name
is given and a search finds it, but a large `.meow/` with a markdown agent
naming a missing one leaves the user to look in every file. The fix is to carry
the recorded origin into the error.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`an_undeclared_reference_names_where_it_was_made`, that expects the file and
line of the `meow.command` call in the error.
