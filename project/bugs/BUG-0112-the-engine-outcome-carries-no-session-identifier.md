---
id: BUG-0112
artifact: bug
status: approved
severity: minor
violates: REQ-1003
found: 2026-09-27
revised: 2026-09-27
issue:
---

# The engine's `Outcome` carries no session identifier

`meow_agent::Outcome` has the stop reason, the detail, the text, the value, the
usage and the steps, and no session identifier. Only the Starlark value
`meow-star` builds from it adds one.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Read `pub struct Outcome` at `crates/meow-agent/src/outcome.rs:26-43`. It
   has no session field.
2. Read `outcome` at `crates/meow-star/src/value.rs:136-195`, which takes the
   session identifier as a separate argument and adds it as `session`.

## What the system does

A Rust caller of `Engine::run` gets an outcome it can't tie to a session. The
Starlark value's `session` is the handler's session, passed in at
`crates/meow-star/src/value.rs:215`, whichever agent produced the outcome.

## What it should do, and why

REQ-1003: "An outcome MUST carry the step transcript, the usage totals, the
session identifier, and a detail string". The engine's `Outcome` should carry
the identifier of the session the run wrote to.

## Triage

The defect enters in `meow-agent`'s `Outcome`. The engine doesn't know about
sessions, which is by design, so the identifier would have to arrive as an
input to the run. It's minor: the user-facing Starlark outcome has the field,
and only Rust callers and tests of the engine lack it. If the engine is meant
never to know the session, REQ-1003 should place the identifier on the Starlark
outcome and this record closes by amending it.

## Closed by

A test in `crates/meow-agent/tests/spec.rs`, named for example
`an_outcome_names_its_session`, that runs with a given session identifier and
expects it on the outcome, or an amendment to REQ-1003.
