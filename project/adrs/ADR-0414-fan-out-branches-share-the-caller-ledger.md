---
id: ADR-0414
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1017, REQ-1057, REQ-1058, REQ-1059]
supersedes: []
---

# 0414. A fan-out's branches share the caller's ledger, and the rest stop

## Decision

A fan-out shares the caller's ledger: six branches each declaring a hundred
steps, against a caller with three, get three between them, and the rest stop
rather than being silently dropped (from
https://github.com/retran/meowg1k/pull/120, high). `run_parallel` gives each
branch `caller.child(spec.budget)`, a ledger narrowed to what the caller has
left and charging the caller's shared counter, and a branch that starts after
the budget is spent is stopped by its own first reservation (from
crates/meow-agent/src/nested.rs:151-175, high).

Once this is accepted, a fan-out run through the engine can't spend more steps
than its caller had, and each branch it couldn't pay for comes back as an
outcome with the `budget` stop reason (from crates/meow-agent/tests/spec.rs
`a_fan_out_cannot_spend_more_than_the_caller_has`, high). What still doesn't
work is a fan-out from Starlark: `meow.parallel` runs its invocations one after
another through `run_agent` and never calls `run_parallel` (from
crates/meow-star/src/value.rs:309-323, high).

## Why

A caller's budget is a cap on everything done on its behalf, so a branch's own
declaration can narrow its share and never widen it (from
docs/design/0.3.0-architecture.md section 7, high). A branch that is dropped
disappears without a reason, while a stopped branch is an outcome whose stop
reason says the budget ran out, and a caller that can't tell a finished answer
from a budget stop treats a partial one as complete (from
https://github.com/retran/meowg1k/pull/120, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: each branch runs on its own declared budget | Each branch behaves the same whoever calls it and however the scheduler interleaves the branches (reasoned from crates/meow-agent/src/budget.rs:164-192, low) | Six branches declaring a hundred steps each would spend six hundred against a caller allowed three, which breaks the top-level cap ADR-0120 records (from https://github.com/retran/meowg1k/pull/120, high) |
| Drop the branches the budget can't cover | The result list holds only finished work, so a caller has nothing to filter (reasoned from crates/meow-agent/src/nested.rs:151-190, low) | They disappear silently, where a stopped branch reports why (from https://github.com/retran/meowg1k/pull/120, high) |
| Check the remaining budget in `run_parallel` before starting each branch | A branch the budget can't pay for never starts a task (reasoned from crates/meow-agent/src/nested.rs:151-175, low) | A check outside the ledger could race with the branches already spending, so the first reservation inside the ledger decides instead (from crates/meow-agent/src/nested.rs:164-166, high) |

## What it costs

Which branches finish depends on how the tasks happen to interleave. The test
asserts that three of six finish and not which three (from
crates/meow-agent/tests/spec.rs
`a_fan_out_cannot_spend_more_than_the_caller_has`, high), so the same fan-out
can report different branches as finished on two runs (reasoned from
crates/meow-agent/src/nested.rs:151-175, low). Sharing one counter across
concurrent branches also made the ledger subtle enough to break twice: a
relative child budget checked against a cumulative counter refused steps the
caller still had (from https://github.com/retran/meowg1k/pull/134, high), and a
child ledger read the shared counter twice and let six branches take four of
three steps (from https://github.com/retran/meowg1k/pull/144, high).

## What would reverse it

- A caller needs every branch of a fan-out to run to completion even when that
  exceeds the caller's own budget, and a requirement says so, contradicting
  REQ-1058 (reasoned from docs/spec/agent.md [R-AGENT-062], low).

## Consequences

- `Ledger::child` measures a branch's spend from the counter reading taken when
  the branch was made, so the relative allowance and the cumulative counter
  are in the same units (from https://github.com/retran/meowg1k/pull/134,
  high).
- `Ledger::child` takes the lock once, so the allowance and the base describe
  the same instant (from crates/meow-agent/src/budget.rs:164-192, high).
- `steps_taken` on a child counts that ledger's own steps, not the shared total,
  so a sub-agent's first step is step one (from
  https://github.com/retran/meowg1k/pull/134, high).

## How I will know it was realised

1. `a_fan_out_cannot_spend_more_than_the_caller_has` in
   crates/meow-agent/tests/spec.rs passes: three of six branches finish and the
   other three stop with `StopReason::Budget`.
2. `a_branch_made_later_still_gets_what_is_left` and
   `making_a_child_while_steps_are_spent_cannot_widen_its_budget` in the same
   file pass, the second over two hundred rounds of six threads.

## What this does not settle

- Only the step axis has a test in a fan-out, because only it fails
  deterministically without a clock; cost and duration get the same correction
  without one (from https://github.com/retran/meowg1k/pull/134, high).
- Whether `meow.parallel` runs its invocations concurrently: it runs them in
  sequence, sharing the ledger, until it can use the engine's fan-out (from
  crates/meow-star/src/value.rs:309-312, high).
