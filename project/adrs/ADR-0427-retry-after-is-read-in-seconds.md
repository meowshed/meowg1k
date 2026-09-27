---
id: ADR-0427
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1630]
supersedes: []
---

# 0427. `Retry-After` is read only in its seconds form

## Decision

`Retry-After` reads only the seconds form (from
https://github.com/meowshed/meowg1k/pull/125, high).

Once this is accepted, a header holding a whole number of seconds sets the
wait before the next attempt, and any other value, an HTTP date included, reads
as no header, so the retry policy falls back to its own jittered backoff (from
crates/meow-llm/src/http.rs:78-86 and crates/meow-llm/src/retry.rs:74-77,
high).

## Why

No vendor here sends the HTTP-date form, and guessing wrong about a date is
worse than falling back to the retry policy's own backoff, which already handles
a missing header (from https://github.com/meowshed/meowg1k/pull/125, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Also parse the HTTP-date form | The HTTP-date form is legal (from https://github.com/meowshed/meowg1k/pull/125, high) | No vendor here sends it, and guessing wrong about a date is worse than the backoff fallback (from https://github.com/meowshed/meowg1k/pull/125, high) |
| Do nothing: ignore `Retry-After` and always use the backoff | One wait rule for every failure, with no server input to trust (reasoned from crates/meow-llm/src/retry.rs:34-47, low) | REQ-1630 requires honouring the header, and a server that says how long to wait knows better than an exponent (from crates/meow-llm/src/retry.rs:50-57, high) |

No third option appears in the code, the history or the design documents.

## What it costs

A server that sends a date gets retried on the backoff schedule, up to the
30-second cap, which may be sooner than it asked (from
crates/meow-llm/src/retry.rs:24-31 and crates/meow-llm/src/retry.rs:74-77,
medium). For Gemini the cost is larger: a `RESOURCE_EXHAUSTED` response with
no readable `Retry-After` counts as a spent quota and isn't retried, so a date
there would turn a rate limit into a quota failure (from
crates/meow-llm/src/gemini.rs:169-172, medium).

## What would reverse it

- A provider this tree talks to sending `Retry-After` as an HTTP date, seen in
  a recording or a reported failure (from
  https://github.com/meowshed/meowg1k/pull/125, high).

## Consequences

- Every provider reads the value through the one transport, so Anthropic,
  OpenAI, Gemini and Voyage all get the same rule (from
  crates/meow-llm/src/anthropic.rs:179, crates/meow-llm/src/openai.rs:195,
  crates/meow-llm/src/gemini.rs:168 and crates/meow-llm/src/voyage.rs:141,
  high).
- A fractional value such as `1.5` doesn't parse as a whole number and falls
  back to the backoff as well (from crates/meow-llm/src/http.rs:85, medium).

## How I will know it was realised

1. `a_status_comes_back_rather_than_failing` in crates/meow-llm/tests/http.rs
   reads `Retry-After: 7` from a real socket as seven seconds (from
   crates/meow-llm/tests/http.rs:55-77, high).
2. `retry_after_is_honoured` in crates/meow-llm/tests/spec.rs waits exactly the
   server's twelve seconds (from crates/meow-llm/tests/spec.rs:434-466, high).
3. No test sends an HTTP date and checks the backoff takes over; one would
   close that (from a search of crates/meow-llm/tests for `Retry-After`, high).

## What this does not settle

- An upper bound on the server's number: the wait comes from the header
  without the policy's 30-second cap, so a large value is waited out in full
  unless the run is cancelled (from crates/meow-llm/src/retry.rs:74-80,
  high).
- Whether REQ-1630's "when the response carries one" covers the date form;
  this decision reads it as covering the seconds form only (from
  docs/spec/llm.md:103 and crates/meow-llm/src/http.rs:78-86, medium).
