---
id: ADR-0112
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2252, REQ-2253]
supersedes: []
---

# 0112. Garbage collection deletes whole sessions and never part of one

## Decision

`meow session gc` applies a configured retention policy and deletes whole
sessions, never parts of one (from docs/design/0.3.0-sessions.md section 8,
high). A session goes together with every session below it, leaves first, so a
parent never goes without its descendants (from
crates/meow-session/src/retention.rs:101-145, high).

## Why

An agent log grows quickly, and a design without a garbage collection story ends
with a gigabyte of SQLite in a repository nobody wants to clone (from
docs/design/0.3.0-sessions.md section 8, high). A half-truncated log is worse
than no log (from docs/design/0.3.0-sessions.md section 8, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no garbage collection | Nothing a person ran is ever lost, and every run stays available to replay and fork (reasoned from docs/design/0.3.0-sessions.md section 1, low) | The database grows to a gigabyte of SQLite in a repository nobody wants to clone (from docs/design/0.3.0-sessions.md section 8, high) |
| Truncate part of a session | A long session can be trimmed to fit a size limit while its recent events stay (reasoned from docs/design/0.3.0-sessions.md section 8, low) | A half-truncated log is worse than no log (from docs/design/0.3.0-sessions.md section 8, high) |
| Delete a session alone and leave its children | A sweep removes exactly the sessions a limit chose (reasoned from crates/meow-session/src/retention.rs, low) | The foreign key refuses to orphan a child, and a parent's totals include its children's (from crates/meow-session/src/retention.rs:138-139 and docs/design/0.3.0-sessions.md section 6.1, high) |

## What it costs

One large session can't be trimmed, only deleted whole. Under a size limit the
sweep deletes the oldest sessions one tree at a time and measures the file
again after each, which is a loop with a database query in it (from
https://github.com/retran/meowg1k/pull/128, high). A named session anywhere in a
tree protects the whole tree, so one name can keep many unnamed sessions alive
(from crates/meow-session/src/retention.rs:121-135, high).

## What would reverse it

- A requirement to bound the size of one session, which whole-session deletion
  can't meet (reasoned from crates/meow-session/src/retention.rs, low).

## Consequences

- Blobs are reference-counted and collected when their last referent goes (from
  docs/design/0.3.0-sessions.md section 8, high).
- A named session is exempt by default, because naming one is the user saying it
  matters (from docs/design/0.3.0-sessions.md section 8, high).
- A fork is a sibling of its origin and not a descendant, so collecting the
  origin doesn't take the fork with it (from
  https://github.com/retran/meowg1k/pull/128, high).

## How I will know it was realised

1. `a_sweep_takes_a_parent_together_with_its_children` in
   crates/meow-session/tests/fork.rs passes: a sweep that chooses the parent
   deletes the parent and the child together (from
   crates/meow-session/tests/fork.rs:193, high).
2. `forking_retains_the_blobs_the_copy_points_at` in the same file passes, so
   deleting an origin leaves the blobs a fork reads (from
   crates/meow-session/tests/fork.rs:122, high).

## What this does not settle

- Which limits apply and how they combine: the strictest of age, count and size
  wins, which is REQ-2255 and REQ-2256 (from
  crates/meow-session/tests/fork.rs:254, high).
- The workspace store, which garbage collection can't reach because it isn't a
  session row (from docs/design/0.3.0-starlark-api.md section 10, high).
