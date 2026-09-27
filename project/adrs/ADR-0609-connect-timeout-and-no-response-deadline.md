---
id: ADR-0609
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1652, REQ-1653]
supersedes: []
---

# 0609. A provider call times out only while it connects

## Decision

The provider transport gives up connecting after 30 seconds and puts no
deadline on the response (from https://github.com/meowshed/meowg1k/pull/125,
high). The code sets `reqwest`'s `connect_timeout` and no `timeout` (from
crates/meow-llm/src/http.rs:24 and :46-49, high). The comment on
`CONNECT_TIMEOUT` calls it "how long to wait for the first byte of a response",
which isn't what `connect_timeout` bounds; REQ-1652 follows the code and the
pull request (reasoned from crates/meow-llm/src/http.rs:19, low).

## Why

A model thinking for two minutes is normal, and a whole-response timeout that
fired on one would make the retry logic hammer a provider that is working (from
https://github.com/meowshed/meowg1k/pull/125, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no timeout at all | Nothing to tune (reasoned from crates/meow-llm/src/http.rs:44-53, low) | A connection that goes nowhere holds the run until the operating system gives up (reasoned from crates/meow-llm/src/http.rs:24, low) |
| A deadline on the whole response | A stalled response ends on its own (reasoned from https://github.com/meowshed/meowg1k/pull/125, low) | It fires on a model that is thinking, and the retry then hammers a working provider (from https://github.com/meowshed/meowg1k/pull/125, high) |

## What it costs

A response that stalls after the connection is made waits until the run is
cancelled, because nothing in the transport ends it (reasoned from
crates/meow-llm/src/http.rs:44-53 and REQ-1648, low).

## What would reverse it

- A vendor starts holding connections open without sending, often enough that
  runs hang (reasoned from crates/meow-llm/src/http.rs:24, low).

## Consequences

- Cancellation is what ends a stalled call, which REQ-1648 requires (from
  docs/requirements/REQ-1648-cancellation-aborts-inflight-http.md, high).

## How I will know it was realised

1. `Http::new` builds its client with `connect_timeout(CONNECT_TIMEOUT)` equal
   to 30 seconds and no `timeout` (from crates/meow-llm/src/http.rs:44-53,
   high).
2. `a_refused_connection_names_the_provider` in crates/meow-llm/tests/http.rs
   passes (from crates/meow-llm/tests/http.rs:143, high).

## What this does not settle

- A read timeout between chunks of a stream, which would end a stall without
  ending a thinking model (reasoned from crates/meow-llm/src/http.rs:151-178,
  low).
- Whether the budget's duration axis ends a call already in flight (reasoned
  from crates/meow-agent/src/budget.rs, low).
