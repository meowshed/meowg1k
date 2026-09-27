---
id: ADR-0419
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2490, REQ-2491]
supersedes: []
---

# 0419. `meow.agent` and `meow.tool` return values rather than register names

## Decision

`meow.agent` and `meow.tool` return values rather than registering a name to
quote later (from https://github.com/retran/meowg1k/pull/123, high). Each value
holds only its name, and everything else is looked up through the evaluator's
run state when the value is used (from crates/meow-star/src/value.rs:13-15,
high).

Once this is accepted, an agent value goes into another agent's `tools` list
and is offered to the model as a tool, and a handler calls `helper.run(...)` on
the value it holds (from crates/meow-star/tests/running.rs
`an_agent_is_usable_where_a_tool_is` and `a_run_returns_text_stop_and_usage`,
high). Names still work where a value can't be held: a `tools` list and
`meow.command` accept a name as well as a value, and a markdown agent can only
write names (from crates/meow-star/src/value.rs:115-123 and
crates/meow-star/src/declare.rs:106-110, high).

## Why

A value is what lets an agent go into another agent's `tools` list and present
the same schema there (from https://github.com/retran/meowg1k/pull/123, high).
It is also what the migration from v0.2.x means by replacing
`ctx.run("name", k = v)` with `tool.run(k = v)` (from
crates/meow-star/src/value.rs:9-11 and docs/design/0.3.0-starlark-api.md
section 11, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: register a name to quote later, as v0.2.x's `ctx.run("name", ...)` did | Declaration order doesn't matter for a quoted name, because every name is resolved after the last file (from crates/meow-star/src/registry.rs:349-356, high) | An agent couldn't go into another agent's `tools` list as a value presenting the same schema (from https://github.com/retran/meowg1k/pull/123, high) |
| A value holding the whole declaration, handler function included | A use needs no lookup in the run state (reasoned from crates/meow-star/src/value.rs:13-15, low) | A function can't outlive the evaluator that made it, and the value would drag the runtime into every frozen module (from https://github.com/retran/meowg1k/pull/123 and crates/meow-star/src/value.rs:13-15, high) |

No third option appears in the code, the history or the design documents.

## What it costs

A value is bound by Starlark's own rules, so a declaration that uses a value has
to come after the line that makes it, where a quoted name could come anywhere
(reasoned from
docs/requirements/REQ-2479-references-resolved-after-all-declarations.md, low).
Pull request 123 rewrote three existing tests that had asserted the old
name-based behaviour (from https://github.com/retran/meowg1k/pull/123, high). A
handler can't hold the value of a markdown agent, which is read after
`meow.star`, so `meow.agent_named` exists to hand one out by name (from
crates/meow-star/src/declare.rs:303-317, high).

## What would reverse it

- An agent in a `tools` list no longer has to present the same schema as a
  tool, so REQ-2491 no longer needs a value (reasoned from
  docs/requirements/REQ-2491-agent-tool-schema-matches-tool.md, low).

## Consequences

`meow.command` takes the tool or agent rather than its name, so the thing it
exposes has to exist before the line exposing it (from
https://github.com/retran/meowg1k/pull/123, high).

- `name_of` turns an agent value, a tool value or a string into a name, and
  both `tools` and `meow.command` go through it (from
  crates/meow-star/src/value.rs:115-123 and
  crates/meow-star/src/declare.rs:320-330, high).
- A tool's handler is found by name in the frozen module when it runs, which
  ADR-0421 records (from https://github.com/retran/meowg1k/pull/123, high).

## How I will know it was realised

1. `an_agent_is_usable_where_a_tool_is` in crates/meow-star/tests/running.rs
   passes: the outer agent's model is offered `inner` as a tool.
2. `a_run_returns_text_stop_and_usage` in the same file passes, calling `run` on
   an agent value from a handler.

## What this does not settle

- Whether `meow.command` should keep accepting a bare name: `name_of` falls back
  to a string, while its error message says it takes a tool or an agent (from
  crates/meow-star/src/value.rs:115-123 and
  crates/meow-star/src/declare.rs:324-329, high).
