---
id: ADR-1405
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1436, REQ-1437]
supersedes: []
---

# 1405. The index records the embedding model that built it

## Decision

The index records which embedding model produced it, and a query against an
index built by a different model fails with both names (from docs/spec/index.md
[R-INDEX-051], high). The model's name is written to the key-value entry
`index.model` when embedding starts, and a query compares it with the model
asking before it embeds the query (from crates/meow-index/src/index.rs:16-17,
245-248 and 307-317, high).

Once this is accepted, a query by another model fails with a message naming
both and saying to rebuild or to go back to the model the index was built with
(from crates/meow-index/src/error.rs:38-50, high). What still doesn't work is a
build over an existing index with a new model: `embed` overwrites
`index.model` before it runs and embeds only the chunks with no vector, so the
old vectors stay and the index is recorded as the new model's (from
crates/meow-index/src/index.rs:236-258, high). A query by the new model then
passes the check and scores against the old vectors, which is the case this
decision exists to catch (reasoned from crates/meow-index/src/index.rs:307-317,
low).

## Why

Nothing in v0.2.x records which embedding model built the index, so changing the
model silently produces nonsense scores (from docs/spec/index.md, high). Two
models put different meanings in the same coordinates, so a query across them
produces numbers that look like scores and are not (from
https://github.com/retran/meowg1k/pull/131, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: record nothing, as in v0.2.x | Changing the model needs no rebuild and no error to handle (reasoned from docs/spec/index.md, low) | Changing the model silently produces nonsense scores (from docs/spec/index.md, high) |
| Record and check the vector dimension | Catches two models with the same name and different dimensions (from https://github.com/retran/meowg1k/pull/131, high) | The model name catches the case that actually happens, which is changing the model; nothing enforces a dimension (from https://github.com/retran/meowg1k/pull/131, high) |
| Rebuild the index when a query finds another model | The query answers without the caller acting (reasoned from crates/meow-index/src/error.rs:31-50, low) | A query must not build an index implicitly, because that would turn a typo into several minutes and a bill (from docs/spec/index.md [R-INDEX-041] and crates/meow-index/src/error.rs:31-36, high) |

## What it costs

A caller who changes the embedding model has to run `meow index clear` and
build again, and pays to re-embed every chunk (reasoned from
crates/meow-index/src/error.rs:38-50, low). The provider is fixed to one model
at construction, so an embedder can't switch model per call (from
crates/meow-llm/src/voyage.rs:40-44, high).

## What would reverse it

- The provider API returns the model and dimension with every vector, so the
  index can check each stored vector rather than one name (reasoned from
  https://github.com/retran/meowg1k/pull/131, low).

## Consequences

`meow index stats` prints the model that built the index, or `-` when none did
(from crates/meow-cli/src/wire.rs:776-785, high). `clear` deletes
`index.model` along with the rows (from crates/meow-index/src/index.rs:147-152,
high). The workspace declares its index model explicitly, and nothing chooses
one for it, because choosing for somebody is how an index gets built by one
model and queried by another (from crates/meow-star/src/declare.rs:339-344,
high).

## How I will know it was realised

1. `a_query_by_the_wrong_model_names_both` in
   crates/meow-index/tests/index.rs passes: an index built by `small` and
   queried by `large` fails with `WrongModel` naming both.
2. `clear_removes_the_index_and_nothing_else` in
   crates/meow-index/tests/index.rs passes: after `clear`, `model()` is empty.
3. A test that builds with one model, builds again with another over the same
   chunks, and queries, fails or re-embeds every chunk. No such test exists
   today, and the code as written would pass the query (from
   crates/meow-index/src/index.rs:236-258, high).

## What this does not settle

- A build with a new model over an existing index. It relabels the index
  without re-embedding the old chunks, which contradicts [R-INDEX-051] (from
  crates/meow-index/src/index.rs:245-248, high).
- The vector dimension. Nothing enforces one (from
  https://github.com/retran/meowg1k/pull/131, high).
- Whether deleting `index.model` on `clear` fits [R-INDEX-052], which says
  `clear` leaves the key-value store untouched. The entry is the index's own,
  and the test expects it gone (from crates/meow-index/src/index.rs:147-152 and
  crates/meow-index/tests/index.rs:447-470, medium).
