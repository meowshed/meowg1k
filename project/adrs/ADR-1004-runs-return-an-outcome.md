---
id: ADR-1004
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1000, REQ-1001, REQ-1002, REQ-1003, REQ-1004, REQ-1005, REQ-1006]
supersedes: []
---

# 1004. Every run returns an outcome carrying its stop reason and transcript

## Decision

Every run that starts returns an `Outcome` with a stop reason, the last
text, the step transcript, the usage and a detail string, and no stop condition
is reported as an error. (from docs/spec/agent.md [R-AGENT-001] to
[R-AGENT-005], high)

## Why

In v0.2.x `ctx.llm.agent_turn` returns a bare string, so a caller can't tell why
it stopped. Reaching `max_iterations` raises an error and discards the whole
transcript, and a model that returns empty text produces the same error. An
error discards the transcript the run already paid for. (from docs/spec/agent.md
Changes from v0.2.x and [R-AGENT-001], high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: return a bare string and raise on `max_iterations`, as v0.2.x did | The simplest return type: a caller that only wants the text gets a `str` (reasoned from docs/design/0.3.0-architecture.md section 2, low) | A caller can't tell why the run stopped, and the error discards the transcript the run paid for, also for a model that returns empty text (from docs/spec/agent.md Changes from v0.2.x, high) |
| Return an outcome for most stops and an error for cancellation, as an early draft did | Cancellation propagates like any other interrupted call (reasoned from https://github.com/meowshed/meowg1k/pull/111, low) | The two requirements contradicted each other, and an error discards the transcript, so cancellation now returns an outcome too (from https://github.com/meowshed/meowg1k/pull/111 and docs/spec/agent.md [R-AGENT-001], high) |

The specification and the history name no third shape for a run's result.

## What it costs

Every caller has to check the stop reason, because a caller that can't tell a
finished answer from a budget stop will treat a partial one as complete (from
https://github.com/meowshed/meowg1k/pull/120, high). Starlark exposes it as
`r.stop` and `r.ok` for that reason (from docs/design/0.3.0-starlark-api.md
section 5.2, high).

## What would reverse it

- A stop condition turns up that leaves no transcript worth keeping, so that an
  error would lose nothing the run paid for (reasoned from docs/spec/agent.md
  [R-AGENT-001], low).

## Consequences

`Outcome` carries `stop`, `detail`, `text`, `value`, `usage` and `steps` (from
crates/meow-agent/src/outcome.rs:20-43, high). The Starlark value adds the
session identifier (from crates/meow-star/src/value.rs:183-194, high). The
command line maps each stop reason to its own exit code: 3 for `budget`, 4 for
`cancelled`, 5 for `denied`, 8 for `tool_aborted` and 9 for `failed` (from
crates/meow-cli/src/exit.rs:40-53, high).

## How I will know it was realised

1. These tests in crates/meow-agent/tests/spec.rs pass:
   `a_budget_stop_keeps_the_text_instead_of_discarding_it`,
   `there_are_exactly_six_stop_reasons`, `the_outcome_says_which_axis_bound_it`
   and `an_empty_answer_finishes_and_says_it_was_empty` (from
   crates/meow-agent/tests/spec.rs:176-272, high).

## What this does not settle

- Which stop reason each failure maps to beyond the six names: `denied` and
  `tool_aborted` are settled by their own requirements (from
  docs/spec/agent.md [R-AGENT-006], medium).
- Where the session identifier lives. [R-AGENT-004] puts it on the outcome, and
  the engine's `Outcome` has no session field; the Starlark layer adds it (from
  crates/meow-agent/src/outcome.rs:26-43 and
  crates/meow-star/src/value.rs:194, high).
