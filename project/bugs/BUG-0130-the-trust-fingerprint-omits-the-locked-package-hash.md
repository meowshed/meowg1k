---
id: BUG-0130
artifact: bug
status: approved
severity: major
violates: REQ-1229
found: 2026-09-27
revised: 2026-09-27
issue: 253
---

# The trust fingerprint omits the locked package hash

The trust record lists each package by name, version and source, and not by the
hash `meow.lock` pins. A change to `meow.lock` that pins different contents for
the same version keeps the workspace trusted, and the new package code runs
without a question.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Serve a package archive at a URL whose contents can change, declare it with
   `meow.package`, run `meow pkg update` and `meow trust`.
2. Replace the archive at the same URL with different Starlark, run
   `meow pkg update` again, which rewrites the hash in `meow.lock`.
3. Run a command that loads the package. It runs the new code without asking.

This follows from the code below.

## What the system does

`Declared::of` writes `package <name> <version> from <source>` for each
package, `crates/meow-cli/src/trust.rs:43-49`, and `fingerprint` hashes only
those lines, `trust.rs:80-92`. The lockfile isn't read.

## What it should do, and why

REQ-1229: "A workspace whose declarations changed MUST NOT silently keep its
trust. What is recorded is what was shown". The code a package brings is the
part of the workspace the person didn't write, and its identity is the hash
REQ-1805 pins, so the hash should be among the lines shown and fingerprinted.

## Triage

The defect enters in `Declared::of`. It's major because the one line of the
trust question the code itself calls "the single most important thing on this
list" doesn't change when the package's contents do. It isn't critical,
because it needs a mutable source, and a pinned version at a reputable host
rarely changes.

## Closed by

A test in `crates/meow-cli/tests/trust.rs`, named for example
`a_changed_lock_hash_asks_again`, that trusts a workspace, edits the pinned
hash in `meow.lock`, and expects the question again.
