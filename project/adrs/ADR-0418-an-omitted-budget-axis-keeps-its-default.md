---
id: ADR-0418
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1012, REQ-1013, REQ-2489, REQ-2528]
supersedes: []
---

# 0418. An omitted budget axis in a declaration keeps the engine's default

## Decision

In a Starlark agent declaration, a `budget` that omits an axis takes the
default budget's value for that axis, so `budget: {steps: 5}` tightens one axis
without quietly removing the other three (from
https://github.com/meowshed/meowg1k/pull/122 and crates/meow-star/src/agent.rs,
high). The engine's own `Budget` type keeps REQ-1012: an axis left unset there
is unbounded. The two don't conflict, because the declaration fills every axis
before it reaches the engine (from crates/meow-star/src/agent.rs, high).

Once accepted, a declared budget is never looser than the default on an axis it
didn't name. What still doesn't work: the default leaves cost unbounded, and
the cost axis can't fire until models carry a price (issue #168).

## Why

A declaration that names one axis means to tighten that axis, not to lift the
others (from https://github.com/meowshed/meowg1k/pull/122, medium). The design
makes a bounded run the default and says an unbounded one requires saying so
(from docs/design/0.3.0-starlark-api.md section 5.1, the `budget` row, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| An omitted axis becomes unbounded | It follows REQ-1012 as written, and a declaration can lift an axis by leaving it out (from docs/spec/agent.md [R-AGENT-012], high) | `budget: {steps: 5}` would quietly remove the other three bounds (from https://github.com/meowshed/meowg1k/pull/122, high) |
| Do nothing: `meow.agent` takes no `budget`, so every declared agent runs on the engine's default | Nothing to decide about a partial budget (reasoned from crates/meow-agent/src/budget.rs:28-45, low) | REQ-2489 requires `meow.agent` to accept `budget` (from docs/spec/starlark.md [R-STAR-040], high) |

No third option appears in the code, the history or the design documents.

## What it costs

A Starlark or markdown declaration can't make a bounded axis unbounded:
`BudgetFields::build` fills every missing or null field from
`Budget::default`, and there is no value that means "no limit" (from
crates/meow-star/src/agent.rs:41-60, high). Only cost, whose default is already
unbounded, can stay unbounded (from crates/meow-agent/src/budget.rs:38-45,
high). The design's promise that an unbounded run is possible by saying so has
no syntax behind it (reasoned from docs/design/0.3.0-starlark-api.md section
5.1 and crates/meow-star/src/agent.rs:41-60, low).

## What would reverse it

- A user needs a declared agent with no step, token or duration bound, and the
  declaration surface gains an explicit way to say so; the default for an
  omitted axis would then need restating against REQ-1012 (reasoned from
  crates/meow-star/src/agent.rs:41-60, low).

## Consequences

- The engine and the declaration surface read an unset axis differently:
  `Budget` in `meow-agent` treats `None` as unbounded, while `BudgetFields` in
  `meow-star` treats an omitted field as the default (from
  crates/meow-agent/src/budget.rs:11-26 and
  crates/meow-star/src/agent.rs:41-60, high).
- A markdown agent and a Starlark agent get the same result from one partial
  budget, because both go through `BudgetFields::build` (from
  crates/meow-star/tests/agents.rs `the_two_ways_of_declaring_an_agent_agree`,
  high).

## How I will know it was realised

1. `an_agent_accepts_every_declared_keyword` in
   crates/meow-star/tests/agents.rs passes: an agent declared with
   `budget = {"tokens": 50000, "steps": 12}` has a thirty-minute duration, the
   engine's default.
2. `an_unset_axis_is_unbounded_and_the_default_is_bounded` in
   crates/meow-agent/tests/spec.rs passes, showing the engine's own reading of
   an unset axis is unchanged.

## What this does not settle

- REQ-1012 says an axis left unset is unbounded. This decision contradicts it
  for a budget declared in Starlark or markdown, and nothing in the record says
  which one holds (from
  docs/requirements/REQ-1012-unset-budget-axis-is-unbounded.md and
  crates/meow-star/src/agent.rs:41-60, high).
- How a declaration asks for an unbounded axis (reasoned from
  crates/meow-star/src/agent.rs:41-60, low).
