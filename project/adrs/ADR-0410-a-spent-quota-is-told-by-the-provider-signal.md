---
id: ADR-0410
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1634, REQ-1635]
supersedes: []
---

# 0410. A spent quota is told from a rate limit by the provider's own signal

## Decision

A 429 is a rate limit unless the provider's own documented signal says the quota
is spent, and that signal is a field the provider fills from its response (from
https://github.com/retran/meowg1k/pull/117, high). `LlmError::Http` carries a
`quota_exhausted` flag, and `LlmError::class` returns `QuotaExhausted` when it
is set and `Transient` for a 429 when it isn't (from
crates/meow-llm/src/error.rs:114-137, high). Each provider reads its own signal:

- Anthropic: `error.type` of `billing_error` or `credit_balance_too_low` (from
  crates/meow-llm/src/anthropic.rs:180, high).
- OpenAI and its compatibles: `error.code` of `insufficient_quota` or
  `billing_hard_limit_reached` (from crates/meow-llm/src/openai.rs:196, high).
- Gemini: `RESOURCE_EXHAUSTED` with no `Retry-After` (from
  crates/meow-llm/src/gemini.rs:170-172, high).
- Voyage: a 402 status (from crates/meow-llm/src/voyage.rs:142-145, high).

Once this is accepted, a spent quota surfaces at once with no retry, and a rate
limit is retried (from crates/meow-llm/src/retry.rs:52-53, high). What still
doesn't work: a failed status on the streaming path of the real `Http`
transport is returned with `quota_exhausted` false and never reaches the
provider's reading, so a spent quota on a streamed request would be retried
(from crates/meow-llm/src/http.rs:128-141 and
crates/meow-llm/src/anthropic.rs:341, high). The engine calls `generate`, not
`stream`, so no run takes that path today (from
crates/meow-agent/src/engine.rs:203 and crates/meow-agent/src/engine.rs:242,
high).

## Why

The ambiguous case takes the cheaper mistake: retrying a spent quota costs a
delay, and refusing a rate limit costs the run (from
https://github.com/retran/meowg1k/pull/117, high). Matching text out of a
message broke whenever a provider reworded an error (from
https://github.com/retran/meowg1k/pull/117, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: match the quota signal out of the error message's text, as v0.2.x did | One function covered every provider, with no per-vendor code (from v0.2.1:internal/adapters/gateway/retry.go:30-46, high) | It broke whenever a provider reworded an error (from https://github.com/retran/meowg1k/pull/117, high) |
| Treat an ambiguous 429 as a spent quota | A spent quota fails at once, with no backoff spent on it (reasoned from crates/meow-llm/src/retry.rs:52-53, low) | Refusing a rate limit costs the run, where retrying a spent quota costs only a delay (from https://github.com/retran/meowg1k/pull/117, high) |
| The provider's documented signal, with an ambiguous 429 read as transient (chosen) | A reworded message doesn't change the classification (from crates/meow-llm/src/anthropic.rs:156-160, high) | It won |

## What it costs

Gemini uses one status for a rate limit and an exhausted quota, so
`RESOURCE_EXHAUSTED` reads as a spent quota only when there is no `Retry-After`,
and a provider that sends neither is retried when it shouldn't be (from
https://github.com/retran/meowg1k/pull/133, high). That retry is bounded: four
attempts, each backoff capped at 30 seconds (from
crates/meow-llm/src/retry.rs:27-29, high). Each provider also carries its own
list of quota codes, which has to follow the vendor's documentation (from
crates/meow-llm/src/openai.rs:174-196, high).

## What would reverse it

- A provider's documented quota code changes and the test for it still
  passes, because the recording holds the old code; the per-provider reading
  would then need a live check (reasoned from
  crates/meow-llm/tests/providers.rs:201-237, low).
- The retry schedule grows long enough that retrying a spent quota costs more
  than refusing a rate limit, which inverts the cost argument (reasoned from
  crates/meow-llm/src/retry.rs:27-29 and
  https://github.com/retran/meowg1k/pull/117, low).

## Consequences

- `LlmError::Http` has a `quota_exhausted` field every provider fills (from
  crates/meow-llm/src/error.rs:37-38, high).
- A new provider has to name its quota signal, or its spent quota reads as a
  rate limit and is retried (reasoned from crates/meow-llm/src/error.rs:127-135,
  low).
- An invalid API key no longer costs the full backoff schedule, because a 401
  is `Fatal` (from crates/meow-llm/src/error.rs:8-11 and
  crates/meow-llm/src/error.rs:132-134, high).

## How I will know it was realised

1. `a_spent_quota_is_told_apart_from_a_rate_limit` in
   crates/meow-llm/tests/spec.rs passes: a 429 with `rate_limit_error` is
   `Transient` and a 429 with `credit_balance_too_low` is `QuotaExhausted`
   (from crates/meow-llm/tests/spec.rs:467-500, high).
2. `a_spent_quota_comes_from_the_api_rather_than_its_wording` in
   crates/meow-llm/tests/providers.rs passes for OpenAI's
   `insufficient_quota` and `rate_limit_exceeded` (from
   crates/meow-llm/tests/providers.rs:201-237, high).

## What this does not settle

- The streaming path: the real `Http` transport's `post_lines` builds the
  error itself with `quota_exhausted` false, so the provider's signal is never
  read there (from crates/meow-llm/src/http.rs:128-141, high).
- How long a rate limit waits; ADR-0427 covers reading `Retry-After`.
