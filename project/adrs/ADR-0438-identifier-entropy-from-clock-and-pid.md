---
id: ADR-0438
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2220]
supersedes: []
---

# 0438. Session identifier entropy comes from the clock and the process id

## Decision

Identifier entropy comes from the clock and the process id, not a random number
generator (from https://github.com/retran/meowg1k/pull/129, high).

Once accepted, identifiers are unique within a workspace without a random
number generator: the short form is the low 24 bits of the process id and the
low 16 bits of the clock's sub-second nanoseconds (from
crates/meow-cli/src/wire.rs:928, high). REQ-2220 was amended on 2026-09-27 to
call the tail the entropy part, not the random part. What still doesn't hold:
two sessions started by one process differ in only 16 bits of the short form,
and the identifier is guessable, which nothing depends on today (from
crates/meow-cli/src/wire.rs:922-927, high).

## Why

An identifier has to be unique within a workspace, not unguessable, and one
fewer dependency is worth more than randomness nothing depends on (from
https://github.com/retran/meowg1k/pull/129, high). The dependency saved is a
direct one for `meow-cli` only: `rand` 0.9 was already in the build through
`meow-llm`, which uses it for retry jitter, a day before this choice (from
crates/meow-llm/Cargo.toml:13, crates/meow-llm/src/retry.rs:77 and
`git log -S'rand = "0.9"'`, commit 1367bba, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A random number generator, as the design's ULID has | Unguessable identifiers, and a tail with 40 random bits whoever creates the session (from https://github.com/retran/meowg1k/pull/129, medium; docs/design/0.3.0-sessions.md section 4, high) | Nothing depends on randomness, and it costs a dependency (from https://github.com/retran/meowg1k/pull/129, high) |
| The timestamp alone, with no entropy part | The shortest identifier and no source of entropy to choose (reasoned from crates/meow-core/src/id.rs:37, low) | Two sessions created in the same millisecond would collide, and the short form is the tail precisely so that they don't (from crates/meow-core/src/id.rs:90, high) |
| A UUID, as v0.2.x used | A standard format any tool can read (reasoned from docs/design/0.3.0-sessions.md section 2, low) | It doesn't sort by creation time and nobody types one (from docs/design/0.3.0-sessions.md section 2, high) |

Doing nothing isn't an option here, because every session needs an identifier
and the Rust tree had no earlier one to keep (from
https://github.com/retran/meowg1k/pull/129, medium).

## What it costs

The short identifier carries less than its 40 bits suggest. It is the last five
bytes of the entropy, which `entropy()` fills with the low 24 bits of the
process id and the low 16 bits of the clock's sub-second nanoseconds (from
crates/meow-cli/src/wire.rs:928 and crates/meow-core/src/id.rs:37, high). Two
sessions created in one process therefore differ in 16 bits of the tail, so one
pair in 65,536 shares a short identifier (reasoned from
crates/meow-cli/src/wire.rs:936, low). The last two entropy bytes repeat bits
the first four already hold, so they add nothing (from
crates/meow-cli/src/wire.rs:936, high).

## What would reverse it

- Identifiers need to be unguessable; the pull request names this as the line to
  change then (from https://github.com/retran/meowg1k/pull/129, high).
- Two sessions in one workspace are observed to share a short identifier, for
  example when one process creates several sessions (reasoned from
  crates/meow-cli/src/wire.rs:928, low).

## Consequences

- `meow-cli` mints every identifier through one function, `entropy()`, called
  for a new run and for a fork (from crates/meow-cli/src/wire.rs:690 and
  crates/meow-cli/src/wire.rs:1213, high).
- `SessionId::new` takes the entropy as an argument, so `meow-core` does no
  input or output and a test can build an identifier from fixed bytes (from
  crates/meow-core/src/id.rs:37, high).
- The doc comment on `SessionId` still says the identifier holds "80 bits of
  randomness", which the code no longer makes true (from
  crates/meow-core/src/id.rs:18, high).

## How I will know it was realised

`the_short_form_is_the_random_tail` and `identifiers_sort_by_creation_time`,
the unit tests in `crates/meow-core/src/id.rs`, pass (from
crates/meow-core/src/id.rs:83, high). Neither exercises `entropy()`; no test
checks that two sessions minted in one process get different short identifiers
(reasoned from crates/meow-cli/src/wire.rs:928, low).

## What this does not settle

- REQ-2220 once called the last eight characters the random part, and ADR-2202
  builds on that; the coordinator's amendment of REQ-2220 now names the clock
  and the process id, and ADR-2202 and the `SessionId` doc comment still say
  random (from
  docs/requirements/REQ-2220-session-has-full-and-short-identifier.md and
  crates/meow-core/src/id.rs:18, high).
- Whether a short identifier that collides is detected: the design's "unique
  prefix, extended automatically on collision" is not what the code does,
  because the short form has a fixed length (from docs/design/0.3.0-sessions.md
  section 4 and crates/meow-core/src/id.rs:66, high).
