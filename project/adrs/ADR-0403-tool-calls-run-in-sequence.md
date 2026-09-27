---
id: ADR-0403
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1032]
supersedes: []
---

# 0403. Tool calls in one response run in the order the model returned them

## Decision

`R-AGENT-025` executes tool calls in the order the model returned them, a
trade-off accepted rather than solved (from
https://github.com/retran/meowg1k/pull/111, high).

Once this is accepted, the engine walks a response's tool calls in a plain
loop, deciding policy, running the tool and recording the result for one call
before it starts the next (from crates/meow-agent/src/engine.rs:284-331,
high). Nothing runs in parallel within one response, and nothing in this
decision is left unbuilt (reasoned from the same lines, low).

## Why

Sequential execution keeps policy evaluation and session writes simple (from
https://github.com/retran/meowg1k/pull/111, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Run the tool calls of one response in parallel | The latency win that parallel tool calls exist for (from https://github.com/retran/meowg1k/pull/111, high) | Sequential execution keeps policy evaluation and session writes simple (from https://github.com/retran/meowg1k/pull/111, high) |
| Run reads in parallel and writes in sequence, using the read-and-write distinction of `R-POLICY-008` and `R-POLICY-009` | Most of the latency win on read-heavy turns, without two writes racing (reasoned from https://github.com/retran/meowg1k/pull/111, low) | The history names it as the opening to revisit this decision, not as the choice for now (from https://github.com/retran/meowg1k/pull/111, high) |

Doing nothing is the decision itself: v0.2.x also ran a response's tool calls
one after another in a `for` loop, so keeping the prior behaviour and choosing
sequential execution are the same option (from
`internal/core/starlark/module_llm.go` at tag v0.2.1, lines 589-600, high).

## What it costs

The run forfeits the latency win that parallel tool calls exist for (from
https://github.com/retran/meowg1k/pull/111, high). The pull request that built
the engine repeats the cost and records it in `agent.md` as accepted, not
solved (from https://github.com/retran/meowg1k/pull/118, high).

## What would reverse it

- The read-and-write distinction in `R-POLICY-008` and `R-POLICY-009` is the
  opening to revisit this (from https://github.com/retran/meowg1k/pull/111,
  high).
- A run's tool calls within one response come to take more wall time than its
  model calls, which is when the forfeited latency is worth the complexity
  (reasoned from https://github.com/retran/meowg1k/pull/111, low).

## Consequences

- `ToolStart` events reach the sink in the order the model returned the calls
  (from crates/meow-agent/tests/spec.rs:499-530, high).
- A call that stops the run, by a denial under `abort`, a failing tool under
  `abort` or an approval answered "stop", returns before the later calls in the
  same response start, so those never run (from
  crates/meow-agent/src/engine.rs:312-327, high).
- At most one approval prompt is open at a time, because the engine asks the
  approver inside the same loop (from crates/meow-agent/src/engine.rs:369-386,
  medium).

## How I will know it was realised

1. `tool_calls_run_in_the_order_they_arrived` in
   crates/meow-agent/tests/spec.rs passes: two calls in one response produce
   `ToolStart` events for `first` then `second` (from
   crates/meow-agent/tests/spec.rs:499-530, high).

## What this does not settle

- Whether sub-agents run concurrently: the budget ledger is built for
  concurrent sub-agents, and this decision covers only the calls in one
  response (from https://github.com/retran/meowg1k/pull/111 and
  crates/meow-agent/src/engine.rs:284, medium).
- What the model is told about the calls after one that stopped the run: the
  engine returns without a result for them (from
  crates/meow-agent/src/engine.rs:312-327, medium).
