---
id: BUG-0128
artifact: bug
status: approved
severity: minor
violates: REQ-1625
found: 2026-09-27
revised: 2026-09-27
issue: 251
---

# A throttled or unavailable credential renewal is fatal and blames the grant

Any non-success answer to the Copilot token exchange becomes `LlmError::Auth`,
which is `Fatal`, so a 429 or a 503 from the exchange ends the run and tells the
person to run `meow auth login` again.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Point an `Exchanged` credential at a mock endpoint that answers `503
   Service Unavailable`.
2. Ask it for a token.
3. It returns `Auth` with "the credential would not renew (503 Service
   Unavailable); run `meow auth login copilot`", and `LlmError::class` puts it
   in `Class::Fatal`.

This follows from the code below.

## What the system does

`crates/meow-llm/src/bearer.rs:161-171` maps every non-success status to
`LlmError::Auth`. `crates/meow-llm/src/error.rs:140` classes every `Auth` as
`Fatal`.

## What it should do, and why

REQ-1625: "HTTP 408, 429, 500, 502, 503, 504, and connection or read timeouts
MUST classify as `Transient`." A renewal refused with 401 or 403 is the revoked
grant ADR-0462 means; a 429 or 5xx is the exchange being busy and should be
retried, with a message that doesn't send the person to log in again.

## Triage

The defect enters in `Exchanged::token`. It's minor: it affects one provider
kind and one request, and BUG-0005 means no provider call is retried yet, so
the classification only changes the message until that is fixed.

## Closed by

A test in `crates/meow-llm/tests/bearer.rs`, named for example
`a_503_on_renewal_is_transient`, expecting a `Transient` error for 503 and 429
and `Auth` for 401.
