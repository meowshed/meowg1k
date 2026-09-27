---
id: BUG-0129
artifact: bug
status: approved
severity: minor
violates: REQ-1207
found: 2026-09-27
revised: 2026-09-27
issue: 252
---

# Two processes writing the credential store share one temporary file

`write_json` writes to the fixed name `<target>.new` with no lock, so two
processes writing `auth.json` or `trust.json` at once can rename a file the
other is still writing, and each loses the other's change.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

The race needs two writers in step, so the steps name the interleaving:

1. Process A reads `auth.json`, adds `anthropic`, opens `auth.json.new` with
   truncate, writes, and calls `sync_all`.
2. Process B reads the old `auth.json`, adds `openai`, and opens
   `auth.json.new` with truncate, which empties A's file.
3. A renames `auth.json.new` over `auth.json` while B is writing into it; for a
   moment `auth.json` is empty or partial.
4. B's rename fails with "No such file or directory", and `auth.json` ends with
   B's content only, without A's `anthropic` key.

This follows from the code below; it wasn't run.

## What the system does

`write_json` at `crates/meow-cli/src/auth.rs:234-259` uses
`path.with_extension("json.new")` and renames it, and `write_private` at
`auth.rs:300-320` opens it with `truncate(true)`. No lock surrounds the read
and the write, and the directory isn't synced after the rename.

## What it should do, and why

REQ-1207: "Writing the store MUST be atomic." REQ-1208: a write "MUST NOT
leave a truncated file if the process dies". Each writer should use its own
temporary name, hold a lock across read, modify and write, and sync the
directory after the rename.

## Triage

The defect enters in `meow-cli`'s `write_json`, shared by the credential store
and the trust record. It's minor because it needs two `meow auth` or
`meow trust` writes at the same instant, which a person rarely makes; when it
happens, a credential is lost without a message.

## Closed by

A test in `crates/meow-cli/tests/auth.rs`, named for example
`concurrent_logins_keep_both_credentials`, that stores two credentials from two
threads a hundred times and expects both each time.
