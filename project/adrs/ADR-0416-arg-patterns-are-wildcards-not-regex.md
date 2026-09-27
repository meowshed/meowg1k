---
id: ADR-0416
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2507]
supersedes: []
---

# 0416. `meow.arg.string(pattern = ...)` accepts `*` and literals, not regular expressions

## Decision

The pattern language `meow.arg.string(pattern = ...)` accepts is `*` and
literals, not regular expressions (from
https://github.com/meowshed/meowg1k/pull/121, high). The whole value has to match:
`*` matches any run of characters and every other character is literal (from
crates/meow-star/src/schema.rs:216-240, high).

Once this is accepted, a string argument declared with a pattern is refused
with "must match `<pattern>`" when a value doesn't fit the anchored shape (from
crates/meow-star/src/schema.rs:135-141, high). What doesn't work is any
character class, alternation or repetition count, because the language has
none of them (from crates/meow-star/src/schema.rs:222-240, high).

## Why

A full engine would be another dependency and another way for a crafted input
to make a declaration file run for a long time (from
https://github.com/meowshed/meowg1k/pull/121, high). What a tool argument needs is
an anchored shape, which `*` and literals express (from
crates/meow-star/src/schema.rs:216-221, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: accept `pattern` and check nothing | No matcher to write or keep, and the model still sees the pattern in the schema (reasoned from crates/meow-star/src/declare.rs:436-440, low) | REQ-2507 requires a `string` argument to accept `pattern`, and a constraint that is never checked lets a value the declaration forbids reach the handler (reasoned from crates/meow-star/src/args.rs:196, low) |
| Regular expressions | The pattern means what JSON Schema's `pattern` keyword means, which is the form the argument is sent to the model in, and a user can say more with it (reasoned from crates/meow-star/src/run.rs:392-401, low) | Another dependency, and another way for a crafted input to make a declaration file run for a long time (from https://github.com/meowshed/meowg1k/pull/121, high) |

The question admits these two alternatives besides the choice: the pattern is
either unchecked, checked as a regular expression or checked in a smaller
language.

## What it costs

A user can't write a character class or an alternation, so `v[0-9]*` is a
literal `v[0-9]` followed by anything (from
crates/meow-star/src/schema.rs:222-240, high). The argument's JSON schema
carries the pattern under the `pattern` keyword and goes to the model as the
tool's schema (from crates/meow-star/tests/arguments.rs:64-78 and
crates/meow-star/src/run.rs:392-401, high). JSON Schema defines that keyword as
a regular expression, so a model can read `v*` as "any run of `v`" while
meowg1k checks it as "starts with `v`" (reasoned from the same lines, low).

## What would reverse it

- A user needs a character class or an alternation in an argument pattern and
  the run time of the `regex` crate on a crafted input is shown to be bounded.
  The other reason has already gone: since pull request 140, `meow-star`
  depends on `regex` directly for `@std//re`, and that crate was already in the
  lock file, so a regular expression engine is no longer another dependency
  (from crates/meow-star/Cargo.toml and
  https://github.com/meowshed/meowg1k/pull/140, high).

## Consequences

- `matches_pattern` in crates/meow-star/src/schema.rs is a hand-written matcher
  of about twenty lines that the crate owns (from
  crates/meow-star/src/schema.rs:222-240, high).
- The pattern is stored in the argument's schema node as given, so the flag,
  the help line and the model's schema carry the same text (from
  crates/meow-star/src/declare.rs:436-440 and
  https://github.com/meowshed/meowg1k/pull/121, high).

## How I will know it was realised

1. `every_argument_type_builds_with_its_constraints` in
   crates/meow-star/tests/arguments.rs passes, showing `pattern = "v*"` lands in
   the schema.
2. No test checks a value against a pattern: a test calling a tool with
   `pattern = "v*"` and the value `x` should fail with "must match `v*`", and
   none exists yet (from `grep -rn "must match" crates/*/tests`, which finds
   nothing, high).

## What this does not settle

- Whether the schema a model receives should carry the wildcard under JSON
  Schema's `pattern` keyword, whose meaning is a regular expression, or leave
  it out (reasoned from crates/meow-star/src/run.rs:392-401, low).
- Whether `@std//re` and argument patterns should share one language now that
  both exist (reasoned from https://github.com/meowshed/meowg1k/pull/140, low).
