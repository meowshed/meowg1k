---
id: ADR-2406
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2459, REQ-2461, REQ-2462]
supersedes: []
---

# 2406. No index and no results are different answers

## Decision

An `index` call with no declared index, or a `query` against an index never
built, fails saying so and never returns an empty list (from
docs/spec/starlark.md [R-STAR-025], high).

## Why

A handler that searches and finds nothing takes a different path from one whose
search couldn't run, and an empty list would collapse the two (from
docs/spec/starlark.md, Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Return an empty list when there is no index | A handler that treats "no index" as "nothing found" needs no error handling, which matters because Starlark has no `try` (reasoned from https://github.com/bazelbuild/starlark/blob/master/spec.md, medium). | It collapses "found nothing" and "couldn't search" into one answer (from docs/spec/starlark.md, Decisions, high). |
| Do nothing: keep the port that answered an empty list, as `quiet::NoIndex` did before PR #147 | Already built (from https://github.com/meowshed/meowg1k/pull/147, high). | A handler searching a workspace where `meow index build` never ran was told the workspace was empty, a wrong answer and not an error it could act on (from https://github.com/meowshed/meowg1k/pull/147, high). |
| Build the index on the first query | The handler gets results (reasoned from crates/meow-index/src/error.rs `Empty`, low). | A query that quietly built an index would turn a typo into several minutes and a bill (from crates/meow-index/src/error.rs `Empty`, high). |

## What it costs

A handler can't recover from the failure, because Starlark has no `try`, so a
handler that wants to work with or without an index must avoid the ranking
calls or use `search.text` (reasoned from
https://github.com/bazelbuild/starlark/blob/master/spec.md and
crates/meow-cli/src/index.rs `Unindexed`, medium).

## What would reverse it

Handlers that mostly treat "no index" as "nothing found" would reverse it,
because for them the failure is the cost above with nothing gained (reasoned
from docs/spec/starlark.md, Decisions, low). A handler has no way today to ask
whether an index exists without failing: `index.stats()` fails too (from
crates/meow-cli/src/index.rs `Unindexed` and crates/meow-star/tests/running.rs
`no_index_is_told_apart_from_no_results`, high).

## Consequences

The same reasoning made `search.code` fail, not return nothing, when `meow index
build` has never been run (from docs/spec/starlark.md, Decisions, high). Each of
the four reasons there is no index carries its own message - "declares no
index", "not a declared model", "no provider" and "the index will not open" -
where all four used to say to run `meow index build` (from
https://github.com/meowshed/meowg1k/pull/147, high).

## How I will know it was realised

`no_index_is_told_apart_from_no_results` in crates/meow-star/tests/running.rs,
`ranking_without_an_index_says_so_rather_than_answering` in
crates/meow-cli/tests/surface.rs,
`ranking_says_why_it_cannot_rather_than_returning_nothing` in
crates/meow-cli/tests/search.rs and `an_empty_index_says_so` in
crates/meow-index/tests/index.rs pass (from those tests, high).

## What this does not settle

Which model an index uses when the workspace declares none: [R-STAR-025] also
forbids choosing one, and ADR-0105 settles that (from
docs/adrs/ADR-0105-nothing-chooses-the-index-model.md, high).
