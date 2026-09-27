---
id: ADR-0120
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1014, REQ-1017]
supersedes: []
---

# 0120. The engine enforces nested budgets, so a fan-out can't exceed the top-level cap

## Decision

A sub-agent's spend counts against its caller's remaining budget, and the engine
enforces the cap (from docs/design/0.3.0-starlark-api.md section 5.3, high).
Budgets nest, so a fan-out can't exceed the top-level cap (from
docs/design/0.3.0-architecture.md section 7, high). `Ledger::child` gives the
child the tighter of what it asked for and what the caller has left, and the
child spends from the caller's own counter (from
crates/meow-agent/src/budget.rs:157-192, high).

## Why

v0.2.x had no budget, so an agent that looped cost whatever it cost (from
docs/design/0.3.0-architecture.md section 2, high). A fan-out can't exceed the
top-level cap because the engine enforces it in place of convention (from
docs/design/0.3.0-starlark-api.md section 5.3, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no budget, as in v0.2.x | No accounting in the loop and no run stopped short of its answer (reasoned from docs/design/0.3.0-architecture.md section 2, low) | An agent that loops costs whatever it costs (from docs/design/0.3.0-architecture.md section 2, high) |
| Budgets kept by convention | The engine stays simpler, and a script decides how to split its allowance (reasoned from docs/design/0.3.0-starlark-api.md section 5.3, low) | The top-level cap holds because the engine enforces it, and a convention enforces nothing (from docs/design/0.3.0-starlark-api.md section 5.3, high) |
| Each sub-agent gets its own declared budget, independent of its caller | A sub-agent behaves the same whoever calls it, with the allowance it declares (reasoned from https://github.com/retran/meowg1k/pull/120, low) | Six branches each declaring a hundred steps against a caller with three would take six hundred; sharing the ledger gives them three between them (from https://github.com/retran/meowg1k/pull/120, high) |

## What it costs

A branch made later in a fan-out gets only what earlier branches left, so how
far each branch gets depends on how the tasks interleave, and the ones that
find nothing left stop with `budget` (from
https://github.com/retran/meowg1k/pull/120 and
crates/meow-agent/tests/spec.rs:1208, high). The shared counter is a mutex
every step takes, and reading it twice while making a child let six branches
against three steps finish four until the reads were made one (from
https://github.com/retran/meowg1k/pull/144, high).

## What would reverse it

- A requirement that a sub-agent's budget be independent of its caller's,
  which would contradict REQ-1017 (reasoned from
  docs/requirements/REQ-1017-sub-agent-budget-within-remaining.md, low).

## Consequences

A parent's totals include its children's, which is how a nested budget is
enforced and how `meow session show` tells you where the money went (from
docs/design/0.3.0-sessions.md section 6.1, high). Each ledger remembers what the
counter read when it was made and measures from there, so a relative budget and
the cumulative counter are in the same units (from
https://github.com/retran/meowg1k/pull/134, high).

## How I will know it was realised

1. `a_child_spends_its_callers_budget_and_cannot_exceed_it` in
   crates/meow-agent/tests/spec.rs passes (from
   crates/meow-agent/tests/spec.rs:303, high).
2. `a_fan_out_cannot_spend_more_than_the_caller_has`,
   `a_child_cannot_outspend_its_caller` and
   `making_a_child_while_steps_are_spent_cannot_widen_its_budget` in the same
   file pass (from crates/meow-agent/tests/spec.rs:1161, 1265 and 1294, high).

## What this does not settle

- The cost axis: `Budget` carries `cost_micros` and no provider reports a cost,
  so a declared cost cap does nothing, tracked by #168 (from docs/philosophy.md
  section 10, high).
- Rate limiting: the rewrite dropped the request ceilings v0.2.x had, tracked by
  #167 (from docs/philosophy.md section 10, high).
- If the ledger's lock is poisoned, `Ledger::child` gives the child the budget
  it asked for, unnarrowed (from crates/meow-agent/src/budget.rs:170-176,
  high).
