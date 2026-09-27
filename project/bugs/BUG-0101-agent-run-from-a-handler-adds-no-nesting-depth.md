---
id: BUG-0101
artifact: bug
status: approved
severity: minor
violates: REQ-1051
found: 2026-09-27
revised: 2026-09-27
issue: 224
---

# `agent.run` from a tool handler adds no nesting depth

An agent whose Starlark tool calls `agent.run` starts the inner agent at the
same depth as the tool, so a chain of agent, tool, agent, tool can grow without
the depth limit ever refusing it.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare an agent `loop` whose `tools` list names a Starlark tool `again`.
2. Write `again`'s handler to call `meow.agent_named("loop").run("go on")`.
3. Drive `loop` with a scripted model that always calls `again`.
4. The run nests until the shared budget stops it; the depth limit of 3 never
   refuses a level.

This follows from the code below.

## What the system does

`crates/meow-star/src/value.rs:213` calls `run_agent` with `state.depth`, the
handler's own depth, and `run_agent` builds the spec at that same depth at
`crates/meow-star/src/run.rs:298`. The tool's handler runs at the tool's depth,
`crates/meow-star/src/run.rs:912-917`. No step adds one.

## What it should do, and why

REQ-1051: "The engine MUST refuse to start a sub-agent that would exceed the
configured maximum nesting depth." An agent started from inside another agent's
tool is a sub-agent of it, so it should count as one more level.

## Triage

The defect enters in `agent.run` in `meow-star`. ADR-0423 lists it under "What
this does not settle". It's minor because the shared step and token budget
still ends the chain, so it can't run forever, but the depth limit it was meant
to meet doesn't hold. Whether a handler level counts is a requirements question
REQ-1051 answers only by implication.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`agent_run_inside_a_tool_counts_as_a_level`, that builds the loop above and
expects a refusal at depth 3 before the budget runs out.
