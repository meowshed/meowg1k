---
id: ADR-2204
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2239, REQ-2240, REQ-2241, REQ-2242, REQ-2243]
supersedes: []
---

# 2204. A session can be forked at a sequence number

## Decision

Forking at sequence `n` creates a new session whose first `n` events are copies
referencing the same blobs, records the origin and leaves the origin unchanged
(from docs/spec/session.md [R-SESSION-052], high).

Once this holds, `meow session fork <id> --at <seq>` prints the new session's
identifier, and a sequence that doesn't exist or falls inside a summarised
range fails as a usage error naming the range that would work (from
crates/meow-cli/src/surface.rs:197-212 and crates/meow-cli/src/wire.rs:681-704,
high). The design's `--model` flag on fork isn't implemented, so a changed
model or prompt comes from the next run, not from the fork command (from
docs/design/0.3.0-sessions.md section 6 and
crates/meow-cli/src/surface.rs:197-212, high).

## Why

v0.2.x had no fork, so investigating a run that went wrong at step 9 of 40 cost
a full rerun (from docs/spec/session.md, high). Forking is cheap because the
log is append-only: a fork copies the prefix and every blob it points at gains
a referent (from https://github.com/meowshed/meowg1k/pull/128, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| No fork, as in v0.2.x (the do-nothing option) | Nothing to build, and no second session sharing blobs with the first (reasoned from crates/meow-store/src/session.rs:144-182, low) | Investigating a run that went wrong at step 9 of 40 costs a full rerun (from docs/spec/session.md, high) |
| Copy the prefix by duplicating every payload | A fork is independent of its origin's blobs, so no reference count is needed (reasoned from crates/meow-store/src/session.rs:174-179, low) | Blobs are shared, not duplicated; a naive log is dominated by duplicated tool output (from docs/design/0.3.0-sessions.md sections 3 and 6, high) |
| Share the blobs without incrementing their reference counts, as the specification first did | One write fewer per blob on fork (reasoned from crates/meow-store/src/session.rs:174-179, low) | Collecting the origin would delete data the fork still pointed at (from commit ed9810f, high) |

## What it costs

A fork duplicates the event, usage and compaction rows of the prefix and adds
one reference-count update per blob, all in one transaction (from
crates/meow-store/src/session.rs:144-182, high). A blob stays in the database
while any fork still refers to it, so collecting an origin frees less (from
docs/spec/session.md [R-SESSION-052], medium).

## What would reverse it

- Session logs showing forks almost never used, so that the copied rows and
  reference counts cost more than the reruns they save (reasoned from
  docs/design/0.3.0-sessions.md section 6, low).

## Consequences

- Forking increments the reference count of every blob the copied events
  reference, because without the increment, collecting the origin would delete
  content the fork still points at (from docs/spec/session.md [R-SESSION-052],
  high).
- A fork is a sibling of its origin, not a child, so collecting the origin
  doesn't take the fork with it (from crates/meow-session/src/fork.rs:28-32,
  high; see ADR-0432).
- A fork can't start inside a summarised range, because it would begin from a
  conversation the origin's model rebuild never showed (from
  crates/meow-session/src/fork.rs:61-75, high).

## How I will know it was realised

1. `a_fork_copies_the_prefix_and_does_not_touch_the_origin`,
   `a_fork_records_its_origin` and
   `forking_retains_the_blobs_the_copy_points_at` in
   crates/meow-session/tests/fork.rs pass (from
   crates/meow-session/tests/fork.rs:83-149, high).
2. `forking_outside_the_range_names_the_range` and
   `forking_inside_a_summarised_range_fails` in the same file pass (from
   crates/meow-session/tests/fork.rs:151-188, high).

## What this does not settle

- Whether a fork is a child of its origin for retention, which ADR-0432
  settles (from `meow-method find fork`, high).
- How a person reruns from a fork with a different model or prompt: fork has
  no `--model` and the binary has no `meow session resume`, only `--continue`
  on the command (from crates/meow-cli/src/surface.rs:62-63 and 167-257, high).
