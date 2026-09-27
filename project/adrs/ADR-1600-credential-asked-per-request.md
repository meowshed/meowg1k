---
id: ADR-1600
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1604, REQ-1605, REQ-1606]
supersedes: []
---

# 1600. A provider asks for its credential on each request

## Decision

A provider obtains what it sends to authenticate per request, where before it
held a token built in when it was created, so a credential that expires is
renewed without rebuilding the provider [REQ-1604] (from docs/spec/llm.md,
high).

The `Bearer` trait carries it: `Fixed` hands back a key that doesn't change,
and `Exchanged` swaps a long-lived grant for a short-lived token, reuses the
token until 60 seconds before it expires, and renews it after that [REQ-1605]
(from crates/meow-llm/src/bearer.rs:18-47, 86-140, high). A renewal the
exchange refuses fails with `LlmError::Auth`, naming the provider and
`meow auth login <provider>` [REQ-1606] (from
crates/meow-llm/src/bearer.rs:161-170, high). Copilot is the one provider
built on `Exchanged`; every other provider uses `Fixed` (from
crates/meow-llm/src/copilot.rs:47-57, high).

## Why

Every provider here but one uses a key that doesn't change, and building the
token into the provider was right for them. It is wrong for one that exchanges a
long-lived grant for a short-lived token, because that provider would have to be
rebuilt on a schedule nobody owns (from docs/spec/llm.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: build the token into the provider when it is created, as `OpenAi::new` still does for a fixed key | Nothing to do per request, and right for every provider whose key doesn't change (from docs/spec/llm.md, high) | A provider that exchanges a long-lived grant for a short-lived token would have to be rebuilt on a schedule nobody owns (from docs/spec/llm.md, high) |
| Rebuild the provider when its token expires | Provider values stay immutable once built (reasoned from crates/meow-llm/src/openai.rs, low) | The rebuild runs on a schedule nobody owns (from docs/spec/llm.md, high) |
| Renew inside one vendor's own gateway, as v0.2.x's `copilot.go` did with `refreshTokenIfNeeded` | Touches no shared trait, and holds a mutex across the exchange so concurrent requests renew once (from `git show v0.2.1:internal/adapters/gateway/copilot.go`, lines 161-168, high) | Copilot's request shape is OpenAI's, so a gateway of its own is a sixth copy of a streaming parser; `with_bearer` on the OpenAI provider needed no other call site changed (from https://github.com/meowshed/meowg1k/pull/156, high) |

## What it costs

Asking per request costs a lock and a comparison for the providers that don't
need it (from docs/spec/llm.md, high). Every request also makes one async
`token` call through a trait object (from crates/meow-llm/src/bearer.rs:19-29,
high), and `Exchanged` builds a `reqwest` client of its own for the exchange
(from crates/meow-llm/src/bearer.rs:101-107, high).

The lock guards only the read and the write of the held token, never the
exchange, so two requests that find the token expired at the same moment each
renew it (from crates/meow-llm/src/bearer.rs:119-124, 177-182, medium). v0.2.x
held its mutex across the exchange and didn't pay this (from
`git show v0.2.1:internal/adapters/gateway/copilot.go`, lines 163-165, high).

## What would reverse it

- Copilot is removed or its exchange stops returning an expiry, so
  `Exchanged` has no user and every `Bearer` in
  `crates/meow-llm/src/bearer.rs` is a `Fixed` (reasoned from
  crates/meow-llm/src/copilot.rs:47, low).

## Consequences

It removes a whole category of "it worked this morning" failures from the
provider that exchanges a grant for a short-lived token (from docs/spec/llm.md,
high).

- A refused renewal is `Fatal` and surfaces at once with the command to run,
  which ADR-0462 records (from crates/meow-llm/src/error.rs:140-148, high).
- A cancelled run stops a renewal before and during the exchange, so a
  cancelled run never sends a request (from
  crates/meow-llm/src/bearer.rs:130-152, high;
  https://github.com/meowshed/meowg1k/pull/161, high).
- An exchange that answers without `expires_at` gets an expiry of 0, so its
  token is renewed on every request (from
  crates/meow-llm/src/bearer.rs:123, 189-195, medium).

## How I will know it was realised

1. `crates/meow-llm/tests/bearer.rs` passes:
   `a_token_that_has_not_expired_is_not_fetched_again` counts one exchange for
   two requests, `an_expired_token_is_renewed` counts a second,
   `a_refused_renewal_says_what_to_run` checks the message, and
   `cancelling_stops_a_renewal` checks cancellation (from
   crates/meow-llm/tests/bearer.rs:102-209, high).
2. No test covers two concurrent requests with an expired token, so
   [REQ-1605] under concurrency is unverified (from
   crates/meow-llm/tests/bearer.rs, high).

## What this does not settle

- Where the grant is stored and how it is obtained: ADR-0467 and ADR-1200
  decide that (from docs/spec/auth.md, medium).
- Whether a failed renewal is retried: ADR-0462 decides it isn't (from
  crates/meow-llm/src/error.rs:140-148, high).
- Whether concurrent renewals share one exchange (from
  crates/meow-llm/src/bearer.rs:119-124, medium).
