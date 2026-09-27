---
id: ADR-0432
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2239, REQ-2240, REQ-2241, REQ-2253]
supersedes: []
---

# 0432. A fork is a sibling of its origin, not a descendant

## Decision

A fork is a sibling, not a descendant (from
https://github.com/retran/meowg1k/pull/128, high).

Once this is accepted, a fork is a session with no parent that records its
origin session and sequence in a column of its own, and its copied events hold
a reference to each blob they point at (from crates/meow-session/src/fork.rs:28
and 82, and crates/meow-store/src/migrations.rs:123, high). No test yet
collects an origin and checks that its fork survives (from a search of
crates/meow-session/tests/fork.rs, high).

## Why

Making a fork a child of its origin would put it under the deletion rule of
`R-SESSION-080`, so collecting the origin would take the fork with it, the
opposite of what the reference counting is for (from
https://github.com/retran/meowg1k/pull/128, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no fork, as in v0.2.x | Nothing to reference-count and no second kind of link between sessions (reasoned from docs/design/0.3.0-sessions.md section 6, low) | A failed run is a dead end, and investigating step 9 of 40 costs a full rerun where a fork costs one step (from docs/design/0.3.0-sessions.md sections 2 and 6, high) |
| Make a fork a child of its origin | The parent link already exists, so the fork would show in the origin's tree with no new column (reasoned from crates/meow-store/src/migrations.rs:44, low) | Collecting the origin would delete the fork (from https://github.com/retran/meowg1k/pull/128, high) |

## What it costs

The origin link is `ON DELETE SET NULL`, so once the origin is collected the
fork no longer says where it came from (from
crates/meow-store/src/migrations.rs:123, high). A fork of a sub-agent's session
has no parent, so its spend appears in no parent's totals (reasoned from
crates/meow-session/src/fork.rs:28 and REQ-2219, low).

## What would reverse it

- Retention has to delete a fork together with its origin, for example to
  bound the forks one run may have. That needs the fork under the origin's
  deletion rule, which parentage gives and the origin link doesn't (reasoned
  from crates/meow-session/src/retention.rs:101, low).

## Consequences

- The store carries an `origin_id` column beside `parent_id` (from
  crates/meow-store/src/migrations.rs:44 and 123, high).
- `Sessions::origin` answers where a fork came from, and a session that isn't
  a fork has none (from crates/meow-session/tests/fork.rs
  `a_fork_records_its_origin`, high).
- A sweep treats a fork as a tree root of its own, deleted by its own age and
  rank rather than its origin's (from crates/meow-session/src/retention.rs:101,
  high).

## How I will know it was realised

1. crates/meow-session/tests/fork.rs `a_fork_records_its_origin` and
   `forking_retains_the_blobs_the_copy_points_at` pass (from those tests,
   high).
2. A sweep that deletes the origin leaves the fork and its payloads readable;
   no test covers it today (from a search of crates/meow-session/tests, high).

## What this does not settle

- Whether `meow session list` shows a fork beside its origin; parentage drives
  the tree view and a fork has none (from docs/design/0.3.0-sessions.md section
  6.1, medium).
- What a fork of a fork records: its immediate origin only, or the chain
  (reasoned from crates/meow-store/src/migrations.rs:123, low).
