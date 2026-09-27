---
id: ADR-0404
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1015, REQ-1016]
supersedes: []
---

# 0404. Budget is reserved before a call, not checked and then charged

## Decision

Budget is reserved, not checked and then charged (from
https://github.com/retran/meowg1k/pull/118, high).

Once this is accepted, `Ledger::reserve_step` checks every axis and takes the
step under one lock, so no two callers can take the same last step, and
`Ledger::charge` records the tokens and cost a call actually used after it
returns (from crates/meow-agent/src/budget.rs:193-234, high). What still
doesn't work is reserving tokens and cost: only the step is reserved, and the
token and cost axes are checked before the call and charged after it, so
concurrent calls can together pass those limits (from
crates/meow-agent/src/budget.rs:198-234, medium).

## Why

Three concurrent sub-agents could each observe the same remaining budget and
each spend it (from https://github.com/retran/meowg1k/pull/111, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: check the remaining budget before a call and charge after it | One simple read before the call and one write after, with no reservation to reconcile (reasoned from crates/meow-agent/src/budget.rs:193-234, low) | Concurrent sub-agents each observe the same remaining amount and each spend it (from https://github.com/retran/meowg1k/pull/111, high) |
| Run sub-agents one at a time, so nothing shares a budget concurrently | No race exists to guard against (reasoned from crates/meow-agent/src/budget.rs:113-116, low) | Fan-out runs branches concurrently against one caller ledger, which ADR-0414 records (from docs/adrs/ADR-0414-fan-out-branches-share-the-caller-ledger.md, medium) |
| Reserve every axis, tokens and cost included, from an estimate before the call | The token and cost axes could not be overspent either (reasoned from crates/meow-agent/src/budget.rs:229-234, low) | A call's token use and cost are known only once the provider reports them, so the ledger charges them after the call (from crates/meow-agent/src/budget.rs:228-234, medium) |

## What it costs

Every reservation and every charge takes one mutex shared by a run and all its
descendants (from crates/meow-agent/src/budget.rs:117-131, high). A child
ledger has to read the shared counter once and derive its allowance from that
same reading, and reading it twice let six branches against a caller with
three steps finish four (from crates/meow-agent/src/budget.rs:164-170 and
https://github.com/retran/meowg1k/pull/144, high).

## What would reverse it

- Nothing runs against a shared ledger concurrently: fan-out and concurrent
  sub-agents are removed, so a check and a charge can't interleave with
  another run's (reasoned from crates/meow-agent/src/budget.rs:113-116, low).

## Consequences

- A step is taken before the model call and refused with the axis that bound
  it, so a run stops with `budget` before it spends, not after (from
  crates/meow-agent/src/budget.rs:193-226, high).
- The reservation scheme is correct only because a child's base plus its limit
  equals its caller's cap, which is why `child` takes the lock once (from
  https://github.com/retran/meowg1k/pull/144, high).

## How I will know it was realised

1. `concurrent_runs_cannot_each_spend_the_same_remaining_budget` in
   crates/meow-agent/tests/spec.rs passes: eight tasks race against a budget of
   five steps and exactly five are granted, whatever the interleaving (from
   crates/meow-agent/tests/spec.rs:379-397, high).

## What this does not settle

- Whether the token and cost axes may be overspent by concurrent calls: the
  code reserves only steps, the test covers only steps, and REQ-1016 speaks of
  one budget without naming an axis (from
  crates/meow-agent/src/budget.rs:229-234 and
  crates/meow-agent/tests/spec.rs:379-397, medium).
- How a child's budget is narrowed to what its caller has left: REQ-1017
  covers that (from crates/meow-agent/src/budget.rs:157-164, high).
