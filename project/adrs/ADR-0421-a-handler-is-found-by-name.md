---
id: ADR-0421
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2518, REQ-2519]
supersedes: []
---

# 0421. A tool handler is found by file and symbol, not held as a value

## Decision

The declaration records the file and the symbol, and the run phase looks the
handler up in the frozen module (from
https://github.com/retran/meowg1k/pull/123, high).

Once this is accepted, a tool bound to a module-level function runs on its own
blocking thread with a fresh evaluator, and a private name such as `_review`
works because the lookup ignores visibility (from
crates/meow-star/src/run.rs:455-461 and crates/meow-star/src/run.rs:912-917,
high). A lambda or a nested `def` still doesn't work as a handler, and says so
(from crates/meow-star/src/run.rs:57-69 and
crates/meow-star/src/run.rs:461-467, high).

## Why

A function can't outlive the evaluator that made it (from
https://github.com/retran/meowg1k/pull/123, high). The Starlark spike found the
reason: a value borrows the heap it lives in, and a heap can't outlive the
closure `Module::with_temp_heap` scopes it to, so moving one to another thread
fails to compile (from docs/design/0.3.0-spike-starlark.md, section "The
correction", high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Hold the handler function as a value | Needs no name recovered from a display string, so a lambda or a nested `def` would work too (reasoned from crates/meow-star/src/run.rs:34-41, low) | A function can't outlive the evaluator that made it (from https://github.com/retran/meowg1k/pull/123, high), and a value borrowed from a heap doesn't compile when it crosses a thread (from docs/design/0.3.0-spike-starlark.md, section "The correction", high) |
| Take the handler's name as a string in `run` | Names the symbol exactly, with no heuristic over a display form (reasoned from crates/meow-star/src/run.rs:34-41, low) | The API passes the function itself, `run = _handler`, and the workspace in this repository is written that way (from docs/design/0.3.0-starlark-api.md:333 and .meow/meow.star:82, high) |

Doing nothing isn't an option here, because before this change no handler ran
at all (from https://github.com/retran/meowg1k/pull/123, high).

## What it costs

The name comes from the function's display form, so a lambda or a nested `def`
fails with a message saying a handler has to be a top-level function; that is a
heuristic over a display string (from
https://github.com/retran/meowg1k/pull/123, high). A display form that parses
but names no module-level symbol fails only when the tool is first called,
not when it is declared, because `Handler::parse` checks the shape and the
lookup happens in `call_handler` (from crates/meow-star/src/declare.rs:238 and
crates/meow-star/src/run.rs:450-467, medium).

## What would reverse it

- A `starlark` release that lets a frozen function value be held and called
  from another thread's evaluator without its module, which removes the reason
  in "Why" (reasoned from docs/design/0.3.0-spike-starlark.md, section "The
  correction", low).
- A change to `starlark`'s display form for functions that breaks the
  `<file>.<name>` shape `Handler::parse` splits on (from
  crates/meow-star/src/run.rs:34-41 and crates/meow-star/src/run.rs:57-69,
  medium).

## Consequences

- Every tool call spawns a blocking thread and builds a fresh evaluator on a
  scoped heap, so nothing a handler allocated survives the call (from
  crates/meow-star/src/run.rs:912-917 and crates/meow-star/src/run.rs:478,
  high).
- A handler has to be a module-level function; a private helper name is
  accepted on purpose (from crates/meow-star/src/run.rs:455-461, high).

## How I will know it was realised

1. `a_tool_value_runs_by_itself` in crates/meow-star/tests/running.rs declares
   a tool with `run = double` and gets `write: 42` back when a handler calls
   it (from crates/meow-star/tests/running.rs:719-750, high).
2. No test covers the refusal of a lambda or a nested `def`; a test that
   declares `run = lambda ctx: ""` and asserts the "top level of a file"
   message would close that (reasoned from a search of crates/meow-star/tests
   for "lambda" and "top-level", low).

## What this does not settle

- Whether a handler whose symbol is missing should be refused at declaration
  rather than at first call (reasoned from crates/meow-star/src/declare.rs:238
  and crates/meow-star/src/run.rs:450-467, low).
- How a handler reaches the engine from its thread; ADR-0100 records that
  handlers block on the engine (from
  docs/adrs/ADR-0100-handlers-block-on-the-engine.md, high).
