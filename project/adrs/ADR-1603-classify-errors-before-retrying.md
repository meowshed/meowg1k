---
id: ADR-1603
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1624, REQ-1625, REQ-1626, REQ-1627, REQ-1628]
supersedes: []
---

# 1603. Provider errors are classified, and only transient ones are retried

## Decision

Every provider error is classified as `Transient`, `Fatal` or `QuotaExhausted`,
only `Transient` errors are retried, and a `Fatal` error surfaces on its first
occurrence [REQ-1624] [REQ-1627] [REQ-1628] (from docs/spec/llm.md, high).

`LlmError::class` does the classifying: 408, 429, 500, 502, 503 and 504 and
every transport failure are `Transient` [REQ-1625], a status the provider marks
as a spent quota is `QuotaExhausted`, and everything else is `Fatal` [REQ-1626]
(from crates/meow-llm/src/error.rs:123-150, high). `with_retry` retries a
`Transient` error and returns any other at once (from
crates/meow-llm/src/retry.rs:84-88, high).

## Why

Retrying every error that isn't a hard quota error makes an invalid API key cost
the full backoff schedule before it surfaces (from docs/spec/llm.md, high). The
classification came before the happy path, because retry behaviour is what the
v0.2.x gateway got wrong and it is hard to retrofit (from
https://github.com/meowshed/meowg1k/pull/117, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: retry every error that isn't a hard quota error, as `RetryWithBackoff` in v0.2.x's `internal/adapters/gateway/retry.go` does | One rule and no per-provider mapping; a failure that looks fatal still gets another try (reasoned from `git show v0.2.1:internal/adapters/gateway/retry.go`, low) | An invalid API key costs the full backoff schedule before it surfaces (from docs/spec/llm.md, high); with v0.2.x's five tries from 2 s that is 30 s of waiting (reasoned from the same file, `DefaultRetryConfig`, medium) |
| Tell a spent quota apart by matching the error text, as v0.2.x's `isHardQuotaError` does | Needs nothing from the provider's response but its message (from `git show v0.2.1:internal/adapters/gateway/retry.go`, high) | It broke whenever a provider reworded an error (from https://github.com/meowshed/meowg1k/pull/117, high) |
| Retry nothing, and surface every error | No delay on any failure (reasoned from crates/meow-llm/src/retry.rs, low) | Refusing a rate limit costs the run (from docs/spec/llm.md [R-LLM-037], high) |

## What it costs

Every provider maps its response to a status and a `quota_exhausted` flag from
its own documented signal (from crates/meow-llm/src/error.rs:28-39, high).
Where a vendor has no clean signal the mapping guesses: Gemini's
`RESOURCE_EXHAUSTED` counts as a spent quota only when no `Retry-After` came
with it, so a provider that sends neither is retried when it shouldn't be (from
crates/meow-llm/src/gemini.rs:168-172, high;
https://github.com/meowshed/meowg1k/pull/133, high).

A status outside the two lists is `Fatal`, so a retryable status the lists
don't name, such as Anthropic's 529 "overloaded", surfaces at once (from
crates/meow-llm/src/error.rs:133-136, high; Anthropic's status code, medium).

## What would reverse it

- A status classed `Fatal` is seen to succeed when repeated, often enough that
  surfacing it at once costs more runs than the backoff costs time (reasoned
  from crates/meow-llm/src/error.rs:133-136, low).

## Consequences

- A `Fatal` error reaches the caller with zero delay (from
  https://github.com/meowshed/meowg1k/pull/117, high).
- A credential that won't renew is `Fatal`, which ADR-0462 records (from
  crates/meow-llm/src/error.rs:140-148, high).
- Nothing outside the tests calls `with_retry`: no provider, the engine or the
  index wraps a request in it, so a `Transient` error fails the run on its first
  occurrence today (from `grep -rn with_retry crates`, high).

## How I will know it was realised

1. In `crates/meow-llm/tests/spec.rs`,
   `every_status_lands_in_exactly_one_class`,
   `a_fatal_error_surfaces_at_once_and_is_not_retried` (one try, zero elapsed
   time), `a_transient_error_is_retried` and
   `a_spent_quota_is_told_apart_from_a_rate_limit` pass (from
   crates/meow-llm/tests/spec.rs:347-503, high).
2. A request the engine sends goes through `with_retry`, so a recorded 503
   followed by a 200 finishes the run; no test checks this, and today it fails
   (from `grep -rn with_retry crates`, high).

## What this does not settle

- The attempt count and the backoff: `Retry::default` is four tries from
  500 ms, capped at 30 s (from crates/meow-llm/src/retry.rs:24-32, high).
- Each provider's quota signal, which ADR-0410 decides (from
  crates/meow-llm/src/error.rs:117-122, high).
- How `Retry-After` is read, which ADR-0427 decides (from
  crates/meow-llm/src/http.rs:80-84, high).
- Where the retry wraps a request, in the provider or in the engine (from
  `grep -rn with_retry crates`, high).
