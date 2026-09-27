---
id: BUG-0132
artifact: bug
status: approved
severity: minor
violates: REQ-2451
found: 2026-09-27
revised: 2026-09-27
issue: 255
---

# No test calls `@std//http` with a cancelled run

The four tests that name REQ-2451 check responses, bodies, the byte cap and
schemes, and none cancels the run, so the check ADR-0466 added before the
request and the `select` around it are unguarded.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. `grep -rn 2451 crates/*/tests`. The hits are
   `crates/meow-star/tests/running.rs:2205`, `:2304`, `:2349` and `:2381`.
2. Read each. None cancels the token or counts requests at the server after a
   cancel.

## What the system does

`@std//http` checks `cancel.is_cancelled()` before building the client,
`crates/meow-star/src/capability_http.rs:178-184`, and races the request
against the token. The package fetch, the device flow and the credential
exchange have cancellation tests, in `crates/meow-cli/tests/fetch.rs`,
`crates/meow-cli/tests/device.rs` and `crates/meow-llm/tests/bearer.rs`.

## What it should do, and why

REQ-2451: "Every `http` call MUST NOT leave a request in flight when the run is
cancelled." A test should prove that a cancelled run makes no request and that
a cancel during a slow request ends it.

## Triage

The defect enters in `meow-star`'s tests. It's minor: the code does what the
requirement says, and only the guard is missing.

## Closed by

A test in `crates/meow-star/tests/running.rs`, named for example
`a_cancelled_run_makes_no_http_request`, that cancels before the call and
expects zero requests at a local server and an error saying the run was
cancelled.
