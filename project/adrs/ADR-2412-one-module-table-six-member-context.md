---
id: ADR-2412
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2414, REQ-2415, REQ-2416, REQ-2445]
supersedes: []
---

# 2412. One module table and a six-member context replace the hand-built contexts

## Decision

Runtime modules are registered in one table that every consumer takes them from,
and the handler context has six members plus `cancelled()` (from
docs/spec/starlark.md [R-STAR-010] [R-STAR-011] [R-STAR-020], high).

## Why

In v0.2.x the handler context carried 26 members and was assembled by hand in
two places, `ctx_run.go` and `module_llm.go`. The two had already diverged on UI
nesting depth, and a module added to one was silently missing from tools running
inside an agent loop (from docs/spec/starlark.md, Changes from v0.2.x, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Do nothing: keep the v0.2.x context, 26 members assembled by hand in `ctx_run.go` and `module_llm.go` | Every capability is one attribute away with no `load` line, because the context is how the runtime hands out capabilities (from docs/design/0.3.0-starlark-api.md, section 1, high). | The two places diverged on UI nesting depth, and a module added to one was missing from tools inside an agent loop (from docs/spec/starlark.md, Changes from v0.2.x, high). |
| One builder, with the capabilities still members of the context | It removes the divergence and keeps capabilities one attribute away (reasoned from docs/design/0.3.0-starlark-api.md, section 1, low). | A capability on the context is invisible to `meow policy explain`, because nothing in the file says the handler will use it (from crates/meow-star/src/context.rs:6-10, high), and the context should carry only what is per invocation (from docs/design/0.3.0-starlark-api.md, section 1, high). |

Neither the code, the history nor the design documents name a further option.

## What it costs

- A handler writes a `load("@std//...")` line for each capability it uses (from
  docs/design/0.3.0-starlark-api.md, section 11, high).
- Every v0.2.x script that used `ctx.fs`, `ctx.git` or another of the 24 removed
  members breaks, deliberately (from docs/design/0.3.0-starlark-api.md, section
  11, high).

## What would reverse it

If a consumer needs a different module set depending on where a handler runs,
such as a tool inside an agent loop that must see fewer modules than the
command line, one table can't express it and REQ-2416 would have to change
first (reasoned from crates/meow-star/src/modules.rs:4-12, low).

## Consequences

- `crates/meow-star/src/modules.rs` is the one table and
  `crates/meow-star/src/context.rs` builds the one context (from CLAUDE.md,
  one_context_builder, high).
- REQ-2416 holds without a mechanism of its own, because there is nowhere for a
  second copy to live (from crates/meow-star/src/modules.rs:10-12, high).
- Every module resolves in both phases, and each builtin checks the phase, so a
  declaration file may `load` a module and fails only when it calls one (from
  https://github.com/retran/meowg1k/pull/123, high).

## How I will know it was realised

These tests pass (high):

- `one_table_separates_an_unknown_module_from_an_unavailable_call` in
  crates/meow-star/tests/loading.rs.
- `a_tool_in_an_agent_loop_sees_the_same_modules` in
  crates/meow-star/tests/running.rs.
- `the_context_has_six_members_and_cancelled` in
  crates/meow-star/tests/running.rs.

That no code path builds a second module set is a review judgement, the judge
REQ-2415 names (from
docs/requirements/REQ-2415-consumers-obtain-modules-from-table.md, high).

## What this does not settle

- Which modules the table holds, which docs/design/0.3.0-starlark-api.md section
  10 lists (from crates/meow-star/src/modules.rs:33-38, high).
- How a fetched package's `@<pkg>//` modules reach a handler (from
  docs/design/0.3.0-starlark-api.md, section 10, medium).
