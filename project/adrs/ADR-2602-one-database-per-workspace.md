---
id: ADR-2602
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2600]
supersedes: []
---

# 2602. One SQLite database holds all of a workspace's data

## Decision

The store keeps all of one workspace's data in a single SQLite database at
`.meow/.data/meow.db` [REQ-2600] (from docs/spec/store.md, high).

## Why

The blob table is shared, and SQLite can't enforce a reference count across
files (from docs/spec/store.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep v0.2.x's separate SQLite files for the index, cache, sessions and metadata, under `internal/adapters/sqlite/` | Each file can be deleted or rebuilt alone, for example dropping the index without touching sessions, and there is no schema to merge (reasoned from docs/spec/store.md, Changes from v0.2.x, low) | The blob table is shared and a cross-file reference count isn't something SQLite can enforce (from docs/spec/store.md, Changes from v0.2.x, high) |
| A sessions-only database at `.meow/.data/sessions.db`, as the sessions design first drew it | Session data stays apart from the index and the cache, so each can grow and be collected on its own (reasoned from docs/design/0.3.0-sessions.md section 4, low) | The blob table is shared, so a session and a fork of it have to count references in one file; the spec replaced the path with `meow.db` (from docs/design/0.3.0-sessions.md section 4, added in commit ad3f907, and docs/spec/store.md [R-STORE-001], added in commit f8e58ea, medium) |

## What it costs

Every writer in the workspace shares one write lock, so an index build and an
agent run contend for it. [R-STORE-024] pays that cost by making a bulk write
commit in batches of 256 rows, so indexing can't block an agent run (from
docs/spec/store.md [R-STORE-024] and crates/meow-store/src/lib.rs:46-51, high).

## What would reverse it

A payload that no longer needs to be shared between the tables that refer to
it would remove the reason: the argument rests wholly on the blob table's
reference count spanning sessions and forks (reasoned from docs/spec/store.md,
Changes from v0.2.x, low).

## Consequences

- `Store::open` resolves `.meow/.data/meow.db` beneath the workspace root and
  creates the directory if it is missing (from
  crates/meow-store/src/lib.rs:60-72, high).
- Sessions, events, blobs, the key-value table, the response cache and the
  index rows are tables in that one file, so deleting a session decrements the
  count of every blob it referenced in the same transaction (from
  docs/spec/store.md, Scope and [R-STORE-040], high).
- `meow-index` still writes its HNSW graph as files beside the database, named
  `index.hnsw.*`, and rebuilds it from the vectors in the database when the
  stamp doesn't match (from crates/meow-index/src/index.rs:390-407 and
  crates/meow-index/src/ann.rs:80-112, high).

## How I will know it was realised

1. `the_database_lives_at_one_known_path` in crates/meow-store/tests/spec.rs
   passes: `Store::open` returns a store whose path is
   `.meow/.data/meow.db` beneath the root, and the file exists (from
   crates/meow-store/tests/spec.rs:24-34, high).
2. No workspace data lives under `.meow/.data/` outside `meow.db` and its
   side files. The HNSW graph files break this today; see the third
   consequence.

## What this does not settle

- Whether the HNSW graph, a cache derived from the vectors in `meow.db`,
  counts as workspace data under [REQ-2600]. The code treats it as rebuildable
  and keeps it outside the database (from
  crates/meow-index/src/index.rs:390-407,
  high); the requirement says "all data" (from docs/spec/store.md [R-STORE-001],
  high).
- Credentials, which live outside the workspace in the operating system's
  secret store (from docs/adrs/ADR-0467, high).
