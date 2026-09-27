---
id: ADR-0466
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1216, REQ-2451, REQ-1820, REQ-1648]
supersedes: []
---

# 0466. Cancellation is checked before a request as well as raced against it

## Decision

The credential exchange, the device-code flow and `@std//http` each check
`is_cancelled` before entering the `select`, and the `select` stays (from
https://github.com/meowshed/meowg1k/pull/161, high). The same fix went into the
package fetch on its own branch (from
https://github.com/meowshed/meowg1k/pull/161, high). The first submission,
https://github.com/meowshed/meowg1k/pull/160, closed without merging and carried
the same change (from https://github.com/meowshed/meowg1k/pull/160, high).

Once this is accepted, an already-cancelled token stops the call before any
request is built in four places: `Exchanged::token` in `meow-llm`, the
device-code `run`, `@std//http` and the package `fetch` (from
crates/meow-llm/src/bearer.rs:135, crates/meow-cli/src/device.rs:136,
crates/meow-star/src/capability_http.rs:182 and crates/meow-cli/src/fetch.rs:93,
high). The provider gateways already did both, following `Engine::generate`
(from https://github.com/meowshed/meowg1k/pull/160 and
crates/meow-llm/src/anthropic.rs:279, high). A token cancelled after the check
and before the `select` still races, and the `select` resolves it (from
https://github.com/meowshed/meowg1k/pull/161, high).

## Why

`tokio::select!` polls its branches in an unspecified order, so against a fast
server a request completes first even though the token was already cancelled,
and a cancelled run makes a request (from
https://github.com/meowshed/meowg1k/pull/161, high). The `select` stops a request
already in flight, and the check stops one from starting (from
https://github.com/meowshed/meowg1k/pull/161, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: race the cancellation in the `select` alone | One cancellation point per call, and it passed on the author's machine, where the server was slower than the scheduler (from https://github.com/meowshed/meowg1k/pull/161, high) | An already-cancelled run can still make a request (from https://github.com/meowshed/meowg1k/pull/161, high) |
| Check before the request and drop the `select` | Simpler code, with no race at all for an already-cancelled token (reasoned from crates/meow-llm/src/bearer.rs:135, low) | The `select` is what stops a request already in flight (from https://github.com/meowshed/meowg1k/pull/161, high) |

The code and the history name no third option.

## What it costs

Each call that makes a request carries two cancellation points, the check and
the `select`, and a new call written without the check reintroduces the defect
with no compiler help (reasoned from
crates/meow-star/src/capability_http.rs:179, low). The four sites repeat the same comment, because nothing shares the
pattern (from crates/meow-llm/src/bearer.rs:130 and
crates/meow-cli/src/fetch.rs:90, high).

## What would reverse it

A helper that every request goes through, checking the token and racing it in
one place, would replace the four copies (reasoned from
crates/meow-llm/src/bearer.rs:130, low).

## Consequences

- `cancelling_stops_a_renewal` no longer depends on the scheduler: it failed on
  Linux and Windows runners and passed locally before the fix (from
  https://github.com/meowshed/meowg1k/pull/161, high).
- An already-cancelled call returns its crate's cancellation error:
  `LlmError::Cancelled`, `DeviceError::Cancelled`, `FetchError::Cancelled`, or
  "the run was cancelled" from `http` (from crates/meow-llm/src/bearer.rs:136,
  crates/meow-cli/src/device.rs:137, crates/meow-cli/src/fetch.rs:94 and
  crates/meow-star/src/capability_http.rs:183, high).

## How I will know it was realised

Twenty-five consecutive runs of `cancelling_stops_a_renewal` pass, and twenty of
`a_connection_that_is_refused_fails` (from
https://github.com/meowshed/meowg1k/pull/161, high). The tests are
`cancelling_stops_a_renewal` in `crates/meow-llm/tests/bearer.rs`,
`a_cancelled_fetch_does_not_start` in `crates/meow-cli/tests/fetch.rs` and
`cancelling_stops_the_flow` in `crates/meow-cli/tests/device.rs` (from
crates/meow-llm/tests/bearer.rs:197, crates/meow-cli/tests/fetch.rs:365 and
crates/meow-cli/tests/device.rs:198, high). No test calls `@std//http` with a
token already cancelled (reasoned from crates/meow-star/tests/running.rs:2205,
low).

## What this does not settle

- A token cancelled in the microseconds between the check and the `select` still
  races, and the `select` resolves it (from
  https://github.com/meowshed/meowg1k/pull/161, high).
- Whether a test should check that a cancelled `@std//http` call makes no
  request, since none does (reasoned from
  crates/meow-star/tests/running.rs:2205, low).
