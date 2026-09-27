---
id: ADR-2408
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2452, REQ-2453, REQ-2454, REQ-2455]
supersedes: []
---

# 2408. An HTTP status is an answer and not a failure

## Decision

An `http` call returns any response the server sent, with its status, headers
and body, and fails only when it never reached a response (from
docs/spec/starlark.md [R-STAR-023], high).

## Why

A handler that polls until something returns 200, or treats 404 as "not yet", is
the normal case, and a module that raised on 404 would make both write the error
handling twice. What failed is reaching the server at all, and that is what
fails (from docs/spec/starlark.md, Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Raise on a 404 or another error status | A handler that never looks at the status can't read an error page as the answer (reasoned from crates/meow-star/src/capability_http.rs:262-265, where `ok` exists so that check is one field, low). | A handler that polls for 200 or treats 404 as "not yet" would write the error handling twice (from docs/spec/starlark.md, Decisions, high). |
| Do nothing: keep the v0.2.x `http` module as it was | It already returned every status with `status_code` and `ok`, and also decoded a JSON body into a `json` field (from v0.2.1:internal/core/starlark/module_http.go:266-272, high). | It agreed with this decision on status, but read the whole body with `io.ReadAll` under `context.Background()`, so it had no read cap and left a request in flight on cancellation, which REQ-2450 and REQ-2451 forbid (from v0.2.1:internal/core/starlark/module_http.go:69,116,248, medium). |

Neither the code, the history nor the design documents name a third option,
such as raising by default with a per-call opt-out, so the table has none.

## What it costs

Every handler that cares about success checks `status` or `ok` itself, and one
that forgets treats an error page as data (reasoned from
crates/meow-star/src/capability_http.rs:260-270, low). The response carries
`ok` for 2xx as "the one judgement worth making here" to keep that check short
(from https://github.com/retran/meowg1k/pull/145, high).

## What would reverse it

If most `http` calls in the workspaces the binary serves are followed by a
check that fails on any status outside 2xx, raising by default would remove
more handler code than it adds, and the reason in Why no longer holds
(reasoned from docs/spec/starlark.md, Decisions, low).

## Consequences

- A response is a struct of `status`, `ok`, `body` and `headers`, whatever the
  status (from crates/meow-star/src/capability_http.rs:260-270, high).
- The failures are the ones that never produced a response, each naming the
  URL: "could not reach", "did not answer in N seconds", "did not finish
  sending" and "the run was cancelled" (from
  crates/meow-star/src/capability_http.rs:211-247, high).
- A redirect is followed up to ten times, so a handler sees a 3xx status only
  after that limit; the limit is `reqwest`'s policy rather than a requirement
  (from https://github.com/retran/meowg1k/pull/145, high).

## How I will know it was realised

The tests `a_404_is_a_response_rather_than_an_error`,
`a_connection_that_is_refused_fails` and
`a_response_carries_what_the_server_sent` in
crates/meow-star/tests/running.rs pass against a real server on a loopback
port, not a mocked client (from https://github.com/retran/meowg1k/pull/145,
high).

## What this does not settle

- How a handler tells a body `max_bytes` cut from a whole one. The comment at
  crates/meow-star/src/capability_http.rs:250 says `truncated` reports it, but
  the response struct has no such field (from
  crates/meow-star/src/capability_http.rs:250-270, high).
- A binary body: the body is decoded as UTF-8 with `from_utf8_lossy`, because
  the Starlark dialect has no bytes type (from
  https://github.com/retran/meowg1k/pull/145, high).
- The deadline, the read cap and cancellation, which REQ-2448 to REQ-2451 cover
  (from docs/spec/starlark.md [R-STAR-022], high).
