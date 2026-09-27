---
id: ADR-2404
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2434, REQ-2435, REQ-2436, REQ-2553]
supersedes: []
---

# 2404. A CSV with a header is a list of dictionaries

## Decision

`csv.parse` returns a list of dictionaries when the first record names the
columns, and a list of lists when it doesn't (from docs/spec/starlark.md
[R-STAR-017], high).

## Why

Returning rows and a separate header list makes every caller zip them, which is
the same work done once per handler in place of once in the module (from
docs/spec/starlark.md, Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Return rows and a separate header list | Keeps a header with repeated names intact, and a ragged record needs no rule (reasoned from crates/meow-star/src/modules.rs `csv.parse`, low). | Every caller zips them, the same work done once per handler in place of once in the module (from docs/spec/starlark.md, Decisions, high). |
| Always return a list of lists | One shape for every file, whatever its first record holds (reasoned from crates/meow-star/src/modules.rs `csv.parse`, low). | It is the header-list option with the header left in the rows, so every caller still zips (reasoned from docs/spec/starlark.md, Decisions, low). |
| Do nothing: no `csv` module, as before PR #141 | No new surface (reasoned from https://github.com/meowshed/meowg1k/pull/141, low). | The design table promised `csv`, and [R-STAR-015] requires it in the table (from https://github.com/meowshed/meowg1k/pull/141 and docs/spec/starlark.md [R-STAR-015], high). |

## What it costs

A record whose length disagrees with the header can't become a dictionary, so
`csv.parse` fails naming the record number (from docs/spec/starlark.md
[R-STAR-017] and crates/meow-star/src/modules.rs `csv.parse`, high). A header
that repeats a name loses a column without an error: `parse("a,a\n1,2\n")`
returns `[{"a": "2"}]` (from `target/debug/meow` built on 2026-09-27, high).
Every field is a string; `csv.parse` converts no number (from
crates/meow-star/src/modules.rs `csv.parse`, high).

## What would reverse it

Evidence that handlers mostly read files whose header is unreliable, for
example with repeated names, would favour the header-list shape (reasoned from
the cost above, low).

## Consequences

A file whose first record is data and not names is the case `header = False`
exists for (from docs/spec/starlark.md, Decisions, high). `csv.encode` reads the
Starlark values directly and not through JSON, because `serde_json::Map` sorts
its keys and would reorder a CSV's columns (from
https://github.com/meowshed/meowg1k/pull/141, high).

## How I will know it was realised

`csv_shape_follows_whether_the_first_record_names_the_columns`,
`a_short_csv_record_names_which_record_it_was` and
`csv_encode_round_trips_both_shapes` in crates/meow-star/tests/running.rs pass
(from those tests, high).

## What this does not settle

The `header` argument itself: no requirement in docs/spec/starlark.md names it,
and the code makes it a named argument defaulting to `True` (from
docs/spec/starlark.md [R-STAR-017] and crates/meow-star/src/modules.rs
`csv.parse`, high). So `csv.parse` doesn't detect whether the first record
names the columns, as [R-STAR-017] reads; the caller says so (from
crates/meow-star/src/modules.rs `csv.parse`, high). What a header with
repeated names should do (from the probe above, high).
