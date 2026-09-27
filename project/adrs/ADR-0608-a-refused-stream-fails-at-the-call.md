---
id: ADR-0608
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1651]
supersedes: []
---

# 0608. A streaming request the vendor refuses fails at the call

## Decision

When a vendor answers a streaming request with a status other than success,
`post_lines` returns the error with the vendor's body, and never a channel
(from crates/meow-llm/src/http.rs:128-142, high). The pull request describes
this as "a stream that fails before its first line"; the code checks the
status, so a stream that starts and then breaks reports through the channel,
and I've written REQ-1651 to what the code does (from
https://github.com/meowshed/meowg1k/pull/125 and
crates/meow-llm/src/http.rs:171-178, high).

## Why

A failed stream never produces a line, so a caller waiting on its channel
can't tell that from a model that is thinking (from
https://github.com/meowshed/meowg1k/pull/125 and
crates/meow-llm/src/http.rs:130-133, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Report the status through the channel, as its first item | One path for every failure, before or during the stream (reasoned from crates/meow-llm/src/http.rs:171-178, low) | The caller has to read the channel to find out the request was refused, and a caller that waits on it first looks hung (from https://github.com/meowshed/meowg1k/pull/125, high) |

Doing nothing isn't an option here, because the transport was new in #125 and
had to do one or the other (from https://github.com/meowshed/meowg1k/pull/125,
high).

## What it costs

A caller handles a refusal in two places, the call's result and the channel's
items, because a stream that breaks after it starts still reports through the
channel (from crates/meow-llm/src/http.rs:171-178, high).

## What would reverse it

- The `Transport` trait changes so a stream carries its status apart from its
  lines (reasoned from crates/meow-llm/src/transport.rs, low).

## Consequences

- The provider reads the status and the vendor's error object from the error
  and classifies it before retrying, as for a non-streaming call (from
  crates/meow-llm/src/http.rs:134-141 and ADR-1603, high).

## How I will know it was realised

1. `a_failed_stream_fails_instead_of_hanging` in crates/meow-llm/tests/http.rs
   passes (from crates/meow-llm/tests/http.rs:114-117, high).

## What this does not settle

- `retry_after` on this error is always `None`, although the non-streaming path
  reads `Retry-After` (from crates/meow-llm/src/http.rs:134-141 and :101,
  high).
