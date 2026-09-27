---
id: BUG-0036
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `meow.index`'s doc comment sits on `meow.package`

In `crates/meow-star/src/declare.rs` the doc comment written for `meow.index`
is followed directly by the one for `meow.package`, so both attach to `fn
package` and `fn index` has none.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Read `crates/meow-star/src/declare.rs:339-350`. "Say how this workspace is
   indexed" starts at line 339, "Declare a package this workspace loads from"
   follows at line 345 with no item between, and `fn package` is at line 350.
2. Read `crates/meow-star/src/declare.rs:370`. `fn index` has no doc comment.

## What the system does

Any generated documentation for `meow.package` opens with the text about
indexing, and `meow.index` has no documentation.

## What it should do, and why

No requirement covers doc comments. `CLAUDE.md`'s `docs_must_match_code` asks
for a stale comment to be fixed where it is found. The Starlark builtins'
doc comments are what a user reads about them, which the
`starlark_api_is_the_product` principle makes part of the product.

## Triage

The defect enters in `meow-star`'s declaration module, most likely when the two
functions were reordered. It is minor. The fix is to move the first comment
block onto `fn index`.

## Closed by

A review of `declare.rs`; no test is needed.
