---
id: ADR-0442
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1423, REQ-1424]
supersedes: []
---

# 0442. The staleness hash covers the chunking parameters as well as the content

## Decision

The staleness hash covers the chunking parameters, not just the content (from
https://github.com/meowshed/meowg1k/pull/131, high).

What works now: `Index::update` hashes the text `lines=N overlap=N max=N`
followed by the file's content, compares it with the hash stored for the path,
and re-chunks the file when the two differ, so changing `chunk_lines` or
`overlap` in `meow.index(...)` makes every file stale on the next update (from
crates/meow-index/src/chunk.rs:85, crates/meow-index/src/index.rs:191 and
crates/meow-index/src/index.rs:492, high). The embedding model is not part of
the hash: a change of model is caught separately, by the index recording which
model built it and refusing a query from another, per `R-INDEX-051` (from
docs/spec/index.md, Storage, high).

## Why

Hashing content alone would leave chunks that look current after somebody
changed the chunk size, and those chunks answer queries (from
https://github.com/meowshed/meowg1k/pull/131, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: hash the content alone, as v0.2.x keyed each document version on its content hash | One hash per file content, shared by any chunking, so a change of chunk size costs no re-embedding (reasoned from v0.2.1:internal/adapters/sqlite/index/migrations.go, low) | Chunks look current after a change to the chunk size, and they answer queries (from https://github.com/meowshed/meowg1k/pull/131, high) |
| Record the chunking parameters once beside the index and refuse or clear on a change, the way the embedding model is recorded | One comparison per update in place of one per file, and a clear message naming the change (reasoned from docs/spec/index.md `R-INDEX-051`, low) | The source doesn't weigh it; a per-file hash also re-chunks only the files a change actually affects and needs no second code path (reasoned from crates/meow-index/src/index.rs:191, low) |

## What it costs

Any change to a chunking parameter re-chunks every file, and every new chunk
is embedded again at the provider's price (from
crates/meow-index/tests/index.rs `changing_the_chunking_makes_everything_stale`,
high; reasoned about the embedding cost from
crates/meow-index/src/index.rs:198, low). `max_chars` is in the hash and is not
declarable, so a release that changes `DEFAULT_MAX_CHARS` makes every user's
index stale on the next update (from crates/meow-cli/src/index.rs:84 and
crates/meow-index/src/chunk.rs:85, medium).

## What would reverse it

Chunking stops depending on its parameters, for example a syntax-aware
splitter whose boundaries come from the grammar alone, which
`docs/spec/index.md` names as the amendment to make once a recall number
justifies it (from docs/spec/index.md, Decisions, "Chunking is
line-oriented", medium).

## Consequences

- Stored hashes are not plain content hashes, so nothing else can reuse them to
  ask whether a file's bytes changed (reasoned from
  crates/meow-index/src/index.rs:492, low).
- An update after a change to `chunk_lines` or `overlap` reports every file as
  `changed` and none as `unchanged` (from crates/meow-index/tests/index.rs:200,
  high).

## How I will know it was realised

In `crates/meow-index/tests/index.rs`,
`changing_the_chunking_makes_everything_stale` passes: an index built with the
default chunking and updated with `lines = 5, overlap = 1` reports three
changed files and none unchanged (from
crates/meow-index/tests/index.rs:200, high). `only_a_changed_file_is_re_chunked`
in the same file pins the content half (from
crates/meow-index/tests/index.rs:179, high).

## What this does not settle

- Whether a change of embedding dimension under the same model name should make
  the index stale; nothing enforces a vector dimension (from
  https://github.com/meowshed/meowg1k/pull/131, "What is not covered", high).
- Whether a change of the walk's `max_bytes` limit belongs in the hash; it isn't
  there, and a file it newly excludes is removed by `R-INDEX-031` instead (from
  crates/meow-index/src/index.rs:222, medium).
