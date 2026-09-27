---
id: ADR-0446
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2478, REQ-2479, REQ-2498, REQ-2499]
supersedes: []
---

# 0446. A handler reaches a markdown agent through `meow.agent_named`

## Decision

`meow.agent_named("reviewer")` gives a markdown agent's value by name, and the
name is checked at load time after every markdown agent has been read (from
https://github.com/meowshed/meowg1k/pull/137, high).

What works now: `meow.agent_named` records the reference during declaration and
returns the same agent value `meow.agent` returns; the loader evaluates
`meow.star`, then reads `.meow/agents/`, then resolves every recorded reference
and fails with "agent `<name>` is not declared" and a closest-name suggestion
(from crates/meow-star/src/declare.rs:309, crates/meow-star/src/loader.rs:410
and crates/meow-star/src/registry.rs:426, high). This repository's own
`.meow/meow.star` reaches its three agents this way (from .meow/meow.star:125,
high). What still doesn't: the error drops the origin of the reference, so it
doesn't say which file and line named the missing agent (from
crates/meow-star/src/registry.rs:428, high).

## Why

A markdown agent is read after the file that would hold it, so
`reviewer.run(...)` had nothing to bind to (from
https://github.com/meowshed/meowg1k/pull/137, high). Checking at load time makes
naming one that doesn't exist fail before anything runs, as `R-STAR-032` asks
(from https://github.com/meowshed/meowg1k/pull/137, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: hold the markdown agent's value in a variable | No new builtin; a handler uses the agent like any other bound name (reasoned from crates/meow-star/src/declare.rs:299, low) | The agent is read after the file that would hold it (from https://github.com/meowshed/meowg1k/pull/137, high) |
| Check the name when the handler runs | Nothing to record during declaration and nothing to resolve afterwards (reasoned from crates/meow-star/src/registry.rs:280, low) | A misspelt name would fail at first use, and `R-STAR-032` asks for load time (from https://github.com/meowshed/meowg1k/pull/137, high) |
| Read `.meow/agents/` before evaluating `meow.star` | A plain variable would then bind (reasoned from crates/meow-star/src/loader.rs:410, low) | The source doesn't weigh it; `meow.star` is the entry point and evaluates first by design, and REQ-2479 already makes declaration order irrelevant for references (reasoned from crates/meow-star/src/loader.rs:410, low) |

## What it costs

One more declaration builtin to document and test, and a second way to get an
agent's value beside `meow.agent`, which a reader has to learn is only for
agents declared elsewhere (reasoned from crates/meow-star/src/declare.rs:302,
low).

## What would reverse it

Markdown agents becoming bindable as Starlark names before `meow.star` runs,
for example by being read first, would leave `meow.agent_named` with no case
`meow.agent` or a variable doesn't cover (reasoned from
crates/meow-star/src/loader.rs:410, low).

## Consequences

- A handler can call `reviewer.run(...)` on a markdown agent, and to the caller
  that value is the same as a Starlark agent's (from
  https://github.com/meowshed/meowg1k/pull/137, "Requirement IDs", high).
- `meow check` on a workspace naming a missing agent exits before any command
  runs (from crates/meow-star/tests/agents.rs:398, high).

## How I will know it was realised

`crates/meow-star/tests/agents.rs` `a_handler_can_name_a_markdown_agent` and
`naming_an_agent_that_does_not_exist_fails_at_load_time` pass; the second
expects "agent `reviewr` is not declared" and "Did you mean `reviewer`?" (from
crates/meow-star/tests/agents.rs:373 and crates/meow-star/tests/agents.rs:398,
high).

## What this does not settle

- Whether the unknown-agent error should name the file and line of the
  reference; the origin is recorded and discarded (from
  crates/meow-star/src/registry.rs:428, high).
