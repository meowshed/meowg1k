---
id: ADR-0462
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1606, REQ-1220]
supersedes: []
---

# 0462. A refused credential renewal is fatal and isn't retried

## Decision

A refused renewal has its own `LlmError::Auth` variant, classed `Fatal`, and its
message names the provider and the command (from
https://github.com/meowshed/meowg1k/pull/156, high).

Once this is accepted, `Exchanged::token` turns any response to the exchange
that isn't a success into `LlmError::Auth` with the message
``the credential would not renew (401 Unauthorized); run `meow auth login
copilot` `` for a 401 from Copilot, and `LlmError::class` puts every
`Auth` in `Class::Fatal`, which the retry loop never retries (from
crates/meow-llm/src/bearer.rs:161, crates/meow-llm/src/error.rs:140 and
crates/meow-llm/src/retry.rs:51, high). A network failure while renewing stays a
`Transport` error and is retried (from crates/meow-llm/src/bearer.rs:152 and
crates/meow-llm/src/error.rs:139, high). Any status counts as a refusal,
including a 503 from the exchange, so a brief outage there is fatal and tells
the person to log in again (from crates/meow-llm/src/bearer.rs:161, high).

## Why

A grant that won't renew doesn't renew on the second attempt, and retrying
spends the whole backoff to arrive at the same sentence a person has to read
anyway (from https://github.com/meowshed/meowg1k/pull/156, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Retry a refused renewal like a transient error | A renewal refused by a passing fault at the exchange, such as a 503, would succeed on a later attempt (reasoned from crates/meow-llm/src/error.rs:134, low) | It spends the whole backoff to reach the same failure (from https://github.com/meowshed/meowg1k/pull/156, high) |
| Do nothing: renew inside the retried request, as v0.2.x's `refreshTokenIfNeeded` did, with a generic "token exchange returned status" error | No error variant of its own, and every failure takes one path (from `git show v0.2.1:internal/adapters/gateway/copilot.go`, medium) | The message named neither the provider nor the command that fixes it, which REQ-1606 requires (from docs/spec/llm.md [R-LLM-004], high) |
| Report the refusal as an `Http` error with its status | Status-based classing would keep a 5xx transient and a 401 fatal (reasoned from crates/meow-llm/src/error.rs:127, low) | `Auth` is its own variant because it is the one failure a person can fix and the message says how (from crates/meow-llm/src/error.rs:50, high) |

## What it costs

A transient fault at the exchange endpoint ends the run at once with advice to
log in again, which is wrong for that case (reasoned from
crates/meow-llm/src/bearer.rs:161, medium). No test talks to Copilot, so a
change in its exchange's wire shape leaves these tests green and the provider
broken (from https://github.com/meowshed/meowg1k/pull/156, high).

## What would reverse it

An exchange endpoint observed to refuse with 429 or 5xx often enough that runs
fail while the grant is still good (reasoned from
crates/meow-llm/src/bearer.rs:161, low).

## Consequences

- The request fails at once with the provider's name and
  `meow auth login <provider>` in the message, with no backoff spent (from
  crates/meow-llm/src/bearer.rs:164 and crates/meow-llm/src/retry.rs:51, high).
- The exchange has no timeout of its own and relies on `reqwest`'s defaults and
  the cancellation token (from https://github.com/meowshed/meowg1k/pull/156,
  high).

## How I will know it was realised

`a_refused_renewal_says_what_to_run` and
`a_refused_renewal_is_not_worth_retrying` in `crates/meow-llm/tests/bearer.rs`
pass: a 401 gives `LlmError::Auth` naming `copilot` and `meow auth login
copilot`, classed `Fatal` (from crates/meow-llm/tests/bearer.rs:163, high).
`a_fatal_error_surfaces_at_once_and_is_not_retried` in
`crates/meow-llm/tests/spec.rs` covers the retry loop's side (from
crates/meow-llm/tests/spec.rs:379, high).

## What this does not settle

- Whether a 429 or 5xx from the exchange should be `Transient` rather than
  `Auth` (from crates/meow-llm/src/bearer.rs:161, high).
- A deadline for the exchange request itself (from
  https://github.com/meowshed/meowg1k/pull/156, high).
