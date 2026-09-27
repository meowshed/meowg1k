---
id: ADR-0102
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1055, REQ-1056, REQ-1057, REQ-2493, REQ-2494, REQ-2495]
supersedes: []
---

# 0102. Concurrency runs over agent invocations and never over Starlark closures

## Decision

`agent.call(...)` builds a pending invocation without running it, and
`meow.parallel` runs a list of them concurrently in Rust and returns the results
in order (from docs/design/0.3.0-starlark-api.md section 5.4, high). A failed
element is a result whose `.stop` isn't `finished`, so one failure doesn't
poison the batch (from docs/design/0.3.0-starlark-api.md section 5.4, high).

What works today: the engine's `run_parallel` runs invocations concurrently on
Tokio tasks and returns them in order (from
crates/meow-agent/src/nested.rs:143-185, high), and `meow.parallel` takes only
invocations and keeps their order. What doesn't: `meow.parallel` still runs its invocations one after another, because
the engine's fan-out takes specs and the Starlark side holds names (from
crates/meow-star/src/value.rs:320-333 and
https://github.com/retran/meowg1k/pull/123, high).

## Why

A Starlark value can't leave the heap it lives in. `Value` implements `Send` and
`Sync`, but it borrows its heap, `Module` isn't `Send`, and a heap can't outlive
the closure `Module::with_temp_heap` scopes it to (from
docs/design/0.3.0-spike-starlark.md The correction, high). An agent's execution
lives in Rust and only its declaration lives in Starlark, so a declaration
crosses the boundary where a closure can't (from
docs/design/0.3.0-architecture.md section 3.1, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no concurrency primitive, as in v0.2.1 | Nothing new to learn or maintain; v0.2.1's planning library left parallel execution to "your orchestration" (from `git show v0.2.1:.meowg1k/lib/planning.star`, line 605, high) | Fanning one agent out over many inputs is the case that matters, and it would run one input at a time (from docs/design/0.3.0-starlark-api.md section 5.4, high) |
| A script spawns two closures and joins them | Runs any Starlark code concurrently, not only agent runs (reasoned from docs/design/0.3.0-architecture.md section 3.1, low) | Moving a Starlark value into `std::thread::spawn` fails to compile with `error[E0521]: borrowed data escapes outside of closure` (from docs/design/0.3.0-spike-starlark.md The correction, high) |
| `meow.parallel` accepts Starlark functions | Reads naturally in a script, which is why the error message has to explain the refusal (from crates/meow-star/src/value.rs:282-285, high) | A function lives on its evaluator's heap and the runs happen on other threads (from crates/meow-star/src/value.rs:282-285, high) |

## What it costs

A script can't run arbitrary code concurrently (from
docs/design/0.3.0-architecture.md section 3.1, high). The source calls the
primitive deliberately narrow, covering the case that matters: fanning one agent
out over many inputs (from docs/design/0.3.0-starlark-api.md section 5.4, high).
Post-processing each result happens in the script after the batch returns,
one result at a time (reasoned from crates/meow-star/src/value.rs:289-336, low).

## What would reverse it

- `starlark-rust` lets a frozen value or a heap move between threads, which
  removes the lifetime reason the spike found (reasoned from
  docs/design/0.3.0-spike-starlark.md The correction, low).
- Workspaces show a recurring need to run something other than an agent
  concurrently, such as several shell commands, which the narrow primitive
  can't express (reasoned from docs/design/0.3.0-starlark-api.md section 5.4,
  low).

## Consequences

The architecture document's claim that a Starlark value can't cross a thread
because it isn't `Send` was wrong about why; the conclusion holds for the
lifetime reason above (from docs/design/0.3.0-spike-starlark.md The correction,
high). The fan-out shares the caller's ledger, so it stops starting new work
once the budget is spent (from crates/meow-agent/src/nested.rs:145-150, high).
Wiring `meow.parallel` to the engine's fan-out needs the spec cache M11 brings
(from https://github.com/retran/meowg1k/pull/123, high).

## How I will know it was realised

1. `parallel_takes_invocations_and_keeps_their_order` and
   `parallel_refuses_a_starlark_function` in crates/meow-star/tests/running.rs
   pass (from crates/meow-star/tests/running.rs:630-687, high).
2. `a_fan_out_returns_in_order_and_survives_one_failure` in
   crates/meow-agent/tests/spec.rs passes (from
   crates/meow-agent/tests/spec.rs:1095-1098, high).
3. `meow.parallel` calls `run_parallel` in place of its sequential loop, and a
   test shows two invocations overlapping in time; no such test exists yet
   (from crates/meow-star/src/value.rs:320-333, high).

## What this does not settle

- How many invocations run at once: `run_parallel` spawns one task per
  invocation with no cap (from crates/meow-agent/src/nested.rs:158-181, high).
- Whether a tool handler written in Starlark can run inside a parallel branch,
  given that each would need its own blocking thread and evaluator (reasoned
  from crates/meow-star/src/run.rs:914-917, low).
