---
id: ADR-0620
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1826, REQ-1827]
supersedes: []
---

# 0620. A package archive that escapes is caught by two checks

## Decision

A fetch checks each archive entry twice: it refuses a name containing `..` or
starting at a root before unpacking, and it fails when `tar`'s `unpack_in`
answers that it declined to write the entry (from
https://github.com/retran/meowg1k/pull/162 and
crates/meow-cli/src/fetch.rs:184-259, high).

Once this is accepted, both checks work, and either one alone refuses an
escaping archive (from https://github.com/retran/meowg1k/pull/162, high).

## Why

The failure mode is silence: `unpack_in` answers `Ok(false)` for an entry it
declines, so with no check of meowg1k's own an escaping archive becomes an
empty package that pins and loads with nothing saying why (from
https://github.com/retran/meowg1k/pull/162, high). The author removed the
explicit check to test the test, and the fetch succeeded and pinned the hash of
an empty directory (from https://github.com/retran/meowg1k/pull/162, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: trust `tar` and ignore the boolean | Least code (from https://github.com/retran/meowg1k/pull/162, high) | An escaping archive silently pinned an empty package (from https://github.com/retran/meowg1k/pull/162, high) |
| The explicit name check alone | One check, before anything is written (from crates/meow-cli/src/fetch.rs:216-230, high) | It catches the ordinary `..` and not whatever else `tar` declines, and the failure mode is silence (from crates/meow-cli/src/fetch.rs:232-237, high) |
| The `unpack_in` boolean alone | `tar`'s own rules decide, with nothing to keep in step (reasoned from crates/meow-cli/src/fetch.rs:238, low) | Two cheap checks are better than one relied upon (from crates/meow-cli/src/fetch.rs:186-189, high) |

## What it costs

Two checks to keep, where each alone passes the test (from
https://github.com/retran/meowg1k/pull/162, high).

## What would reverse it

- `tar`'s `unpack_in` starts failing, in place of answering `false`, for an
  entry it declines (reasoned from crates/meow-cli/src/fetch.rs:232-237, low).

## Consequences

- Nothing is written outside the package, and a failed fetch leaves only
  staging behind (from https://github.com/retran/meowg1k/pull/162 and
  REQ-1818, high).

## How I will know it was realised

1. `an_archive_that_escapes_is_refused` in crates/meow-cli/tests/fetch.rs
   passes with a hand-built entry named `../../x`, and nothing is written
   outside (from crates/meow-cli/tests/fetch.rs:204-231, high).

## What this does not settle

- Symbolic and hard link entries in an archive, which neither check names
  (reasoned from crates/meow-cli/src/fetch.rs:216-221, low).
