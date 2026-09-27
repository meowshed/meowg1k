---
id: ADR-0448
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2432, REQ-2433]
supersedes: []
---

# 0448. `yaml.encode` and `toml.encode` convert to the target's own value type

## Decision

`yaml.encode` and `toml.encode` convert to the target's own value type
explicitly rather than reusing the JSON value (from
https://github.com/retran/meowg1k/pull/141, high).

What works now: both encoders turn the Starlark value into a `serde_json`
value, rebuild it node by node as a `serde_yaml_ng::Value` or a `toml::Value`
through `as_yaml` and `as_toml`, and serialise that, so a `3` comes out as `3`
(from crates/meow-star/src/modules.rs:228 and
crates/meow-star/src/modules.rs:255, high). What still doesn't: a number that
doesn't fit `i64` becomes an `f64` and may lose digits, and the keys of a
mapping come out sorted, because the intermediate `serde_json::Map` sorts them
(from crates/meow-star/src/modules.rs:298 and
https://github.com/retran/meowg1k/pull/141, high).

## Why

`starlark` turns on `serde_json`'s `arbitrary_precision`, so every number is
stored as text behind a marker only `serde_json`'s own serialiser understands,
and a `3` handed to `toml::to_string` came out as a table named
`$serde_json::private::Number` (from https://github.com/retran/meowg1k/pull/141,
high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: reuse the JSON value for every encoder | One path for every format and no conversion code to keep (reasoned from crates/meow-star/src/modules.rs:142, low) | Numbers don't survive a foreign serialiser (from https://github.com/retran/meowg1k/pull/141, high) |
| Turn off `arbitrary_precision` | The JSON value would serialise correctly through any serde format (reasoned from crates/meow-star/src/modules.rs:222, low) | Features are additive across a workspace and `starlark` turns it on, so no crate here can turn it off (from https://github.com/retran/meowg1k/pull/141, high) |
| Read the Starlark values directly, as `csv.encode` does | Keeps a dict's insertion order, which the JSON detour sorts away (from https://github.com/retran/meowg1k/pull/141, high) | The source doesn't weigh it for YAML and TOML; it was chosen for CSV only because column order is the output there (from https://github.com/retran/meowg1k/pull/141, high) |

## What it costs

Two conversion functions, one per format, that have to follow every variant
of the JSON value (from crates/meow-star/src/modules.rs:228 and
crates/meow-star/src/modules.rs:255, high). An unsigned number above `i64::MAX`
or a fraction is written as `f64`, which isn't exact (from
crates/meow-star/src/modules.rs:298, high).

## What would reverse it

`starlark` no longer turning on `serde_json`'s `arbitrary_precision`, visible in
`cargo tree -e features -i serde_json`, would let the JSON value go straight to
`serde_yaml_ng` and `toml` (reasoned from
https://github.com/retran/meowg1k/pull/141, low).

## Consequences

- `toml.encode(yaml.parse("name: meow\ncount: 3\n"))` produces
  `count = 3\nname = "meow"\n`, with the integer intact and the keys sorted
  (from crates/meow-star/tests/running.rs:1644, high).
- A null reaches `as_toml` as a JSON null and is refused there, which is where
  ADR-0447's refusal lives (from crates/meow-star/src/modules.rs:260, high).

## How I will know it was realised

`crates/meow-star/tests/running.rs` `a_document_read_as_yaml_writes_as_toml`
passes, expecting `count = 3` rather than a `$serde_json::private::Number`
table, and `yaml_toml_and_json_agree_about_the_same_data` passes for the parse
side (from crates/meow-star/tests/running.rs:1644 and
crates/meow-star/tests/running.rs:1603, high).

## What this does not settle

- Whether `yaml.encode` and `toml.encode` should keep a dict's insertion order,
  as `csv.encode` does; today they sort keys (reasoned from
  crates/meow-star/tests/running.rs:1644, medium).
- How to encode an integer larger than `i64` without losing digits (from
  crates/meow-star/src/modules.rs:298, high).
