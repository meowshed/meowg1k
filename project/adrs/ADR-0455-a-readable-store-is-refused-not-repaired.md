---
id: ADR-0455
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1206]
supersedes: []
---

# 0455. A credential store readable by others is refused, not repaired

## Decision

The file is `0600`, and one that isn't is refused rather than repaired, with an
error naming the mode it found and the mode it wants (from
https://github.com/meowshed/meowg1k/pull/153, high).

With this in place, `Store::open` checks the mode before reading and fails with
`AuthError::TooOpen` when any group or other bit is set; the message names the
path, the mode found and `600` (from crates/meow-cli/src/auth.rs:28-36, 155 and
275-292, high). On Windows the check does nothing (from
crates/meow-cli/src/auth.rs:294-298, high).

## Why

A file that was world-readable has already been readable, and quietly tightening
the mode hides that from the person who needs to know (from
https://github.com/meowshed/meowg1k/pull/153, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Tighten the mode quietly | The user keeps working with no step to take (reasoned from crates/meow-cli/src/auth.rs:275-292, low) | It hides that the file was readable (from https://github.com/meowshed/meowg1k/pull/153, high) |
| Do nothing: read the file whatever its mode | Nothing refuses a store the user made readable on purpose (reasoned from crates/meow-cli/src/auth.rs:155, low) | [R-AUTH-011] says `meow auth` MUST refuse it; with the check replaced by `if false`, `a_store_readable_by_others_is_refused` fails (from docs/spec/auth.md [R-AUTH-011] and https://github.com/meowshed/meowg1k/pull/153, high) |

The source names no third option.

## What it costs

Windows has no mode to check, so the check is a no-op there and `R-AUTH-011` is
vacuous on Windows (from https://github.com/meowshed/meowg1k/pull/153, high). A
user whose store was made readable has to run `chmod 600` before `meow auth`
works again (reasoned from crates/meow-cli/src/auth.rs:28, low).

## What would reverse it

This is reversed if the file stops being where credentials live, as ADR-0467
proposes by moving them into the operating system's secret store with the file
as a fallback (from
docs/adrs/ADR-0467-credentials-live-in-the-os-secret-store.md, medium).

## Consequences

- `meow auth` exits with an error and does not read the file; the test sees
  exit failure and both `644` and `600` in the message (from
  crates/meow-cli/tests/auth.rs:184-205, high).
- During a run, a store that won't open is skipped and the key is looked for in
  the environment, so the refusal is reported by `meow auth` and `meow doctor`
  and doesn't stop a run that has a key elsewhere (from
  crates/meow-cli/src/wire.rs:1428-1443, high).

## How I will know it was realised

`a_store_readable_by_others_is_refused` and `the_store_is_created_private` in
`crates/meow-cli/tests/auth.rs` pass on Unix (from
crates/meow-cli/tests/auth.rs:184 and 211, high).

## What this does not settle

- What protects the file on Windows, where it inherits the user's access
  control list (from crates/meow-cli/src/auth.rs:294-298, high).
- Whether a run that skipped a refused store should say so; today the run falls
  through to the environment variable silently (from
  crates/meow-cli/src/wire.rs:1428-1443, high).
