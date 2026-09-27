---
id: BUG-0042
artifact: bug
status: approved
severity: minor
violates: REQ-1634
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A streamed request's HTTP error loses the provider's quota signal and `Retry-After`

When a streaming request fails with an HTTP status, the transport builds the
error itself with `quota_exhausted: false` and `retry_after: None`. The
provider's own `error_from`, which reads the vendor's quota signal and the
header, is never consulted, so a spent quota on a streamed call classifies as
`Transient`.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Serve Anthropic's streaming endpoint from a local server that answers 429 with
   body `{"error": {"type": "billing_error", "message": "..."}}` and header
   `Retry-After: 30`.
2. Call `generate_stream` on an `Anthropic` provider pointed at it and call
   `.class()` and `.retry_after()` on the error.

The error is `Transient` with no wait. The same body through `generate` gives
`QuotaExhausted` and 30 seconds. This was traced through the code and not run.

## What the system does

`Http::post_lines` at `crates/meow-llm/src/http.rs:128-141` returns
`LlmError::Http { quota_exhausted: false, retry_after: None, .. }` for any
non-success status, and the providers pass it on with `?`
(`crates/meow-llm/src/anthropic.rs:341`, `crates/meow-llm/src/gemini.rs:348`).
The non-streamed `post` returns the status, body and header to the provider,
which classifies them (`crates/meow-llm/src/anthropic.rs:161-180`).

## What it should do, and why

REQ-1634: "A provider MUST distinguish `QuotaExhausted` from a rate limit using
its own documented signal, not by matching text in a message." REQ-1630 asks
retry to honour `Retry-After`, and REQ-1632 asks a spent quota to surface at
once with the provider named. ADR-0410 and ADR-0427 record the decisions.

## Triage

The defect enters in `meow-llm`'s `Http::post_lines`, which classifies an error
it can't interpret. The engine streams whenever the renderer wants deltas, which
the terminal renderer does, so this is the common path. It is minor today only
because no call is retried (BUG-0005), so both classes fail the run; once retry
is wired, a spent quota would be retried for nothing.

## Closed by

A test in `crates/meow-llm/tests/spec.rs`, named for example
`a_streamed_quota_error_is_quota_exhausted`, that records the response above for
a streamed call and expects `QuotaExhausted` and a 30-second `retry_after`.
