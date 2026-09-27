---
id: BUG-0033
artifact: bug
status: approved
severity: major
violates: REQ-2450
found: 2026-09-27
revised: 2026-09-27
issue: 206
---

# `http` reads the whole response body before cutting it to `max_bytes`

The `@std//http` module waits for the complete body and then keeps the first
`max_bytes`. A large or endless response is read into memory in full, so the cap
limits what the handler sees and not what the process reads.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Serve a 2 GiB response from a local HTTP server.
2. Run a command whose handler calls `http.get(url, max_bytes = 1024)`.

The process reads the whole body, until the timeout or memory runs out, before
returning 1024 bytes. This was traced through the code and not run.

## What the system does

`crates/meow-star/src/capability_http.rs:236-251` awaits `response.bytes()`,
which buffers the whole body, and then slices `bytes[..cap]`. The comment at
line 250 says a `truncated` field tells the handler what happened, and the
returned struct at `crates/meow-star/src/capability_http.rs:261-271` has
`status`, `ok`, `body` and `headers` only. A cut in the middle of a UTF-8
sequence is replaced by `from_utf8_lossy`.

## What it should do, and why

REQ-2450: "Every `http` call MUST cap how much of a response it will read." The
requirement caps what is read, not what is kept.

## Triage

The defect enters in `meow-star`'s `http` capability. It is major because the
requirement's main behaviour, bounding the read, doesn't happen, and a handler
fetching a URL a model chose can be made to exhaust memory. The fix is to read
the body as a stream and stop at the cap, and either add `truncated` or remove
the comment that promises it.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`http_stops_reading_at_max_bytes`, against a local server that streams without
end, expecting the call to return promptly with `max_bytes` of body.
