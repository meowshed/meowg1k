---
id: BUG-0012
artifact: bug
status: approved
severity: minor
violates: REQ-1605
found: 2026-09-27
revised: 2026-09-27
issue: 185
---

# A renewal answer without `expires_at` makes every request renew

`Renewed::expires_at` defaults to 0 when the exchange answer omits it. A token
held with an expiry of 0 is never usable, so every later request exchanges the
grant again.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Point an `Exchanged` at a local HTTP server that counts requests and answers
   `{"token": "t"}` with no `expires_at`.
2. Call `token` three times in sequence.

The server counts three exchanges.

## What the system does

`crates/meow-llm/src/bearer.rs:190-195` declares `expires_at` with
`#[serde(default)]`, so a missing field reads as 0.
`crates/meow-llm/src/bearer.rs:122` treats a held token as usable only when
`expires - EARLY > now_secs()`, which is false for 0.

## What it should do, and why

REQ-1605: "Renewing a provider's credential MUST happen at most once for a
credential's lifetime." A token whose lifetime the server didn't state has one
all the same, and renewing it per request both breaks the requirement and sends
the long-lived grant on every call.

## Triage

The defect enters in the parsing of the exchange answer in `meow-llm`'s
`Exchanged`. It is minor, because GitHub's Copilot token endpoint sends
`expires_at` today. The fix is to refuse an answer without an expiry, or to hold
the token until a request is refused with 401.

## Closed by

A test in `crates/meow-llm/tests/`, named for example
`a_token_without_an_expiry_is_not_renewed_per_request`, that runs the steps
above and expects one exchange, or a clear error.
