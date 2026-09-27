---
id: ADR-1403
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1435]
supersedes: []
---

# 1403. The index lives in the one database

## Decision

Vectors are stored through `meow-store` in the same database as everything else
(from docs/spec/index.md [R-INDEX-050], high). Migration 4 adds the
`index_files` and `index_chunks` tables to the workspace database, and a chunk's
vector is a column of its row (from crates/meow-store/src/migrations.rs:128-160,
high).

Once this is accepted, the index is one more set of tables in the file that
holds sessions, the cache and the blobs, so there's one file to keep in step
(from crates/meow-store/src/index.rs:6-9, high). What still doesn't work is the
reason the specification gives: a chunk keeps its text in the `text` column of
`index_chunks`, not in the blob table, so a chunk and a tool result quoting the
same file share nothing yet (from crates/meow-store/src/migrations.rs:147-156,
high). The `hnsw_rs` graph also lives outside the database, as files under
`.meow/.data/`, and is a cache rebuilt from the rows (from
crates/meow-index/src/ann.rs:12-13, high).

## Why

One database lets a chunk share the blob table with a tool result quoting the
same file (from docs/spec/index.md, high). A second SQLite file means two files
to keep in step (from crates/meow-store/src/index.rs:6-9, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: a separate SQLite database under `internal/adapters/sqlite/index/`, as in v0.2.x | Index writes and `meow index clear` can't touch the session tables, because they are in another file (reasoned from docs/spec/index.md [R-INDEX-052], low) | Two files have to be kept in step, and a chunk can't share the blob table with a tool result quoting the same file (from crates/meow-store/src/index.rs:6-9 and docs/spec/index.md, high) |
| Keep the vectors only in the `hnsw_rs` graph file | No second copy of each vector, and no rebuild from rows (reasoned from crates/meow-index/src/ann.rs:12-13, low) | A graph that is stale, half written or damaged would lose the index, where a graph that is a cache over the rows costs only a rebuild (from https://github.com/retran/meowg1k/pull/131, high) |

The history names no third option; these two are all that the specification,
the code and the pull requests record.

## What it costs

Every vector is stored twice, once in its row and once in the graph files the
query walks (from crates/meow-index/src/ann.rs:12-13 and
crates/meow-store/src/migrations.rs:147-156, high). The index shares the one
SQLite file with sessions, so a build's writes and a session's appends go to the
same database (reasoned from crates/meow-store/src/index.rs:6-9, low).
`clear` has to delete the index tables by name and nothing else, because
sessions and the key-value store sit beside them (from
crates/meow-store/src/index.rs:226-241, high).

## What would reverse it

- A build's writes measurably delay a session's appends in the shared database
  (reasoned from crates/meow-store/src/index.rs:6-9, low).

## Consequences

`index_chunks` rows carry a nullable `vector`, and a chunk with no vector is
the work an interrupted build resumes (from
crates/meow-store/src/migrations.rs:143-159, high). `clear` deletes the rows of
`index_chunks` and `index_files` in one transaction and leaves the rest of the
database alone (from crates/meow-store/src/index.rs:226-241, high). The index
records its model under the key-value entry `index.model` in the same database
(from crates/meow-index/src/index.rs:16-17, high). Sharing the blob table with
tool results is work still to do.

## How I will know it was realised

1. `the_index_shares_one_database` in crates/meow-index/tests/index.rs passes:
   after a build, `.meow/.data/` holds one `.db` file.
2. `clear_removes_the_index_and_nothing_else` in
   crates/meow-index/tests/index.rs passes: a key-value entry written before
   `clear` survives it.

## What this does not settle

- Whether a chunk's text moves into the blob table. The specification gives
  that sharing as the reason for this decision, and the schema doesn't do it
  (from crates/meow-store/src/migrations.rs:147-156, high).
- Where the graph lives. It stays in files beside the database, as a cache,
  under ADR-1401 (from crates/meow-index/src/ann.rs:12-13, high).
