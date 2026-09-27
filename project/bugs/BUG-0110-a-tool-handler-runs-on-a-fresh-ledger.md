---
id: BUG-0110
artifact: bug
status: approved
severity: major
violates: [REQ-1014, REQ-1017]
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A Starlark tool's handler runs on a fresh ledger, outside its caller's budget

When a model calls a Starlark tool, the handler gets `Ledger::new(Budget::default())`,
so an agent it starts with `agent.run` spends against a new 200,000-token,
40-step budget and never against the calling agent's.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare an agent `outer` with `budget: {steps: 2}` and one Starlark tool
   `inner_run` whose handler calls `meow.agent_named("inner").run("work")`.
2. Declare `inner` with no budget.
3. Drive `outer` with a scripted model that calls `inner_run` once, and `inner`
   with one that takes ten steps.
4. `inner` takes all ten steps, although `outer` had two.

This follows from the code below.

## What the system does

`StarlarkTool::call` builds `Ledger::new(Budget::default())` at
`crates/meow-star/src/run.rs:911` and hands it to `call_handler`. `agent.run`
then makes a child of that ledger, `crates/meow-star/src/value.rs:213` and
`crates/meow-star/src/run.rs:299-302`. `tools_for` receives the caller's ledger
at `run.rs:381` and doesn't give it to the `StarlarkTool` it builds at
`run.rs:397-405`. `run_command` does the same for a top-level command at
`run.rs:221`, which is correct there because no caller exists.

## What it should do, and why

REQ-1014: "A sub-agent's spend MUST count against its caller's remaining
budget, transitively." REQ-1017: a sub-agent "MUST NOT be given a budget larger
than its caller's remaining budget." The handler should run on a child of the
calling agent's ledger.

## Triage

The defect enters where `meow-star` wraps a handler as a tool. It's major
because the budget, the one hard cap on spend a person sets, doesn't hold for
any agent reached through a tool, which is the common way a workspace composes
agents.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`an_agent_inside_a_tool_spends_its_callers_budget`, running the reproduction
above and expecting `inner` stopped with `budget` after at most the steps
`outer` had left.
