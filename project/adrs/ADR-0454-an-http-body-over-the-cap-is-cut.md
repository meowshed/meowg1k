---
id: ADR-0454
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2450]
supersedes: []
---

# 0454. An `http` body over the read cap is cut, not refused

## Decision

A body over the cap is cut rather than refused; `max_bytes` defaults to 8 MiB
and a caller can raise it deliberately (from
https://github.com/retran/meowg1k/pull/145, high).

With this in place, each verb takes `max_bytes`, the default is
`8 * 1024 * 1024`, and the body a handler gets is at most that many bytes,
decoded as UTF-8 with replacement characters (from
crates/meow-star/src/capability_http.rs:43, 173 and 251, high). The cap does
not bound what is read: `response.bytes()` reads the whole body into memory
and the cut happens afterwards (from
crates/meow-star/src/capability_http.rs:245-251, high). The response has no
field saying the body was cut, although the comment above the cut says
`truncated` tells the handler (from
crates/meow-star/src/capability_http.rs:249-266, high).

## Why

A handler that asked for a megabyte of a stream wants the megabyte (from
https://github.com/retran/meowg1k/pull/145, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Refuse a body over the cap | The handler can't mistake a partial body for the whole one (reasoned from crates/meow-star/src/capability_http.rs:249-266, low) | A handler that asked for a megabyte of a stream wants the megabyte (from https://github.com/retran/meowg1k/pull/145, high) |
| Do nothing: read the whole body with no cap | No argument to learn, and a body is never partial (reasoned from https://github.com/retran/meowg1k/pull/145, low) | A server that answers forever hangs a handler that did nothing wrong (from https://github.com/retran/meowg1k/pull/145, high) |

The source names no third option.

## What it costs

A handler can't tell a cut body from a whole one, because the response carries
`status`, `ok`, `body` and `headers` and nothing else (from
crates/meow-star/src/capability_http.rs:260-270, high). A cut can split a
multi-byte character, which then decodes as a replacement character (reasoned
from crates/meow-star/src/capability_http.rs:251, low).

## What would reverse it

This is reversed if handlers are found treating cut bodies as whole ones, for
example a JSON parse failing on a body that ended at `max_bytes`, and the
module switches to refusing (reasoned from
crates/meow-star/src/capability_http.rs:251, low).

## Consequences

- `get(url, max_bytes = 4).body` on a ten-byte body returns the first four
  bytes and no error (from crates/meow-star/tests/running.rs:2351-2375, high).
- The read is still bounded in time by `timeout_secs`, 30 by default, which is
  what stops a server that sends forever (from
  crates/meow-star/src/capability_http.rs:36 and 236-246, high).

## How I will know it was realised

`max_bytes_cuts_a_body_that_is_too_long` in
`crates/meow-star/tests/running.rs` passes (from
crates/meow-star/tests/running.rs:2351, high).

## What this does not settle

- Whether the cap should bound the read itself. [R-STAR-022] says a call MUST
  cap how much of a response it will read, and the code reads everything before
  cutting, so a large body is held in memory in full (from
  docs/spec/starlark.md [R-STAR-022] and
  crates/meow-star/src/capability_http.rs:245-251, high).
- Whether the response should carry a `truncated` flag, which the code comment
  promises and the code does not return (from
  crates/meow-star/src/capability_http.rs:249-266, high).
- Binary bodies, which come back with replacement characters because the
  dialect has no bytes type (from https://github.com/retran/meowg1k/pull/145,
  high).
