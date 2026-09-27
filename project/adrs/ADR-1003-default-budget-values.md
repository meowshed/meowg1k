---
id: ADR-1003
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1013, REQ-1019]
supersedes: []
---

# 1003. The default budget is 200,000 tokens, 40 steps and 30 minutes, with cost unbounded

## Decision

A spec with no budget runs with 200,000 tokens, 40 steps and 30 minutes, and no
cost cap. (from docs/spec/agent.md [R-AGENT-016], high)

## Why

Tokens and steps are what bound cost, and they are set so a runaway loop costs
cents, not dollars. Wall clock bounds patience, not spend: a slow provider or a
long tool makes it fire on a run that is working, turning a good answer into a
`budget` stop with partial results. Thirty minutes still stops a wedged run
without punishing a slow one. Cost stays unbounded because capping it means
estimating the price of a call before making it, and the token cap is the same
guard with fewer moving parts. (from docs/spec/agent.md Decisions, high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no default budget, as v0.2.x had, where `max_iterations` was the only bound | Never stops a working run on a limit its author didn't choose (reasoned from docs/spec/agent.md Changes from v0.2.x, low) | An agent that loops costs whatever it costs (from docs/design/0.3.0-architecture.md section 2, high), and a spec with no budget has to take a bounded default rather than run unbounded (from docs/spec/agent.md [R-AGENT-012], high) |
| A five-minute wall clock, the first number chosen | Stops a wedged run sooner (reasoned from docs/spec/agent.md Decisions, low) | Five minutes is less than 40 steps of a slow model, so it turns a working run into a `budget` stop (from docs/spec/agent.md Decisions and https://github.com/retran/meowg1k/pull/111, high) |
| A default cost cap | Bounds spend in money, the unit a person pays in (reasoned from docs/spec/agent.md Decisions, low) | It means estimating the price of a call before making it, and the token cap is the same guard with fewer moving parts (from docs/spec/agent.md Decisions, high) |

## What it costs

A wedged run can take up to 30 minutes before the wall clock stops it. (from
docs/spec/agent.md Decisions, medium) A run that needs more than 40 steps or
200,000 tokens stops with `budget` unless its author declares a larger budget
(reasoned from crates/meow-agent/src/budget.rs:29-45, low).

## What would reverse it

- Real runs show the defaults binding: working runs stop with `budget` on the
  steps or tokens axis, or a runaway loop under the default costs dollars and
  not cents. The first draft called the numbers a product decision that wants
  real runs behind it, and the trade-off pass listed the default budget among
  four decisions that want a measurement (from git show
  f8e58ea:docs/spec/agent.md Open questions and
  https://github.com/retran/meowg1k/pull/111, high).

## Consequences

`Budget::default()` holds `tokens: Some(200_000)`, `steps: Some(40)`,
`duration: 30 * 60` seconds and `cost_micros: None` (from
crates/meow-agent/src/budget.rs:29-45, high). `AgentSpec::new` starts from that
default (from crates/meow-agent/src/spec.rs:107, high), and a Starlark
declaration that names some axes keeps the default for the others, so
`budget: {steps: 5}` tightens one axis without removing the other three (from
crates/meow-star/src/agent.rs:40-58, high).

## How I will know it was realised

1. `the_default_budget_is_the_one_the_specification_names` in
   crates/meow-agent/tests/spec.rs passes (from
   crates/meow-agent/tests/spec.rs:366-378, high).
2. `an_unset_axis_is_unbounded_and_the_default_is_bounded` in the same file
   passes (from crates/meow-agent/tests/spec.rs:272-288, high).

## What this does not settle

- How a Starlark declaration makes one axis unbounded. `BudgetFields::build`
  fills every omitted field from the default, so a declaration has no way to
  leave an axis unset, which sits against [R-AGENT-012]'s "a budget axis left
  unset MUST be unbounded" (from crates/meow-star/src/agent.rs:40-58 and
  docs/spec/agent.md [R-AGENT-012], medium).
- The budget of a handler run as a tool inside an agent. That path makes a
  fresh default ledger, not a child of the caller's (from
  crates/meow-star/src/run.rs:911, high), and whether that meets
  [R-AGENT-013]'s transitive accounting isn't decided here.
