---
id: BUG-0001
artifact: bug
status: approved
severity: major
violates: REQ-1437
found: 2026-09-27
revised: 2026-09-27
issue: 174
---

# Embedding with a new model relabels the old vectors as the new model's

`Index::embed` writes the embedder's name into `index.model` before it embeds
anything, and it embeds only chunks that have no vector yet. After the model
changes, the index claims the new model built it while every existing vector
still comes from the old one, so a query by the new model passes the model
check and compares vectors from two models.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments. In a test beside
`a_query_by_the_wrong_model_names_both` in `crates/meow-index/tests/index.rs`:

1. Build an index over a small workspace with `index.update()`.
2. Embed it with `index.embed(&Counting::new("small"), 8)`.
3. Embed it again with `index.embed(&Counting::new("large"), 8)`. It returns 0,
   because no chunk is pending.
4. Query with `index.query(&Counting::new("large"), "retry", &Query::default())`.

The query returns hits. It should fail with `IndexError::WrongModel` naming
both models, or the second embed should have re-embedded every chunk.

## What the system does

`crates/meow-index/src/index.rs:246-248` puts `embedder.model()` under
`index.model` unconditionally, before the loop. The loop reads
`index_pending`, which selects only rows whose `vector` is null
(`crates/meow-store/src/index.rs:115-121`), so the vectors the old model wrote
stay. `Index::query` then compares the stored name with the query's model
(`crates/meow-index/src/index.rs:308-316`), finds them equal, and ranks the old
vectors against a query vector from the new model.

## What it should do, and why

REQ-1437: "A query against an index built by a different embedding model MUST
fail with both names rather than returning meaningless scores." REQ-1436 asks
the index to record which model produced it, and the recorded name is only
true if it covers every vector. ADR-1405 records the decision.

## Triage

The defect enters in `meow-index`, in `Index::embed`. It is major because the
requirement's main behaviour, refusing to mix models, doesn't happen on the
path a user takes after changing `meow.index(model = ...)` and running
`meow index build`. The fix is to compare the stored name before embedding and
either clear the vectors or refuse when it differs.

## Closed by

A test in `crates/meow-index/tests/index.rs`, named for example
`embedding_with_another_model_does_not_relabel_the_index`, that runs the steps
above and expects either `WrongModel` from the query or every chunk re-embedded
by the second model.
