---
id: BUG-0025
artifact: bug
status: approved
severity: minor
violates: REQ-2604
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `meow doctor` doesn't report that the database is unencrypted, or its path

`meow doctor` prints the workspace, the configuration directory, the provider
kinds and the credentials. It says nothing about the database.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `meow doctor` in any workspace.

No line names `.meow/.data/meow.db` or says it is unencrypted.

## What the system does

`crates/meow-cli/src/wire.rs:485-525` prints `workspace`, `config`, `kinds` and
`credentials` lines and returns.

## What it should do, and why

REQ-2604: "`meow doctor` MUST report that the database is unencrypted, together
with its path, so that a user deciding what a workspace may hold is told rather
than left to assume."

## Triage

The defect enters in `meow-cli`'s `doctor`. It is minor: a missing line of
output, with no effect on what is stored.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`doctor_names_the_database_and_says_it_is_unencrypted`, that runs `meow doctor`
and expects a line holding the database path and the word `unencrypted`.
