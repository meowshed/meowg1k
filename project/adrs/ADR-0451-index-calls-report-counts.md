---
id: ADR-0451
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2457]
supersedes: []
---

# 0451. `index` calls report counts rather than sentences

## Decision

`index.build`, `index.update` and `index.stats` report counts rather than
sentences (from https://github.com/meowshed/meowg1k/pull/142, high).

With this in place, `build` and `update` return a struct with `files`,
`added`, `changed`, `removed` and `embedded`, and `stats` returns `chunks`,
`embedded` and `model` (from crates/meow-star/src/modules.rs:881-913 and
984-995, high). `update` leaves `embedded` at zero, and `build` reports what
that call embedded (from crates/meow-cli/src/index.rs:208-225, high).

## Why

A handler deciding whether to keep going needs a number, and parsing one back
out of a sentence is how a report goes stale (from
https://github.com/meowshed/meowg1k/pull/142, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Report sentences | Printing as it is, the way `meow index build` reports to a person (reasoned from crates/meow-cli/src/index.rs, low) | A handler has to parse the number back out, and that is how a report goes stale (from https://github.com/meowshed/meowg1k/pull/142, high) |
| Do nothing: no `index` module, so a handler shells out to `meow index build` | No new module to keep (reasoned from https://github.com/meowshed/meowg1k/pull/142, low) | A script that indexes before it searches would shell out to the binary running it (from https://github.com/meowshed/meowg1k/pull/142, high) |

The source names no third shape for the result.

## What it costs

A handler that wants to show a person what happened formats the counts itself
(reasoned from crates/meow-star/src/modules.rs:984-995, low). The counts are
cast to `u32` for Starlark, so a count above about four billion would wrap
(from crates/meow-star/src/modules.rs:989-993, high).

## What would reverse it

This is reversed if a handler needs a fact that no count expresses, such as
which files were skipped and why, and the result grows a text field to carry
it (reasoned from crates/meow-index/src/walk.rs:35-50, low).

## Consequences

- A handler can call `update` and then decide from `added + changed` whether to
  spend on `build` (from crates/meow-star/src/modules.rs:889-892, high).
- With no index, each call fails saying so rather than reporting zeros (from
  crates/meow-star/tests/running.rs:2636, high).

## How I will know it was realised

`build_and_update_report_counts_rather_than_a_sentence` and
`stats_says_how_much_is_indexed_and_by_what` in
`crates/meow-star/tests/running.rs` pass (from
crates/meow-star/tests/running.rs:2528 and 2566, high).

## What this does not settle

- The walk's skipped files, which `Walked` records as `binary`, `too_large`
  and `escaping`, are not in the result, so a handler can't ask why a file is
  missing (from crates/meow-index/src/walk.rs:35-50 and
  crates/meow-star/src/modules.rs:984-995, high).
- `stats` has no count of files, only of chunks (from
  crates/meow-star/src/modules.rs:900-913, high).
