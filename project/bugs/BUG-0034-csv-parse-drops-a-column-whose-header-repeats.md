---
id: BUG-0034
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `csv.parse` silently drops a column whose header name repeats

With a header row that names a column twice, `csv.parse` builds each row as a
dictionary, and the later column overwrites the earlier one. `a,a\n1,2` parses
to `[{"a": "2"}]`, and the `1` is gone without an error.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Create a workspace whose command handler writes
   `repr(parse("a,a\n1,2"))`, with `parse` loaded from `@std//csv`.
2. Run `meow trust`, then the command.

It prints `csv=[{"a": "2"}]`. This was run against `target/debug/meow`.

## What the system does

`crates/meow-star/src/modules.rs:311-360` zips the header names with each
record's fields into an `AllocDict`, so a repeated name keeps the last field.
The function also takes a `header` argument (line 313), which no requirement
describes.

## What it should do, and why

No requirement covers a repeated header name. REQ-2434 says `csv.parse` returns
a list of dictionaries when the first record names the columns, which can't
hold two columns of one name. The module already refuses a record whose length
differs from the header's, so refusing a repeated name, with the column
numbers, fits what it does. This is a gap in the requirements and routes to the
requirements step, which should also describe the `header` argument.

## Triage

The defect enters in `meow-star`'s `csv` module. It is minor: it needs a file
with a repeated header, but data is lost silently when it happens.

## Closed by

A test in `crates/meow-star/tests/running.rs`, named for example
`csv_parse_refuses_a_repeated_header`, that expects an error naming the
repeated column for the input above.
