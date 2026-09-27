---
id: ADR-2407
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2441, REQ-2442]
supersedes: []
---

# 2407. `search.text` does not need an index

## Decision

`search.text` and `search.files` search the workspace without an index, and obey
the same walk the index obeys (from docs/spec/starlark.md [R-STAR-019], high).

## Why

Matching literal text is a walk and a comparison. Requiring an embedding model
and a built graph for it would make the cheap search depend on the expensive one
(from docs/spec/starlark.md, Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Serve `search.text` from the index | A search reads indexed chunks and doesn't scan every file line by line (reasoned from https://github.com/retran/meowg1k/pull/142, low). | The cheap search would depend on an embedding model and a built graph (from docs/spec/starlark.md, Decisions, high). |
| Make `search.text` and `search.files` capabilities beside `fs` | They would need no port (from https://github.com/retran/meowg1k/pull/142, high). | They must reach the files the index reaches, and that walk lives in `meow-index`; a capability would need a `meow-star` to `meow-index` edge the architecture doesn't have (from https://github.com/retran/meowg1k/pull/142, high). |
| Do nothing: answer both with an empty list when there is no index, as the wiring did before PR #147 | Already built (from https://github.com/retran/meowg1k/pull/147, high). | `search.files("**/*.rs")` came back empty while `fs.glob("**/*.rs")` in the same handler found the file, so a handler was told the workspace was empty (from https://github.com/retran/meowg1k/pull/147, high). |

## What it costs

`search.text` reads every file the walk returns and scans it line by line. The
walk refuses binary and oversized files, so the cost is bounded by the walk and
not by the corpus, but it is a linear scan; a workspace where it is too slow
wants `search.code` (from https://github.com/retran/meowg1k/pull/142, high).
Each search call canonicalises the root once, which is one `stat` (from
https://github.com/retran/meowg1k/pull/147, high).

## What would reverse it

A workspace where the linear scan is too slow for the searches handlers make
would favour serving literal text from an index (from
https://github.com/retran/meowg1k/pull/142, medium).

## Consequences

`search.text` shares the walk with the index, so one `.gitignore` decides what
is searchable however a handler searches (from docs/spec/starlark.md, Decisions,
high). A hit from `search.text` carries a score of 1.0, so every search returns
one shape (from https://github.com/retran/meowg1k/pull/142, high).

## How I will know it was realised

`search_text_reports_where_each_hit_was`,
`search_text_takes_a_regular_expression_when_asked` and
`search_files_answers_with_paths` in crates/meow-star/tests/running.rs,
`search_text_and_files_work_without_an_index` in
crates/meow-cli/tests/surface.rs, and
`a_root_that_is_not_canonical_still_finds_its_files` in
crates/meow-cli/tests/search.rs pass (from those tests, high). No test checks
that a file the walk ignores is missing from a `search.text` result, which is
the second half of [R-STAR-019] (reasoned from a search of crates/*/tests for
`gitignore` and `meowignore`, medium).

## What this does not settle

The walk a workspace gets when it declares an index whose provider or model is
missing: `searcher` then builds `Unindexed`, which walks with `Walk::default()`
and not the declared limits, so `search.text` can reach files the index would
refuse (from crates/meow-cli/src/wire.rs `searcher`,
crates/meow-cli/src/index.rs `Unindexed::new` and
https://github.com/retran/meowg1k/pull/147, high).
