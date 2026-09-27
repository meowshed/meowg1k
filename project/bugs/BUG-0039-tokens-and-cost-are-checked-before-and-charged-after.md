---
id: BUG-0039
artifact: bug
status: approved
severity: minor
violates: REQ-1016
found: 2026-09-27
revised: 2026-09-27
issue:
---

# Tokens and cost are checked before a call and charged after, so concurrent calls can overspend

`Ledger::reserve_step` reserves a step under the lock, and for tokens and cost
it only checks that the spend is under the limit. The call's usage is charged
afterwards by `Ledger::charge`. Several invocations sharing one ledger each see
the same remaining tokens, each proceed, and together spend past the cap.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Create a `Ledger` with `Budget { tokens: Some(100), .. }`, standing for two
   invocations that share it.
2. Call `reserve_step` twice. Both calls succeed.
3. Call `charge` twice, each with a `Usage` of 90 tokens.

The shared spend is 180 against a cap of 100.

## What the system does

`crates/meow-agent/src/budget.rs:198-227` checks the token and cost axes with
`>=` against what has already been charged and increments only `steps`.
`crates/meow-agent/src/budget.rs:230-235` adds the real usage later.

## What it should do, and why

REQ-1016: "Concurrent invocations sharing one budget MUST NOT be able to
overspend it by each observing the same remaining amount." ADR-0404 records
that only steps are reserved.

## Triage

The defect enters in `meow-agent`'s `Ledger`. It is minor and dormant: the
engine runs a turn's tool calls one after another, and `meow.parallel` runs its
invocations in sequence, so no two invocations share a ledger at the same time
today. A call's token use isn't known before it is made, so the fix needs a
reservation, for example the call's `max_output_tokens` plus its prompt
estimate, reconciled on `charge`.

## Closed by

A test in `crates/meow-agent/tests/`, named for example
`concurrent_children_cannot_overspend_tokens`, that runs the steps above and
expects the second reservation refused.
