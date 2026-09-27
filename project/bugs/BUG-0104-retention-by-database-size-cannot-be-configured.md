---
id: BUG-0104
artifact: bug
status: approved
severity: major
violates: REQ-2255
found: 2026-09-27
revised: 2026-09-27
issue:
---

# Retention by total database size can't be configured

`meow session gc` takes an age and a count and always passes `max_bytes: None`,
and no declaration sets a size limit, so the size axis of retention is
unreachable.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `meow session gc --help`. It offers `--older-than-days`, `--keep` and
   `--named`, and no size option.
2. Search `.meow/meow.star`'s declarations for a retention setting: `meow.index`
   takes `max_bytes` for chunking, and no call takes a session size limit.

## What the system does

`crates/meow-cli/src/wire.rs:731-737` builds `meow_session::Retention` with
`max_bytes: None` on every call. `Sessions::sweep` handles a size limit at
`crates/meow-session/src/retention.rs:80-94`, and nothing outside it sets one.

## What it should do, and why

REQ-2255: "Retention MUST be configurable by age, by count, and by total
database size." A person should be able to state a size limit, on the command
line or in a declaration, and have `meow session gc` apply it.

## Triage

The defect enters at the command line in `meow-cli`, which exposes two of the
three limits. It's major because one of the requirement's three limits doesn't
exist for a user. Exposing it as it stands would also expose BUG-0105, where a
size limit deletes every unprotected session, so the two fixes belong
together.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`session_gc_takes_a_size_limit`, that records several sessions, runs
`meow session gc` with a size limit below the database size, and expects the
oldest sessions gone and the database at or under the limit.
