---
id: ADR-2205
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2400, REQ-2600]
supersedes: []
---

# 2205. Sessions are workspace-local

## Decision

A run against a monorepo subdirectory writes its session to that subdirectory's
`.meow/`, which [R-SESSION-034] and [R-STAR-001] already require between them
(from docs/spec/session.md, high).

Once this holds, the workspace is the nearest ancestor with `.meow/meow.star`,
and its sessions live in `.meow/.data/meow.db` beneath that root, in the one
database the store keeps (from crates/meow-star/src/workspace.rs:21-47 and
crates/meow-store/src/lib.rs:61-71, high). Sessions belong to the repository
they describe and don't follow the user between checkouts (from
docs/design/0.3.0-sessions.md section 4, high).

## Why

A global store would need a global identifier scheme and a way to decide which
workspace a session belongs to, and nobody has asked for either (from
docs/spec/session.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A global session store, such as the per-user database v0.2.x kept under `$XDG_DATA_HOME/meowg1k/` for its cache | One list of every run across repositories, which survives deleting a checkout (reasoned from v0.2.1 internal/adapters/sqlite/path/service.go:72-100, low) | It needs a global identifier scheme and a way to decide which workspace a session belongs to, and nobody has asked for either (from docs/spec/session.md, high) |

Doing nothing isn't a separate option: v0.2.x already kept sessions in the
workspace, in `.meowg1k/.data/project.db`, and this decision keeps that and
moves them into the single `.meow/.data/meow.db` (from v0.2.1
internal/adapters/sqlite/host.go:150-160 and
internal/adapters/sqlite/path/service.go:48-70, high). The question admits two
places, the workspace or the user, so the table has one row.

## What it costs

A person can't list runs from several repositories at once, and deleting a
checkout deletes its sessions (reasoned from docs/design/0.3.0-sessions.md
section 4, low). Two checkouts of one repository keep separate histories (from
docs/design/0.3.0-sessions.md section 4, medium).

## What would reverse it

- Someone asking for a global identifier scheme or a cross-workspace session
  list, the two things the source says nobody has asked for (from
  docs/spec/session.md, high).

## Consequences

- `@last` and `@<agent-name>` resolve within the one workspace, because the
  store they query is that workspace's database (from
  crates/meow-session/src/resolve.rs:74-92 and
  docs/spec/session.md [R-SESSION-032], high).
- A short identifier needs to be unique only within a workspace, which lets the
  entropy come from the clock and the process identifier (from
  https://github.com/meowshed/meowg1k/pull/129, high).
- A nested workspace inside a monorepo keeps sessions apart from its parent's,
  because discovery stops at the first `.meow/meow.star` (from
  crates/meow-star/src/workspace.rs:34-47, high).

## How I will know it was realised

1. `discovery_takes_the_nearest_ancestor_and_merges_nothing` in
   crates/meow-star/tests/loading.rs passes (from
   crates/meow-star/tests/loading.rs:47-66, high).
2. `the_database_lives_at_one_known_path` in crates/meow-store/tests/spec.rs
   passes (from crates/meow-store/tests/spec.rs:24-26, high).

## What this does not settle

- Whether the database is gitignored or shared between machines: the design
  says it is gitignored, and no requirement here checks it (from
  docs/design/0.3.0-sessions.md section 4, medium).
- How long sessions are kept, which retention settles (from
  docs/spec/session.md [R-SESSION-082], high).
