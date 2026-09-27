---
id: ADR-1002
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1047, REQ-1048]
supersedes: []
---

# 1002. The compaction policy names its own model and falls back to the agent's

## Decision

The compaction policy accepts a model of its own and falls back to the agent's
model when none is given. (from docs/spec/agent.md [R-AGENT-045], high)

## Why

Summarising is a cheap task. Running it on the agent's model would run it on an
expensive model at the worst possible moment, on every long run, and that cost
is why the policy can name its own model. (from docs/spec/agent.md Decisions,
high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: leave the compaction model unspecified, so it summarises on the agent's model, as the first draft did | One model to declare and nothing to fall back from (reasoned from crates/meow-agent/src/compaction.rs:20-25, low) | It runs a cheap task on the agent's expensive model at the worst possible moment, on every long run (from docs/spec/agent.md Decisions and https://github.com/meowshed/meowg1k/pull/111, high) |

The trade-off pass in pull request 111 recorded only the first answer,
"Unspecified", and the answer that replaced it; no third option appears in the
specification, the design documents or the history (from
https://github.com/meowshed/meowg1k/pull/111, high).

## What it costs

A workspace that wants the saving has to declare a second model and point the
compaction policy at it; one that doesn't still summarises on the agent's model
(reasoned from crates/meow-star/src/agent.rs:61-83, low).

## What would reverse it

- Summaries from a cheaper model measurably lose what the next turn needs, so
  that runs compacted on the named model fail where the agent's own model would
  have finished (reasoned from docs/spec/agent.md Decisions, low).

## Consequences

`Compaction::model` is an `Option<String>`, and `None` falls back to the
agent's own model (from crates/meow-agent/src/compaction.rs:20-25 and
crates/meow-agent/src/engine.rs:186-190, high). A declaration sets it as
`compaction = {"model": ...}` on `meow.agent`, and an omitted field keeps the
default (from crates/meow-star/src/agent.rs:61-83, high).

## How I will know it was realised

1. `compaction_uses_its_own_model_when_given_one` in
   crates/meow-agent/tests/spec.rs passes: the agent's first request names the
   agent's model and the summary request names the compaction model (from
   crates/meow-agent/tests/spec.rs:944-1003, high).
2. A test that runs a compaction with no model named and sees the agent's model
   on the summary request. No such test exists yet; the fallback is visible
   only in crates/meow-agent/src/engine.rs:186-190 (from
   crates/meow-agent/tests/spec.rs, high).

## What this does not settle

- Whether the compaction model has to come from the same provider as the
  agent's. The command line builds one engine per provider and the engine sends
  the summary request through that provider, so a compaction model another
  provider serves would fail the compaction (from
  crates/meow-cli/src/wire.rs:1406 and crates/meow-agent/src/engine.rs:186-205,
  medium).
