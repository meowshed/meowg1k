---
id: ADR-1000
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1053, REQ-1054]
supersedes: []
---

# 1000. A sub-agent sees only the task and the arguments its caller passed

## Decision

A sub-agent's message list holds none of its caller's messages, and only the
task and the arguments the caller passed cross to it. (from docs/spec/agent.md
[R-AGENT-053], high)

## Why

Sharing the caller's history would make the same agent behave differently
depending on who called it, which makes it untestable and its budget
unpredictable. (from docs/spec/agent.md Decisions, high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Share the caller's history with the sub-agent | It gives a sub-agent more context (from docs/spec/agent.md Decisions, high) | The same agent would behave differently depending on who called it, which makes it untestable and its budget unpredictable (from docs/spec/agent.md Decisions, high) |

The first draft of the specification posed this as a question with exactly two
answers, sharing and isolating, and recommended isolating for a clean budget and
a reusable agent (from git show f8e58ea:docs/spec/agent.md Open questions,
high). Doing nothing wasn't an option, because the engine has to build a
sub-agent's message list one way or the other.

## What it costs

A sub-agent gets less context than sharing the caller's history would give it.
(from docs/spec/agent.md Decisions, high) The calling model pays for that in
tokens: whatever the sub-agent needs has to be written into the `task` string,
because that is the only property the sub-agent's tool schema accepts (reasoned
from crates/meow-agent/src/nested.rs:65-74, low).

## What would reverse it

- Callers routinely copy their own conversation into the `task` string to get
  a sub-agent to work, so that isolation costs more tokens than sharing would
  and still leaves the agent's behaviour dependent on its caller (reasoned from
  docs/spec/agent.md Decisions, low).

## Consequences

`SubAgent` offers one argument, `task`, and starts the child run from that
string alone (from crates/meow-agent/src/nested.rs:65-88, high). The child runs
in its own session with the caller recorded as its parent, and its spend counts
against the caller's remaining budget (from docs/design/0.3.0-starlark-api.md
section 5.3, high). Pull request 120 describes the same trade: inheriting the
caller's messages would give context and would make the agent's behaviour
depend on who called it (from https://github.com/retran/meowg1k/pull/120,
high).

## How I will know it was realised

1. `a_sub_agent_starts_from_its_task_and_nothing_else` in
   crates/meow-agent/tests/spec.rs passes: the child's provider is asked with
   exactly one message, the sub-task, and nothing of the caller's (from
   crates/meow-agent/tests/spec.rs:1049-1096, high).

## What this does not settle

- Which policy the sub-agent runs under. An agent may narrow the workspace
  policy and never widen it, which is a separate rule (from
  docs/design/0.3.0-starlark-api.md section 5.1, high).
- Structured arguments beyond `task`. The requirement says "the task and the
  arguments the caller passed", and the engine's sub-agent schema has only
  `task`, so whether a sub-agent can declare typed arguments is open (from
  crates/meow-agent/src/nested.rs:65-74, medium).
- Session state. Isolation covers the message list; whether a sub-agent reads
  the caller's `ctx.session` values isn't decided here (reasoned from
  docs/design/0.3.0-starlark-api.md section 8, low).
