---
id: ADR-0100
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2518, REQ-2519, REQ-2520, REQ-2521]
supersedes: []
---

# 0100. A handler blocks its own script thread on the async engine

## Decision

Each script invocation owns one blocking thread and one `Evaluator`, and a
builtin that performs input or output is a synchronous wrapper that blocks on an
async engine call through a `tokio::runtime::Handle` (from
docs/design/0.3.0-architecture.md section 3.2, high). From the script's side
`agent.run(...)` blocks, and underneath it is an async state machine (from
docs/design/0.3.0-architecture.md section 3.2, high). The code does this today:
`Runtime::block_on` wraps `Handle::block_on` and is only called from a
`spawn_blocking` thread, and a tool handler the engine calls gets a
`spawn_blocking` thread and an evaluator of its own (from
crates/meow-star/src/run.rs:203-210 and 914-917, high).

## Why

`starlark-rust` has no async evaluation hook for builtins, and adding one means
forking it (from docs/design/0.3.0-architecture.md section 3.3, high).
`Module::with_temp_heap_async` exists and doesn't help, because a builtin is a
synchronous function and blocking inside one from an async context panics (from
docs/design/0.3.0-spike-starlark.md What it changed, high). The spike ran the
evaluator inside `tokio::task::spawn_blocking`, which isn't an async context, so
`block_on` there doesn't panic (from docs/design/0.3.0-spike-starlark.md What
was checked, high). Handing Starlark a future or a callback in place of a
blocked thread would mean a value crossing a thread boundary, which the type
system refuses (from crates/meow-star/src/run.rs:6-11, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Async Starlark builtins | No OS thread held per running script, because a builtin would yield to the reactor while the model call is in flight (reasoned from docs/design/0.3.0-architecture.md section 3.3, low) | `starlark-rust` has no async hook for builtins, and adding one means forking it (from docs/design/0.3.0-architecture.md section 3.3, high) |
| Evaluate under `Module::with_temp_heap_async` | Keeps evaluation on a reactor thread with no `spawn_blocking` hop, because the library already offers it (reasoned from docs/design/0.3.0-spike-starlark.md What it changed, low) | A builtin is synchronous, so blocking inside one from an async context panics (from docs/design/0.3.0-spike-starlark.md What it changed, high) |
| Hand Starlark a future or a callback to resolve | Lets a script start several calls and wait for them itself (reasoned from crates/meow-star/src/run.rs:6-11, low) | A future carrying a Starlark value would cross a thread boundary, which the type system refuses (from crates/meow-star/src/run.rs:6-11, high) |

Doing nothing wasn't an option here: v0.2.1 was a Go program and its code is
gone from this branch, so a Rust runtime has to bridge the synchronous
evaluator to the async engine somehow (from CLAUDE.md project section, high).

## What it costs

One OS thread per concurrent agent, which the source calls a rounding error next
to the latency of the model call it waits on (from
docs/design/0.3.0-architecture.md section 3.3, high). Every value a script
builds stays in its heap's arena until the scoping closure ends, so a long run
holds everything it made; the spike left the size of that for measurement (from
docs/design/0.3.0-spike-starlark.md What was not checked, high).

## What would reverse it

- `starlark-rust` ships an async evaluation hook for builtins, which removes the
  reason the source gives for rejecting async builtins (reasoned from
  docs/design/0.3.0-architecture.md section 3.3, low).
- A fan-out large enough that one blocking thread per concurrent agent costs
  more than the model latency it waits on, which is the comparison the source
  used to accept the cost (reasoned from docs/design/0.3.0-architecture.md
  section 3.3, low).

## Consequences

Starlark is never concurrent and Rust is (from docs/design/0.3.0-architecture.md
section 3.1, high). `Module::with_temp_heap` scopes the module to a closure in
`starlark` 0.14, so the evaluator lives inside that closure on the blocking
thread, and implementers should expect the closure in place of a constructor
(from docs/design/0.3.0-spike-starlark.md What it changed, high); the loader and
the run phase both use that shape (from crates/meow-star/src/loader.rs:333 and
crates/meow-star/src/run.rs:478, high). Every caller of a handler has to start
it on a `spawn_blocking` thread, which the binary and the tests both do (from
crates/meow-cli/src/wire.rs:1156-1157 and
crates/meow-star/tests/running.rs:358, high).

## How I will know it was realised

1. `a_blocking_builtin_returns_a_value_not_a_future` in
   crates/meow-star/tests/running.rs passes: a handler calls `helper.run("go")`
   and reads `.text` on the next line as a string (from
   crates/meow-star/tests/running.rs:689-715, high).
2. `two_loads_of_one_workspace_are_independent` in
   crates/meow-star/tests/loading.rs passes, showing each load owns its
   evaluators (from crates/meow-star/tests/loading.rs:372-375, high).
3. `a_cancelled_run_stops_and_keeps_what_it_had` in
   crates/meow-agent/tests/spec.rs passes, showing the engine future a builtin
   blocks on returns when its token is cancelled (from
   crates/meow-agent/tests/spec.rs:535, medium).

## What this does not settle

- Cancellation from the script's side: the engine checks its token (from
  crates/meow-agent/tests/spec.rs:535, high), but no test cancels a run while a
  Starlark builtin is blocked on it, which the spike left for M4 to prove (from
  docs/design/0.3.0-spike-starlark.md What was not checked, high).
- Memory under a long run: every value lives in the arena until the closure
  ends, which the spike left as a measurement for M4 (from
  docs/design/0.3.0-spike-starlark.md What was not checked, high).
- How many handlers may run at once: the binary builds its runtime with
  Tokio's default blocking pool and sets no limit of its own (from
  crates/meow-cli/src/wire.rs:1066-1068, high).
