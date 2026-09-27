---
id: BUG-0200
artifact: bug
status: approved
severity: major
violates: REQ-1230
found: 2026-09-27
revised: 2026-09-27
issue: 166
---

<!-- Written to the writing standard meow-prose ships: lead with the answer,
give each rule its reason in the same sentence, and show the failing case. -->

# Credentials are stored in a plaintext file, not the platform secret store

## Reproduction

At `d67ae50`, on any platform:

1. Run `meow auth login` for a provider that takes a key, and enter a key.
2. Run `cat ~/.meow/auth.json`.

The key is in the file in plain text (from
https://github.com/meowshed/meowg1k/issues/166, high).

## What the system does

`~/.meow/auth.json` holds API keys and OAuth refresh tokens as plaintext JSON.
It is created `0600`, refused if its mode is wider, and written atomically, and
no command prints from it, but the secret is still a file on disk that meowg1k
owns (from https://github.com/meowshed/meowg1k/issues/166, high). No keychain code
exists: `crates/meow-cli/src/auth.rs` is file-only (from
crates/meow-cli/src/auth.rs, high).

## What it should do, and why

REQ-1230: credentials MUST be kept in the platform secret store (Keychain on
macOS, Credential Manager on Windows, Secret Service on Linux), with REQ-1231 to
REQ-1233 making `~/.meow/auth.json` the announced fallback. The vision's first
quality goal, security by design, says meowg1k persists no secret of its own.

## Triage

It enters in the credential store the rewrite introduced in pull request 153 and
extended in pull request 155. Major, not critical: the file is owner-only and
never printed, so a secret leaks only to something that can already read the
user's files, but the goal ranked first in the vision is not met.

## Closed by

A test that logs in and finds no credential in `~/.meow/auth.json` on a machine
with a secret service, and a test that removes the service and finds the
fallback announced (REQ-1233). Issue #166 closes with the fix.
