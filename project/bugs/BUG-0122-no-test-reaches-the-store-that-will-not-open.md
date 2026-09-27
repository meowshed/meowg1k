---
id: BUG-0122
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# No test reaches the `store` that won't open, and the binary fails before it can

ADR-0453 has every `store` call refuse with the reason when the database won't
open. No test drives that path, and in the binary the session log opens the
same database first and ends the run with exit 7, so `quiet::Unopened` is never
reached. `quiet::Ephemeral`'s comment also offers it "for a run with no
workspace database", which the decision rules out.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments; `cargo build -p meow-cli`, `HOME` set to
a scratch directory.

1. Create `.meow/meow.star` with a command whose handler calls
   `put("k", 1)` from `@std//store` and prints `get("k")`.
2. Run `meow trust`.
3. Replace `.meow/.data` with a plain file: `rm -rf .meow/.data` and
   `echo x > .meow/.data`.
4. Run the command. It prints `could not create .../.meow/.data: File exists
   (os error 17)` and exits 7; the handler never runs.
5. Search the tests: `grep -rn Unopened crates/*/tests` finds nothing.

## What the system does

`crates/meow-cli/src/wire.rs:1129-1134` wires `Unopened` when
`Durable::open` fails, but the session store opened earlier fails on the same
database and stops the run. `Unopened` at `crates/meow-star/src/port.rs:433-455`
has no test. The comment on `Ephemeral` at `port.rs:392-396` says it's for
tests "and for a run with no workspace database".

## What it should do, and why

No requirement states the refusal; ADR-0453 decides it in support of REQ-2464
and REQ-2465. The refusal should be proved by a test, or removed if the store
can't fail to open once the session log has opened, and the `Ephemeral` comment
should say it's for tests only.

## Triage

The defect enters in `meow-cli`'s wiring and in `meow-star`'s port comments.
No requirement is violated, so it routes to the design step to decide whether
`Unopened` is needed. It's minor: a missing test and a misleading comment, with
no wrong behaviour a user can reach.

## Closed by

A test in `crates/meow-star/tests/running.rs`, named for example
`a_store_that_will_not_open_refuses_each_call`, that wires `Unopened` and
expects each of `get`, `put`, `delete` and `keys` to fail with its message, or
the removal of `Unopened` with the decision amended.
