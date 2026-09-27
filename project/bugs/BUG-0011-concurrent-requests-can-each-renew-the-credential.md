---
id: BUG-0011
artifact: bug
status: approved
severity: minor
violates: REQ-1605
found: 2026-09-27
revised: 2026-09-27
issue: 184
---

# Concurrent requests can each renew an expired exchanged credential

`Exchanged::token` checks the held token under a lock, releases the lock, and
performs the exchange. Two requests that find the token expired at the same time
both exchange it.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Point an `Exchanged` at a local HTTP server that counts requests and answers
   each with a token and an `expires_at` an hour ahead, after a short delay.
2. Call `token` from two tasks at once with nothing held.

The server counts two exchanges.

## What the system does

`crates/meow-llm/src/bearer.rs:119-124` takes the lock in `usable`, returns, and
drops it. The exchange at `crates/meow-llm/src/bearer.rs:142-175` runs with no
lock held, and the result is stored at `crates/meow-llm/src/bearer.rs:178-183`.
Nothing makes a second caller wait for the first exchange.

## What it should do, and why

REQ-1605: "Renewing a provider's credential MUST happen at most once for a
credential's lifetime." ADR-1600 records that the credential is asked for per
request.

## Triage

The defect enters in `meow-llm`'s `Exchanged` bearer, used by the `copilot`
provider. It is minor: the engine makes one model call at a time and
`meow.parallel` runs its invocations in sequence, so two concurrent requests on
one provider are rare today. The fix is an async mutex held across the exchange,
with a second check once it is acquired.

## Closed by

A test in `crates/meow-llm/tests/`, named for example
`two_callers_share_one_renewal`, that runs the steps above and expects one
exchange.
