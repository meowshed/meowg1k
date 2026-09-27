---
id: ADR-0423
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1051, REQ-2489]
supersedes: []
---

# 0423. The sub-agent nesting depth is 3 and isn't declarable

## Decision

`max_depth` is 3 and not declarable (from
https://github.com/meowshed/meowg1k/pull/123, high).

Once this is accepted, `meow-star` gives every agent spec it builds
`max_depth: 3`, and while it builds a tool set it leaves out a sub-agent that
would sit at depth 3 or deeper, so two agents that name each other produce a
finite spec (from crates/meow-star/src/run.rs:33,
crates/meow-star/src/run.rs:361 and crates/meow-star/src/run.rs:408-414,
high). A workspace can't raise or lower the limit, because `meow.agent`
accepts no keyword for it (from crates/meow-star/src/declare.rs:267-279,
high).

## Why

`R-STAR-040` doesn't list it among `meow.agent`'s keywords, and spec
construction has to terminate on two agents that name each other (from
https://github.com/meowshed/meowg1k/pull/123, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A declarable `max_depth` keyword | A workspace that needs a deeper chain, or wants a shallower one, could say so (reasoned from crates/meow-agent/src/spec.rs:61, where the engine already takes the limit as a field, low) | `R-STAR-040` doesn't list it among `meow.agent`'s keywords (from https://github.com/meowshed/meowg1k/pull/123, high) |
| No limit at construction, leaving the engine's runtime check to refuse a sub-agent that is too deep | The model would be told about the refusal as a tool error, as REQ-1052 describes (from crates/meow-agent/src/nested.rs:78-86, high) | `tools_for` builds each sub-agent's spec eagerly, so construction has to terminate on two agents that name each other before any run starts (from https://github.com/meowshed/meowg1k/pull/123 and crates/meow-star/src/run.rs:372-377, high) |

Doing nothing isn't an option here, because before this change `meow-star`
built no agent specs at all (from https://github.com/meowshed/meowg1k/pull/123,
high).

## What it costs

- A workspace that needs a chain deeper than three agents has no way to ask
  for one short of a change to REQ-2489 (reasoned from
  crates/meow-star/src/run.rs:33, low).
- An agent at depth 2 silently loses every sub-agent in its `tools` list: the
  model is never offered it, so it never gets the tool error REQ-1052 asks for
  (from crates/meow-star/src/run.rs:408-411, medium).
- `meow-star`'s limit of 3 differs from the default of 4 that `AgentSpec::new`
  in `meow-agent` gives a spec built in Rust (from
  crates/meow-agent/src/spec.rs:113 and crates/meow-star/src/run.rs:33, high).

## What would reverse it

- REQ-2489 gaining a `max_depth` keyword for `meow.agent` (from
  https://github.com/meowshed/meowg1k/pull/123, high).
- Sub-agent specs built lazily when a sub-agent is first called, which removes
  the need for construction to stop at a fixed depth (reasoned from
  crates/meow-star/src/run.rs:373-414, low).

## Consequences

- Agents at depths 0, 1 and 2 run; a sub-agent at depth 3 is left out of its
  caller's tool set rather than refused at call time (from
  crates/meow-star/src/run.rs:408-414, high).
- The engine's own check in `SubAgent::call` still guards a spec built in Rust
  with a different limit (from crates/meow-agent/src/nested.rs:78-86, high).

## How I will know it was realised

1. `nesting_past_the_limit_is_reported_rather_than_fatal` in
   crates/meow-agent/tests/spec.rs shows the engine refusing a sub-agent past
   its `max_depth` (from crates/meow-agent/tests/spec.rs:1031-1047, high).
2. No test in crates/meow-star/tests covers the fixed limit or two agents
   that name each other; a test that declares such a pair and runs one of them
   to completion would close that (from a search of crates/meow-star/tests for
   "depth", low).

## What this does not settle

- Whether a handler that calls `agent.run` counts as a level: it passes its
  own depth unchanged, so an agent whose tool runs the same agent again is
  bounded by the shared budget and not by the depth limit (from
  crates/meow-star/src/value.rs:213 and crates/meow-star/src/run.rs:912-917,
  medium).
- The comment on `MAX_DEPTH` says the limit holds "unless a declaration says
  otherwise", which no declaration can do (from crates/meow-star/src/run.rs:32,
  high).
