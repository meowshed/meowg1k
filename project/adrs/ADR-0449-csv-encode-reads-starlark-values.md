---
id: ADR-0449
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2436]
supersedes: []
---

# 0449. `csv.encode` reads the Starlark values directly

## Decision

`csv.encode` reads the Starlark values directly, where a dict keeps insertion
order (from https://github.com/meowshed/meowg1k/pull/141, high).

With this in place, `csv.encode` writes the header in the order the first
dict's keys were inserted, and a list of lists writes no header (from
crates/meow-star/src/modules.rs:375-410, high). `xml.encode` still converts its
argument to a JSON value first, so an attribute dict it encodes comes out in
sorted key order (from crates/meow-star/src/modules.rs:479, high).

## Why

`serde_json::Map` sorts its keys, so going through JSON would have renamed a
CSV's first column to whichever name sorts first (from
https://github.com/meowshed/meowg1k/pull/141, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Encode through the JSON value | One conversion shared with `json`, `yaml`, `toml` and `xml`, which all start from `to_json_value` (from crates/meow-star/src/modules.rs:137-205, medium) | `serde_json::Map` sorts keys and reorders the columns (from https://github.com/meowshed/meowg1k/pull/141, high) |
| Read the Starlark values directly (chosen) | Keeps the column order the handler wrote (from crates/meow-star/src/modules.rs:375-378, high) | Chosen |

Doing nothing was not an option, because `csv.encode` had to exist and accept
either shape by [R-STAR-017] (from docs/spec/starlark.md [R-STAR-017], high).
The source names no third route.

## What it costs

`csv.encode` walks Starlark values on its own path, so it is the one encoder
that does not share the JSON conversion the others use, and a fix to that
conversion does not reach it (reasoned from crates/meow-star/src/modules.rs,
low). It also has to check that every dict row has the same keys as the first,
because the header is written once; a row with different keys fails naming the
row (from crates/meow-star/src/modules.rs:392-405, high).

## What would reverse it

This is reversed if the workspace turns on `serde_json`'s `preserve_order`
feature, so that a `serde_json::Map` keeps insertion order and the JSON route
no longer reorders columns (reasoned from Cargo.toml, which does not enable
`preserve_order` today, low).

## Consequences

- A CSV parsed from a file and encoded again keeps its column order, so
  `encode(parse(text))` gives back `text` for a well-formed file (from
  crates/meow-star/tests/running.rs:1798, high).
- A field that is not a string is written the way Starlark prints it, because a
  CSV field has no type to keep (from crates/meow-star/src/modules.rs:428-434,
  high).

## How I will know it was realised

`csv_encode_round_trips_both_shapes` in `crates/meow-star/tests/running.rs`
passes: it parses `name,count` and expects `name,count` back, where the JSON
route would have written `count,name` (from
crates/meow-star/tests/running.rs:1798-1823, high).

## What this does not settle

- Key order for `json.encode`, `yaml.encode`, `toml.encode` and `xml.encode`,
  which still go through a JSON value and so sort mapping keys (from
  crates/meow-star/src/modules.rs:147, 180, 205 and 479, high).
- Whether a row with extra or missing keys should be filled in instead of
  refused; the code refuses it (from crates/meow-star/src/modules.rs:392-405,
  high).
