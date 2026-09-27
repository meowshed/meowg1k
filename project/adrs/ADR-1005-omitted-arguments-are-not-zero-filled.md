---
id: ADR-1005
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1021, REQ-1022, REQ-1023, REQ-1024, REQ-1025, REQ-1026]
supersedes: []
---

# 1005. An omitted argument is never filled with a zero value

## Decision

A required argument the model omitted gets no default or zero value: the engine
names it to the model and continues. An omitted optional argument takes its
declared default, or stays absent. (from docs/spec/agent.md [R-AGENT-021]
[R-AGENT-022], high)

## Why

In v0.2.x `executeToolForAgentic` fills a missing argument with the zero value
for its type, so a model that omits a required integer receives `0` and returns
a confident wrong answer. (from docs/spec/agent.md Changes from v0.2.x, high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: fill a missing argument with its type's zero value, as v0.2.x did | The tool runs on the first call, with no extra turn (reasoned from docs/design/0.3.0-architecture.md section 2, low) | A model that omits a required integer receives `0` and returns a confident wrong answer (from docs/spec/agent.md Changes from v0.2.x, high) |
| Treat a missing argument as a tool error | The existing error path carries it, with no separate kind of result (reasoned from crates/meow-agent/src/engine.rs:305-310, low) | An agent with the `abort` error policy would die on the model's first typo, so a correction is not a tool error (from https://github.com/retran/meowg1k/pull/118 and docs/spec/agent.md [R-AGENT-021], high) |

The specification and the history name no third way to handle an omitted
argument.

## What it costs

A correction costs a model turn: the engine returns the message as a tool result
and the model has to call the tool again, which spends a step and its tokens
from the budget (reasoned from crates/meow-agent/src/engine.rs:407 and
crates/meow-agent/src/tool.rs:116-150, low).

## What would reverse it

- Models that are told which argument is missing still fail to supply it on
  the next turn, so corrections spend the step budget without fixing the call
  (reasoned from crates/meow-agent/src/tool.rs:105-111, low).

## Consequences

`check` returns `Checked::Correction` naming the argument and its type, or
`Checked::Ready` with defaults applied and absent optionals left out (from
crates/meow-agent/src/tool.rs:98-150, high). The engine turns a correction into
a tool result the model reads, not a tool error, so no error policy sees it
(from crates/meow-agent/src/engine.rs:305-312 and 406-407, high). The Starlark
side binds a model's arguments against the same declaration as the command
line, so a constraint holds on both paths (from
crates/meow-star/src/run.rs:901-907, high).

## How I will know it was realised

1. These tests in crates/meow-agent/tests/spec.rs pass:
   `a_missing_required_argument_is_corrected_rather_than_invented`,
   `an_argument_correction_does_not_abort_an_aborting_agent` and
   `an_absent_optional_argument_is_absent_rather_than_zero` (from
   crates/meow-agent/tests/spec.rs:401-461, high).

## What this does not settle

- How many corrections a run may take. Nothing bounds them apart from the step
  and token budget (reasoned from crates/meow-agent/src/engine.rs:305-310,
  low).
- An argument the model supplied with the wrong type, which `check` reports as
  a separate correction, "argument is wrong" (from
  crates/meow-agent/src/tool.rs:144, high).
