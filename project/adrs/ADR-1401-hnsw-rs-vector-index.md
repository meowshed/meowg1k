---
id: ADR-1401
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1427, REQ-1434]
supersedes: []
---

# 1401. The vector index is `hnsw_rs`

## Decision

The index uses `hnsw_rs` for its vectors (from docs/spec/index.md, high). A
query walks an `hnsw_rs` graph with cosine distance to rank the nearest chunks,
which is how [REQ-1427] is answered, and passes the path filter into that walk,
which is how [REQ-1434] is answered (from crates/meow-index/src/ann.rs:172-238
and crates/meow-index/src/index.rs:296-336, high).

Once this is accepted, a query touches only the pages of the graph its walk
lands on, because the graph file is memory-mapped, and it reads chunk text only
for the rows that won (from https://github.com/meowshed/meowg1k/pull/131, high).
What still doesn't work is knowing the recall: the graph parameters (16
connections, `ef_construction` 200, a search four times as wide as its limit)
are the usual starting points and nothing measures them (from
crates/meow-index/src/ann.rs:22-39 and
https://github.com/meowshed/meowg1k/pull/131, high).

## Why

`hnsw_rs` is pure Rust, so it keeps the single static binary and the
`unsafe_code = "deny"` lint intact, which `usearch` wouldn't (from
docs/spec/index.md, high). Of the pure Rust options, it's the one with a track
record on the path every search takes (from
https://github.com/meowshed/meowg1k/pull/131, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: compare the query against every stored vector, as the first version of the branch did | No dependency and exact recall (reasoned from crates/meow-index/src/ann.rs:6-10, low) | It is linear in the corpus and read about a thousand chunks and eight megabytes off disk per question on this repository, growing with the corpus (from https://github.com/meowshed/meowg1k/pull/131, high) |
| `usearch`, a C++ implementation | Recall or build time, which is when it earns its cost (from docs/spec/index.md, medium) | It breaks the single static binary and the `unsafe_code = "deny"` lint (from docs/spec/index.md, high) |
| `fast-hnsw` | Two transitive crates in place of about twenty (from https://github.com/meowshed/meowg1k/pull/131, high) | It is a new crate with no track record on the path every search takes, and the specification names `hnsw_rs` (from https://github.com/meowshed/meowg1k/pull/131, high) |

v0.2.x used `coder/hnsw`, a Go library, so keeping it wasn't an option for the
Rust tree (from docs/design/0.3.0-architecture.md:317,
high).

## What it costs

`hnsw_rs` pulls about twenty crates, including `mmap-rs`, `rayon`, `sysctl`,
`jiff` and `windows 0.48` (from https://github.com/meowshed/meowg1k/pull/131,
high). It pins the unmaintained `bincode` 1.x for its file format, so
`deny.toml` carries RUSTSEC-2025-0141 as an ignored advisory (from
deny.toml:31-35, high). The graph lives in files under `.meow/.data/` beside the
database, and a missing, stale or damaged graph costs a rebuild on the next
query (from crates/meow-index/src/ann.rs:12-13 and
crates/meow-index/src/index.rs:372-400, high).

## What would reverse it

- Recall or build time on this repository is measured and found unacceptable
  (from docs/spec/index.md, high).
- A vulnerability, not only an unmaintained notice, is published against
  `bincode` 1.x while `hnsw_rs` still has no feature that drops it (reasoned
  from deny.toml:31-35, low).

## Consequences

The graph is a cache over the `index_chunks` rows, stamped with the chunk
count and the highest identifier; a query rebuilds it when the stamp doesn't
match or the graph won't load (from crates/meow-index/src/ann.rs:12-13, 41-51
and crates/meow-index/src/index.rs:372-400, high). `clear` removes the stamp,
so the next query rebuilds (from crates/meow-index/src/ann.rs:106-111, high).
The `deny.toml` exception has to be revisited whenever `hnsw_rs` is upgraded
(reasoned from deny.toml:31-35, low).

## How I will know it was realised

1. `the_graph_is_written_once_and_reused`, `a_stale_graph_is_rebuilt`,
   `a_damaged_graph_is_rebuilt_rather_than_fatal` and `clear_forgets_the_graph`
   in crates/meow-index/tests/index.rs pass.
2. `results_are_ranked_and_citable` and `a_path_filter_applies_before_ranking`
   in crates/meow-index/tests/index.rs pass.
3. `mise run deny` passes with `hnsw_rs` in the tree, and `unsafe_code =
   "deny"` stays in Cargo.toml.

## What this does not settle

- The graph parameters. They are untuned and are the numbers to move once
  recall on a real corpus is measured (from
  https://github.com/meowshed/meowg1k/pull/131, high).
- A check on the vector dimension. Nothing enforces one, so two models with the
  same name and different dimensions would score badly rather than fail (from
  https://github.com/meowshed/meowg1k/pull/131, high).
- Where the vectors are stored. They stay in the database, and
  ADR-1403 settles that.
