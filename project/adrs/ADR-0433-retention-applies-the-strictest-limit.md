---
id: ADR-0433
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2253, REQ-2254, REQ-2256]
supersedes: []
---

# 0433. Retention applies the strictest of its limits

## Decision

Retention takes the strictest of the limits it was given rather than the first
that matches (from https://github.com/meowshed/meowg1k/pull/128, high). A name
protects a whole tree, because a parent can't go without its descendants (from
https://github.com/meowshed/meowg1k/pull/128, high).

Once this is accepted, a sweep deletes every session that any configured limit
selects, oldest first, and keeps a whole tree when any session in it is named
and named sessions weren't asked for (from
crates/meow-session/src/retention.rs:72 and 121, high). Only the age and count
limits are reachable today: `meow session gc` passes no size limit, and no
declaration configures one (from crates/meow-cli/src/wire.rs:737, and a search
for `keep_days` and `max_size` under crates/meow-star/src, high).

## Why

Three limits that each select a different set would otherwise depend on the
order they were checked in, which nobody could predict from the configuration
(from https://github.com/meowshed/meowg1k/pull/128, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no retention | Never deletes a run somebody might want (reasoned, low) | An agent log grows fast, and a design without a collection story ends with a gigabyte of SQLite nobody wants to clone (from docs/design/0.3.0-sessions.md section 8, high) |
| Apply the first limit that matches | Stops at one check per session (reasoned from crates/meow-session/src/retention.rs:64, low) | The result would depend on the order the limits were checked in (from https://github.com/meowshed/meowg1k/pull/128, high) |

## What it costs

A sweep deletes the union of what the limits select, so it deletes at least as
much as the most aggressive limit alone (from
crates/meow-session/src/retention.rs:72, high). One named descendant keeps its
whole tree, however old or large the unnamed parent is (from
crates/meow-session/src/retention.rs:121, high).

## What would reverse it

- A user needs one limit to override another, for example "keep the last 200
  whatever their age". The strictest rule can't express an exception (reasoned
  from crates/meow-session/src/retention.rs:72, low).

## Consequences

- A sweep reports what it kept as well as what it deleted, so a protected tree
  is visible (from crates/meow-session/src/retention.rs, struct `Swept`, high).
- Deletion runs leaves first, because the parent foreign key refuses to orphan
  a child (from crates/meow-session/src/retention.rs:140 and
  crates/meow-store/src/migrations.rs:44, high).

## How I will know it was realised

1. crates/meow-session/tests/fork.rs `the_strictest_limit_wins`,
   `a_named_session_is_kept_unless_it_is_asked_for` and
   `a_sweep_takes_a_parent_together_with_its_children` pass (from those tests,
   high).
2. A sweep over a tree whose only named session is a child keeps the parent;
   no test covers it today (from a search of crates/meow-session/tests, high).
3. `meow session gc` applies a size limit alongside age and count; it can't
   today (from crates/meow-cli/src/wire.rs:737, high).

## What this does not settle

- Where the limits are configured. The design proposes `meow.sessions(keep_days,
  keep_last, max_size)`, and no such builtin exists (from
  docs/design/0.3.0-sessions.md section 8 and a search of crates/meow-star/src,
  high).
- How the size limit is applied; ADR-0434 decides that.
