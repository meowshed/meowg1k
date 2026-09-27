---
id: BUG-0005
artifact: bug
status: approved
severity: major
violates: REQ-1629
found: 2026-09-27
revised: 2026-09-27
issue:
---

# No provider call is retried, so a 429 or 503 fails the run at once

`meow_llm::with_retry` implements backoff, jitter, `Retry-After` and the attempt
cap, and nothing in production calls it. The engine calls `generate` or
`generate_stream` once, and any error, transient or not, ends the step with stop
reason `failed`.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `grep -rn with_retry crates`. The only callers are in
   `crates/meow-llm/tests/spec.rs`, and the only other hit is the re-export in
   `crates/meow-llm/src/lib.rs:38`.
2. Drive `meow_agent::Engine` with a recorded provider whose first answer is an
   HTTP 503 and whose second is a text reply. The run ends `failed` after one
   request.

## What the system does

`crates/meow-agent/src/engine.rs:235-249` makes one call and maps every error
other than `Cancelled` to `Turn::Stop(StopReason::Failed, ...)`. No provider in
`crates/meow-llm/src/` wraps its own requests in `with_retry` either.

## What it should do, and why

REQ-1629: "Retry MUST use exponential backoff with random jitter." REQ-1630 and
REQ-1631 add `Retry-After` and the attempt cap, and REQ-1627 limits retry to
`Transient` errors. All four assume retry happens. ADR-1603 records the
decision to classify before retrying.

## Triage

The defect enters in the wiring between `meow-agent` and `meow-llm`: the policy
exists and nothing applies it. It is major because the requirement's main
behaviour doesn't happen, and a rate limit that a retry would absorb costs the
user the whole run.

## Closed by

A test in `crates/meow-agent/tests/`, named for example
`a_transient_provider_error_is_retried_before_the_run_fails`, that records a
503 then a reply and expects the run to finish with two requests sent.
