---
id: ADR-2600
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2603, REQ-2604, REQ-2605]
supersedes: []
---

# 2600. The database is plaintext, with owner-only permissions

## Decision

The store doesn't encrypt the database, `meow doctor` reports that and the path,
and the database and its side files are readable and writable by their owner
only [REQ-2603] [REQ-2604] [REQ-2605] (from docs/spec/store.md, high).

## Why

Encrypting the database would need SQLCipher, a C dependency the workspace
otherwise avoids, and a key management story nobody has designed. File
permissions cost one flag and stop every other account on the machine from
reading a transcript (from docs/spec/store.md, high). The "C dependency"
reason is narrower than it reads: `rusqlite` is built with `bundled`, which
already compiles SQLite from C, so what SQLCipher adds is its cryptography
library and the build it needs (reasoned from crates/meow-store/Cargo.toml:15,
low).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Encrypt the database with SQLCipher | Covers tool output that holds a secret the policy didn't catch, which file permissions leave readable to anyone who gets the file (from `git show f8e58ea:docs/spec/store.md`, Open questions, high) | A C dependency the workspace otherwise avoids, and a key management story nobody has designed (from docs/spec/store.md, Decisions, high) |
| Plaintext, and say so, with no permission change | Nothing to write or test: the file keeps the mode the process umask gives it (reasoned from `git show f8e58ea:docs/spec/store.md`, low) | It accepted the risk without doing the cheap part: file permissions cost one flag (from docs/spec/store.md, Decisions, high) |
| Do nothing: plaintext and say nothing, as v0.2.x's separate SQLite files did | No `meow doctor` line and no permission code to keep (reasoned from docs/spec/store.md, Changes from v0.2.x, low) | A user deciding what a workspace may hold is left to assume instead of told, which [REQ-2604] exists to prevent (from docs/spec/store.md [R-STORE-007], high) |

## What it costs

File permissions aren't encryption (from docs/spec/store.md, high). Tool output
that holds a secret the policy didn't catch sits in the file in the clear, so
anyone who reads the file as its owner, or copies it off the machine, reads the
secret (from `git show f8e58ea:docs/spec/store.md`, Open questions, high).

## What would reverse it

A user asking for encryption at rest reopens the question: the first draft's
recommendation was to leave encryption out of v0.3.0 and "revisit when someone
asks" (from `git show f8e58ea:docs/spec/store.md`, Open questions, high). A
key management design, which the source names as missing, would remove one of
the two reasons the SQLCipher option lost (reasoned from docs/spec/store.md,
Decisions, low).

## Consequences

The honest place to keep a secret out of the log is still the policy (from
docs/spec/store.md, high). On Unix, `Store::open_at` sets mode `0o600` on
`meow.db`, `meow.db-wal` and `meow.db-shm` after migrating, and on Windows it
sets nothing because the user profile's access control list already restricts
the file (from crates/meow-store/src/lib.rs:152-178, high). `Store` exposes
`is_encrypted()`, which always returns `false`, and `path()` for `meow doctor`
to report (from crates/meow-store/src/lib.rs:114-121, high).

## How I will know it was realised

1. `the_database_is_readable_by_its_owner_only` in
   crates/meow-store/tests/spec.rs passes on Unix: the database file's mode is
   `0o600` (from crates/meow-store/tests/spec.rs:113-126, high).
2. `the_database_reports_that_it_is_unencrypted` in the same file passes (from
   crates/meow-store/tests/spec.rs:106-111, high).
3. `meow doctor` prints a line saying the database is unencrypted, with its
   path. It doesn't yet: `doctor` in crates/meow-cli/src/wire.rs:485-525 prints
   the workspace, the config directory, the provider kinds and the credentials,
   and nothing about the database (from crates/meow-cli/src/wire.rs, high).

## What this does not settle

- The window between SQLite creating the file and `restrict_permissions`
  narrowing it: SQLite creates the file with its default mode less the umask,
  and the store narrows it after the migration, where [REQ-2605] says
  "created" owner-only (from crates/meow-store/src/lib.rs:75-92, high).
- The mode of `.meow/.data/` itself, and of the HNSW graph files
  `meow-index` writes beside the database, which `restrict_permissions` doesn't
  touch (from crates/meow-index/src/index.rs:404-407 and
  crates/meow-store/src/lib.rs:157-170, high).
- Where credentials live: that is ADR-0467, which moves them to the operating
  system's secret store (from docs/adrs/ADR-0467, high).
- Redacting a secret a handler put in a prompt, which is export's job (from
  <https://github.com/meowshed/meowg1k/issues/166>, What is not in scope, high).
