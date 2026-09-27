---
id: BUG-0026
artifact: bug
status: approved
severity: major
violates: REQ-2605
found: 2026-09-27
revised: 2026-09-27
issue: 199
---

# The index's graph files are created readable by everyone

The HNSW graph and its side files, `index.hnsw.*`, sit beside `meow.db` in
`.meow/.data/` and are written with the process's default mode, 0644 under the
usual umask. The database itself is narrowed to 0600 only after SQLite has
created it.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. In a workspace that declares an index, run `meow index build` and then any
   `search.code` query, which builds the graph.
2. Run `ls -l .meow/.data/`.

`meow.db`, `meow.db-wal` and `meow.db-shm` are `-rw-------`; `index.hnsw.data`,
`index.hnsw.graph`, `index.hnsw.ids` and `index.hnsw.stamp` are `-rw-r--r--`.
This was traced through the code and not run.

## What the system does

`crates/meow-index/src/index.rs:404-407` places the graph in `.meow/.data/`.
`crates/meow-index/src/ann.rs:149` has `hnsw_rs` dump the graph, and
`crates/meow-index/src/ann.rs:157-167` writes the ids and the stamp with
`std::fs::write`; none of them sets a mode. `Store::open_at` in
`crates/meow-store/src/lib.rs:75-92` lets SQLite create the file and only then
calls `restrict_permissions` (`crates/meow-store/src/lib.rs:158-170`), so for
the time between the two the database is readable by other users. The test at
`crates/meow-store/tests/spec.rs:113-126` checks only the main file.

## What it should do, and why

REQ-2605: "The database and its side files MUST be created readable and
writable by their owner only, on every platform that has file permissions." The
graph holds a vector for every chunk of the workspace's code, and the ids file
maps them back to chunks, so it is as sensitive as the rows it caches. REQ-2600
keeps all of a workspace's data in one database; ADR-1403 records the graph as a
cache outside it.

## Triage

The defect enters in `meow-index`'s `ann` module, which writes files the store
doesn't know about, and in the store's create-then-restrict order. It is major:
the requirement's main behaviour fails for four of seven files. It isn't rated
critical because `.meow/.data/` already sits inside the user's checkout, whose
own mode usually limits other users.

## Closed by

A test in `crates/meow-index/tests/index.rs`, named for example
`the_graph_files_are_owner_only`, that builds a graph and expects mode 0600 on
every file under `.meow/.data/`, and a store test that checks the WAL and SHM
files as well as the main one.
