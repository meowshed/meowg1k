---
id: BUG-0107
artifact: bug
status: approved
severity: major
violates: REQ-1055
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `meow.parallel` runs its invocations one after another

`meow.parallel([...])` calls `run_agent` for each invocation in a loop and
never uses the engine's `run_parallel`, so a Starlark fan-out takes the sum of
its branches' time.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare an agent whose scripted model takes one second to answer.
2. In a handler, call `meow.parallel([a.call("x"), a.call("y"), a.call("z")])`.
3. Time the call. It takes about three seconds, not one.

This follows from the code below.

## What the system does

`crates/meow-star/src/value.rs:308-322` loops over the invocations and blocks on
`run_agent` for each, with a comment "Sequentially for now". `run_parallel` in
`crates/meow-agent/src/nested.rs:151` runs branches at once and gives each a
child ledger, and nothing in `meow-star` calls it. One branch that fails to
build also ends the whole call, because of the `?` at `value.rs:318`.

## What it should do, and why

REQ-1055: "The engine MUST accept a list of agent invocations and run them
concurrently, returning results in the order given." The engine can, and
`meow.parallel` is the only way a workspace reaches it, so the concurrency
should hold there too. REQ-1059 asks that new invocations stop once the budget
is spent, which `run_parallel` already does.

## Triage

The defect enters in `meow.parallel` in `meow-star`. ADR-0414 records it. It's
major because the requirement's main behaviour, concurrency, doesn't happen on
the path a user has; the order guarantee holds.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`parallel_runs_its_branches_at_once`, that drives three branches whose scripted
provider waits on a shared barrier of three, which can only release if all
three are running at once.
