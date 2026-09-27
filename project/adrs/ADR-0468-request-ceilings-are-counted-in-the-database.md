---
id: ADR-0468
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1074]
supersedes: []
---

# 0468. Request ceilings are counted in the workspace database, shared by every invocation

## Decision

Requests are counted in the workspace database, so two invocations share the
ceiling (from https://github.com/meowshed/meowg1k/issues/167, high). The issue is
open, so this records a proposal nobody has approved.

Nothing of this works yet: no declaration accepts a request ceiling and no
code counts requests, so `grep -rn requests_per crates` finds nothing (from
https://github.com/meowshed/meowg1k/issues/167, high). The database it would count
in exists: one SQLite file per workspace at `.meow/.data/meow.db`, with tables
for sessions, the key-value store, the cache, usage and the index and none for
request counts (from docs/spec/store.md section Boundary and
crates/meow-store/src/migrations.rs:25, high).

## Why

A limit that resets with the process isn't a limit: run `meow` in a loop and
every invocation starts with a full allowance (from
https://github.com/meowshed/meowg1k/issues/167, high). The Go implementation
replaced a per-process limiter with a database-backed one for the same reason,
and the rewrite dropped it (from https://github.com/meowshed/meowg1k/issues/167,
high) (from https://github.com/meowshed/meowg1k/pull/38, high). The store already
has one SQLite file per workspace, which `meow-store` owns (from
https://github.com/meowshed/meowg1k/issues/167, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| An in-memory limiter per process | No store write per request, and no state that outlives the run (reasoned from crates/meow-store/src/migrations.rs:89, low) | It resets with the process, so a loop of invocations each gets a full allowance (from https://github.com/meowshed/meowg1k/issues/167, high) |
| One global state database in an XDG directory, as the Go design proposed | It shares a limit across every project using the same API key (from https://github.com/meowshed/meowg1k/issues/34, high) | The rewrite keeps one database per workspace, because the blob table is shared and SQLite can't enforce a reference count across files (from docs/spec/store.md section Changes from v0.2.x, high); the issue names the workspace database and gives no reason of its own (from https://github.com/meowshed/meowg1k/issues/167, medium) |
| Do nothing: bound a run with the budget alone | The budget already bounds steps, tokens, wall time and cost, with no new axis (from docs/spec/agent.md [R-AGENT-010], high) | A budget bounds one run and resets when the process exits, and a rate limit is a property of the machine over time (from https://github.com/meowshed/meowg1k/issues/167, high) |

## What it costs

Two workspaces that use the same API key don't share a ceiling, so together
they can spend twice what either declares; the Go design's global database was
chosen to prevent exactly that (reasoned from
https://github.com/meowshed/meowg1k/issues/34, low). Every request would cost a
write to the workspace database before the call (reasoned from
docs/requirements/REQ-1075-over-ceiling-request-refused-before-call.md, low).

## What would reverse it

Users hitting a provider's per-key limit while running several workspaces in
parallel would show that the ceiling has to follow the key, not the workspace
(reasoned from https://github.com/meowshed/meowg1k/issues/34, low).

## Consequences

- The limiter lives in `meow-store`'s database, so a sub-agent and a parallel
  fan-out in the same workspace count against one ceiling as well (reasoned from
  docs/spec/store.md section Scope, low).
- Deleting `.meow/.data/` resets the count, as it resets the cache (reasoned
  from docs/spec/store.md section Boundary, low).
- The vision's sixth goal stays "partly met" until this lands (from
  docs/vision.md goal 6, high).

## How I will know it was realised

A test runs two processes and shows the second refused by what the first spent,
because a single-process test can't tell a shared limiter from a per-process one
(from https://github.com/meowshed/meowg1k/issues/167, high). No such test exists
yet (from `grep -rn requests_per crates`, high).

## What this does not settle

- Whether the ceiling becomes a fifth budget axis or lives beside the budget is
  left to whoever builds it, against the decision notes of `docs/spec/agent.md`
  (from https://github.com/meowshed/meowg1k/issues/167, high).
- What a request over the ceiling does: issue #34 had it wait for the window,
  and REQ-1075, not this decision, requires it to be refused before the call
  (from https://github.com/meowshed/meowg1k/issues/34 and
  docs/requirements/REQ-1075-over-ceiling-request-refused-before-call.md,
  high).
- Whether tokens per minute are limited too: issue #34 listed them, and issue
  #167 names requests only (from https://github.com/meowshed/meowg1k/issues/34
  and https://github.com/meowshed/meowg1k/issues/167, high).
