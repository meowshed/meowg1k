---
id: BUG-0043
artifact: bug
status: approved
severity: minor
violates: REQ-1630
found: 2026-09-27
revised: 2026-09-27
issue: 216
---

# A `Retry-After` in HTTP-date form is ignored, and Gemini then reads a rate limit as a spent quota

The transport reads `Retry-After` only as a number of seconds. A date, which
HTTP also allows, becomes `None`. Gemini's provider uses the header's absence
as its quota signal, so a `RESOURCE_EXHAUSTED` rate limit carrying a date
classifies as `QuotaExhausted`.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Give the Gemini provider a recorded 429 whose body has `"status":
   "RESOURCE_EXHAUSTED"` and whose headers carry `Retry-After: Wed, 21 Oct 2026
   07:28:00 GMT`.
2. Call `generate` and call `.class()` on the error.

It returns `QuotaExhausted` with no `retry_after`. With `Retry-After: 30` it
returns `Transient`. This was traced through the code and not run.

## What the system does

`retry_after` in `crates/meow-llm/src/http.rs:78-86` parses the header as
`u64` and returns `None` otherwise; its comment says no vendor here sends a
date. `crates/meow-llm/src/gemini.rs:172` sets `quota_exhausted` to
`status == "RESOURCE_EXHAUSTED" && response.retry_after.is_none()`.

## What it should do, and why

REQ-1630: "Retry MUST honour a `Retry-After` header when the response carries
one." A date is a `Retry-After` header. REQ-1635 makes an unsignalled 429
`Transient` "because retrying a spent quota costs a delay while refusing a rate
limit costs the run", and the Gemini path makes the costlier mistake. ADR-0410
and ADR-0427 record the decisions.

## Triage

The defect enters in `meow-llm`'s header parser and in Gemini's use of it as a
signal. It is minor: no vendor the code serves is known to send a date, and no
call is retried yet (BUG-0005). The fix is to parse the date form, and for
Gemini to read the quota from the body's `details` where Google documents it.

## Closed by

A test in `crates/meow-llm/tests/spec.rs`, named for example
`retry_after_as_a_date_is_honoured`, and one named
`gemini_rate_limit_with_a_dated_retry_after_is_transient`.
