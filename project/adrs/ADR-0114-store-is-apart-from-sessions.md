---
id: ADR-0114
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2463, REQ-2464, REQ-2465, REQ-2632, REQ-2633]
supersedes: []
---

# 0114. The workspace store and session state are separate scopes, and neither is session metadata

## Decision

Session state (`ctx.session.set` and `get`) is scoped to one run and recorded in
the log as it changes, and the workspace store (`@std//store`) is durable across
runs, in its own table, with explicit keys (from docs/design/0.3.0-sessions.md
section 7, high). `meow session gc` can't take the store with it, because the
store isn't reachable from a session row (from docs/design/0.3.0-starlark-api.md
section 10, high). The store is the `kv` table, which has no column naming a
session (from crates/meow-store/src/migrations.rs:58-63, high).

## Why

In v0.2.x session metadata was the general-purpose store:
`.meowg1k/lib/memory.star` kept JSON blobs under `context_*`, untyped, unbounded
and mixed with runtime bookkeeping in one flat namespace (from
docs/design/0.3.0-sessions.md section 2, high). Separating the two stops one
namespace from holding both `llm_duration_ms_3_1762...` and a user's cached
analysis (from docs/design/0.3.0-sessions.md section 7, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: session metadata as the general-purpose store, as in v0.2.x | One place for every value a run keeps, and no second table or module (reasoned from docs/design/0.3.0-sessions.md section 2, low) | Untyped, unbounded and mixed with runtime bookkeeping in one flat namespace (from docs/design/0.3.0-sessions.md section 2, high) |
| State kept by a userland library such as `memory.star` | The runtime stays smaller, and a workspace shapes its own memory (reasoned from docs/design/0.3.0-starlark-api.md section 8, low) | `memory.star`, `planning.star` and `compaction.star` total 2,500 lines that exist because neither scope had runtime support (from docs/design/0.3.0-starlark-api.md section 8, high) |
| One store keyed by session, where a run's values die with its session | Garbage collection would clean up stored values with the runs that wrote them (reasoned from crates/meow-store/src/migrations.rs:58-63, low) | A value a user means to keep, such as a cached analysis, would vanish when `meow session gc` collects the run that wrote it (from docs/design/0.3.0-sessions.md section 7 and crates/meow-cli/src/keep.rs:10-14, high) |

## What it costs

A workspace has two places to keep a value, and a handler author has to choose
the scope. The store's values never expire: `meow session gc` can't reach them,
so a value stays until a handler deletes it (from
crates/meow-cli/tests/surface.rs:736, high).

## What would reverse it

- A requirement that stored values expire with the runs that wrote them, which
  would contradict REQ-2465 (reasoned from
  docs/requirements/REQ-2465-store-value-survives-session-collection.md, low).

## Consequences

Neither scope holds large blobs: a tool result belongs in the log and an
artifact in a file (from docs/design/0.3.0-sessions.md section 7, high). The
store has its own module with `get`, `put`, `delete` and `keys`, and each call
is refused during declaration (from crates/meow-star/src/modules.rs:1166-1200,
high).

## How I will know it was realised

1. `the_store_outlives_the_run_that_wrote_it` in
   crates/meow-cli/tests/surface.rs passes: three processes count to three
   through the store (from crates/meow-cli/tests/surface.rs:702, high).
2. `session_gc_does_not_take_the_store_with_it` in the same file passes: a
   value survives `meow session gc --keep 0 --named` (from
   crates/meow-cli/tests/surface.rs:736, high).

## What this does not settle

- Whether session state is replayed on resume. `ctx.session.set` writes a
  `Note` with level `state`, and `get` reads from memory within the run, so a
  resumed session doesn't see what the original left behind; making it
  replayable needs an eleventh event kind (from
  https://github.com/meowshed/meowg1k/pull/129 and
  crates/meow-cli/src/session.rs:122-136, high).
- A size limit on the store, which no requirement sets (reasoned from
  docs/requirements/REQ-2632-durable-workspace-key-value-table.md, low).
