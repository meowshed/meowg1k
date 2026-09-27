---
id: BUG-0044
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A server's `Retry-After` is used without a ceiling

`with_retry` sleeps for whatever `Retry-After` says, with no upper bound. A
server that answers 503 with `Retry-After: 86400` makes the run wait a day
before the next attempt.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Call `with_retry` with the default `Retry` and an operation whose first
   result is `LlmError::Http { status: 503, retry_after: Some(86400 s), .. }`.

It sleeps 86400 seconds before the second attempt, unless the run is
cancelled. This was traced through the code and not run.

## What the system does

`crates/meow-llm/src/retry.rs:73-77` takes `last.retry_after()` when present
and falls back to the backoff only when it is absent. Nothing compares the
server's value with the policy's maximum delay.

## What it should do, and why

No requirement sets a ceiling. REQ-1630 asks retry to honour `Retry-After`, and
REQ-1631 stops retry after a configured number of attempts, which doesn't bound
the time between them. A ceiling, after which the error surfaces with the wait
the server asked for, is a gap in the requirements and routes to the
requirements step. ADR-0410 records the retry decision.

## Triage

The defect enters in `meow-llm`'s `with_retry`. It is minor and dormant, since
nothing calls `with_retry` today (BUG-0005). Once it is wired, a run could
hang with no output for as long as a server says.

## Closed by

A test in `crates/meow-llm/tests/spec.rs`, named for example
`a_long_retry_after_surfaces_the_error`, that expects a `Retry-After` above the
ceiling to fail at once with the wait named.
