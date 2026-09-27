---
id: BUG-0047
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue: 220
---

# An argument's glob `pattern` reaches the model as a JSON Schema regular expression

`meow.arg.string(pattern = "src/*.rs")` is a glob, where `*` matches any run of
characters, and the declaration puts it unchanged into the tool's JSON Schema
under `pattern`. JSON Schema defines `pattern` as a regular expression, so a
model or a provider that validates schemas reads `src/*.rs` as "`src`, any
number of `/`, one character, `rs`".

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `cargo test -p meow-star --test arguments
   every_argument_type_builds_with_its_constraints`. It passes, asserting
   `properties["title"]["pattern"] == "v*"` at
   `crates/meow-star/tests/arguments.rs:79`.

`v*` as a regular expression matches every string, including `x`, while the
glob `v*` matches only strings starting with `v`. This was reasoned from the
test and not run against a provider.

## What the system does

`crates/meow-star/src/declare.rs:437-441` writes the declared text into
`"pattern"`. The runtime check at `crates/meow-star/src/schema.rs:135-141` and
`crates/meow-star/src/schema.rs:216-240` applies glob semantics, so the
argument check and the model's reading of the schema differ. No test checks a
value against a pattern; the one above checks only the schema's text.

## What it should do, and why

No requirement covers how a pattern is sent to the model. REQ-2507 says a
string argument accepts `pattern`, and ADR-0416 fixes the language as `*` and
literals. A schema should say what the check enforces, so the model's guess and
the runtime agree. This is a gap in the requirements and routes to the
requirements step. ADR-0416 also argues against a regex engine as a dependency,
and `meow-star` already depends on `regex` (`crates/meow-star/Cargo.toml:21`),
so that part of its reason no longer holds.

## Triage

The defect enters in `meow-star`'s `meow.arg.string`. It is minor: a mismatch
costs a correction round trip, because the runtime check still refuses a bad
value. The fix is to translate the glob into an anchored regular expression for
the schema, or to omit `pattern` from the schema and state it in the
description.

## Closed by

A test in `crates/meow-star/tests/arguments.rs`, named for example
`a_glob_pattern_is_sent_as_an_equivalent_regex`, and one named
`a_value_outside_the_pattern_is_corrected`, which drives the runtime check.
