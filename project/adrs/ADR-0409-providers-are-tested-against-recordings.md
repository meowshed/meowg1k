---
id: ADR-0409
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1618]
supersedes: []
---

# 0409. Providers are tested against recorded exchanges behind a `Transport` trait

## Decision

HTTP sits behind a `Transport` trait, so a provider is tested against a recorded
exchange instead of a live account, and no test reaches the network (from
https://github.com/meowshed/meowg1k/pull/117, high). The trait has two methods,
`post` and `post_lines`, and `Recorded` replays responses and streams in order
and keeps every body sent so a test can check what was asked (from
crates/meow-llm/src/transport.rs:26-90, high). The real transport, `Http`, is
tested against a server on localhost that answers one request (from
crates/meow-llm/tests/http.rs:1-20, high).

Once this is accepted, every provider, Anthropic, OpenAI and its compatibles,
Gemini and Voyage, is tested in both directions against the same recorded
exchanges (from https://github.com/meowshed/meowg1k/pull/133, high). What still
doesn't work is any check that a vendor's real endpoint accepts the bodies built
here (from https://github.com/meowshed/meowg1k/pull/133, high).

## Why

Two live calls to a model can't be compared, because the model is free to answer
differently, which is why `R-LLM-021` is stated over a recording (from
https://github.com/meowshed/meowg1k/pull/117, high). The retry tests also run on a
paused clock, so a `retry_after` of twelve seconds is checked as exactly twelve
seconds (from https://github.com/meowshed/meowg1k/pull/117, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: v0.2.x's per-provider `httptest` mock server, simulating each vendor's API | The provider's real HTTP client runs in every test (from v0.2.1:internal/adapters/gateway/anthropic_test.go:48-53, high) | Each provider test would pay for a socket and a server to check a translation the trait isolates; the one test of real HTTP now lives in crates/meow-llm/tests/http.rs, because a test that stubs out HTTP tests the stub (reasoned from crates/meow-llm/tests/http.rs:1-9, low) |
| Test providers against a live account | It proves the vendor accepts the body, which no recording can (from https://github.com/meowshed/meowg1k/pull/133, high) | Two live calls to a model can't be compared (from https://github.com/meowshed/meowg1k/pull/117, high) |
| A `Transport` trait with a `Recorded` replay (chosen) | No network, no account and an answer that doesn't change between runs (from crates/meow-llm/src/transport.rs:57-59, high) | It won |

## What it costs

No live call is tested, so nothing checks that each vendor's real endpoint
accepts the bodies built here (from https://github.com/meowshed/meowg1k/pull/133,
high). A recording is written by hand in the test, so it is only as current as
the vendor documentation its author read (reasoned from
crates/meow-llm/tests/providers.rs:20-35, low).

## What would reverse it

- A vendor changes its request or response shape and a user reports a failure
  that every recorded test passes through, which shows the recordings have
  drifted from the vendor (reasoned from
  https://github.com/meowshed/meowg1k/pull/133, low).

## Consequences

- Every provider is generic over its transport, and a test reads what was sent
  through `transport().bodies()` (from crates/meow-llm/tests/spec.rs:205-207,
  high).
- A status is not an error at the `post` boundary: the transport returns it and
  the provider classifies it, because only the provider knows which of its
  codes are transient (from crates/meow-llm/src/http.rs:103-106, high).
- The test suite needs no API key, so CI runs it without secrets (reasoned from
  crates/meow-llm/src/transport.rs:57-59, low).

## How I will know it was realised

1. `aggregating_a_recorded_stream_matches_the_whole_answer` in
   crates/meow-llm/tests/spec.rs passes: a recorded stream aggregates to the
   same response as the non-streaming body (from
   crates/meow-llm/tests/spec.rs:209-212, high).
2. No test under crates/meow-llm/tests opens a connection outside localhost;
   the only socket is the one crates/meow-llm/tests/http.rs binds on
   `127.0.0.1` (from crates/meow-llm/tests/http.rs:19-21, high).

## What this does not settle

- Whether a vendor's live endpoint accepts what is sent: no contract test
  against a real account exists (from
  https://github.com/meowshed/meowg1k/pull/133, high).
- The streaming path of `Http` returns a failed status as an error itself, with
  no quota signal and no `Retry-After`, where `post` leaves the status to the
  provider (from crates/meow-llm/src/http.rs:128-141, high).
