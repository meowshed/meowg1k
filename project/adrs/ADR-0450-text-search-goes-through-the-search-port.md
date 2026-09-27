---
id: ADR-0450
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2441, REQ-2442]
supersedes: []
---

# 0450. `search.text` and `search.files` go through the search port

## Decision

`search.text` and `search.files` go through the port rather than becoming
capabilities beside `fs` (from https://github.com/retran/meowg1k/pull/142,
high).

With this in place, both calls are methods of the `Search` port, and the binary
answers them with a `Walker` that runs `meow_index::Walk`, the walk the index
uses (from crates/meow-star/src/port.rs:194-217 and
crates/meow-cli/src/index.rs:299-318, high). A workspace that declares no
index gets `Unindexed`, which still walks and still answers both calls (from
crates/meow-cli/src/index.rs:380-442, high).

## Why

`R-STAR-019` asks them to reach exactly the files the index reaches, and that
walk lives in `meow-index` (from https://github.com/retran/meowg1k/pull/142,
high). A port costs nothing and keeps one `.gitignore` deciding what is
searchable however a handler searches (from
https://github.com/retran/meowg1k/pull/142, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Capabilities beside `fs` | A pure function with no port, like the other modules that need no state (from crates/meow-star/src/port.rs:194-199, medium) | They'd need a new edge from `meow-star` to `meow-index`, which the architecture doesn't have (from https://github.com/retran/meowg1k/pull/142, high) |
| A port whose `text` and `files` answer an empty list when no index is built | Nothing to wire for a workspace without an index (reasoned from crates/meow-cli/src/index.rs:255-262, low) | A handler searching a workspace that never ran `meow index build` was told there was nothing in it (from crates/meow-cli/src/index.rs:255-262 and crates/meow-cli/tests/surface.rs:774-781, high) |
| Do nothing: only `search.code`, which needs an index | No new calls to keep (reasoned from https://github.com/retran/meowg1k/pull/142, low) | A workspace where `meow index build` has never run should still be searchable (from https://github.com/retran/meowg1k/pull/142, high) |

## What it costs

Every implementor of `Search` has to implement `text` and `files`, including
the test doubles and the refusing ports (from crates/meow-star/src/port.rs:340
and crates/meow-star/tests/running.rs:201, high). `search.text` reads each file
the walk returns and scans it line by line, so its cost grows with the walked
files and not with an index (from https://github.com/retran/meowg1k/pull/142,
high).

## What would reverse it

This is reversed if the walk moves out of `meow-index` into a crate that
`meow-star` may depend on, because then a capability could share it without a
new edge (reasoned from CLAUDE.md, architecture table, low).

## Consequences

- `search.text` and `search.files` work in a workspace with no index, and the
  calls that need one say so (from crates/meow-cli/tests/surface.rs:782 and
  crates/meow-cli/tests/search.rs:44, high).
- A hit from `search.text` carries a score of 1.0, so every search returns one
  shape (from https://github.com/retran/meowg1k/pull/142, high).
- Paths are reported relative to the canonical root with `/`, which a defect on
  Windows and through symbolic links forced (from
  crates/meow-cli/src/index.rs:278-290, high).

## How I will know it was realised

These tests pass: `search_text_and_files_work_without_an_index` in
`crates/meow-cli/tests/surface.rs`,
`a_root_that_is_not_canonical_still_finds_its_files` in
`crates/meow-cli/tests/search.rs`, and `search_text_reports_where_each_hit_was`
and `search_files_answers_with_paths` in `crates/meow-star/tests/running.rs`
(from those files, high). No test yet checks that a file `.gitignore` excludes
is absent from `search.text`, which is the half of [R-STAR-019] this decision
exists for (from grep over crates/meow-cli/tests and crates/meow-star/tests,
medium).

## What this does not settle

- Whether `search.text` is fast enough on a large workspace; the pull request
  says a workspace where it is too slow wants `search.code` (from
  https://github.com/retran/meowg1k/pull/142, high).
- `Unindexed` walks with `Walk::default()`, not a walk built from the
  workspace's declaration; today `Walk` carries only `max_bytes`, so the two
  agree, but a future walk setting would need wiring here too (from
  crates/meow-cli/src/index.rs:396 and crates/meow-index/src/walk.rs:61-72,
  high).
