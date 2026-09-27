---
id: ADR-0602
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2531, REQ-2532]
supersedes: []
---

# 0602. A handler observes a run through typed callbacks, one per kind

## Decision

`agent.run` takes `on = meow.on(text = ..., tool = ..., step = ...)`, where each
callback is optional and receives one kind of event, and a run given no `on`
is drawn by the built-in renderer (from docs/design/0.3.0-starlark-api.md
section 5.5, high).

Once this is accepted, only the second half works: `agent.run` takes the task
alone and always draws through the built-in renderer, and `meow.on` doesn't
exist (from crates/meow-star/src/value.rs:205 and
crates/meow-star/src/declare.rs, high).

## Why

v0.2.x gave a script one handler that received a dictionary with a string
`kind` field to switch on, and `make_agentic_stream_handler` existed to work
around it; typed callbacks remove the switch, and omitting `on` gives what
nearly every command wants (from docs/design/0.3.0-starlark-api.md section
5.5, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: `agent.run` takes no observer and the built-in renderer always draws | No Starlark code runs while the engine holds the thread, so nothing crosses the blocking bridge (reasoned from ADR-0100 and crates/meow-star/src/value.rs:205, low) | A command can't present an agent's progress its own way, which `ui_helpers.star` did in v0.2.x (from docs/design/0.3.0-starlark-api.md section 11, high) |
| One handler receiving a dictionary with a `kind` field, as v0.2.x did | One callable covers every event kind, including kinds added later (reasoned from docs/design/0.3.0-starlark-api.md section 5.5, low) | The script has to switch on a string, and the boilerplate to do it became a library every user copied (from docs/design/0.3.0-starlark-api.md sections 1 and 5.5, high) |

## What it costs

A callback is a Starlark value and can't leave the handler's thread, while the
engine runs on the async runtime and the handler thread blocks on it, so each
event has to be relayed back to the blocked thread before a callback can run
(reasoned from ADR-0100 and REQ-2521, low). A sink backed by a script callback
takes the completed text once per step in place of every delta, because a
call per token is too dear (from
docs/requirements/REQ-1062-declined-deltas-delivered-per-step.md, high).

## What would reverse it

- The relay back to the handler thread proves unable to run a callback without
  stalling the engine, and `ctx.out` plus the built-in renderer covers the
  commands in `.meow/` (reasoned from ADR-0100, low).

## Consequences

- `meow.on` is a new builtin on the `meow` global, allowed only while a handler
  runs, as REQ-2525 asks of every module but `load` and `env` (reasoned from
  REQ-2525, low).
- The `.meow/` commands can draw their own progress where the default renderer
  doesn't suit them (reasoned from .meow/meow.star, low).

## How I will know it was realised

1. A test in crates/meow-star/tests/running.rs runs an agent with
   `on = meow.on(tool = ...)` and sees the callback called once per tool call,
   and a second run with no `on` produces the built-in renderer's events.

## What this does not settle

- What each callback receives: the design shows `delta`, `call` and `step`
  with fields `name`, `args` and `index` and doesn't define them further (from
  docs/design/0.3.0-starlark-api.md section 5.5, high).
- Whether a callback that fails stops the run or is reported and dropped, as
  REQ-1063 does for a sink that fails (reasoned from REQ-1063, low).
