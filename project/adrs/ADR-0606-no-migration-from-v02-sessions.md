---
id: ADR-0606
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2263]
supersedes: []
---

# 0606. v0.2.x sessions are not migrated

## Decision

The binary has no migration path from v0.2.x sessions, and `v0.2.1` stays
installable for anyone who needs to read an old one (from
docs/design/0.3.0-sessions.md section 11, high).

Once this is accepted, it holds by construction: the store opens only
`.meow/.data/meow.db`, discovery looks only for `.meow/meow.star`, and nothing
in `crates/` names `.meowg1k/` or `project.db`, where v0.2.x kept its sessions
(from crates/meow-store/src/lib.rs:63-71,
crates/meow-star/src/workspace.rs:11-12 and
v0.2.1:internal/adapters/sqlite/path/service.go:64-69, high). A v0.2.x
workspace with no `.meow/` is therefore no workspace, and its database is
never opened, read or changed (reasoned from
crates/meow-star/src/workspace.rs:34, low).

## Why

The event model, the identifier scheme and the storage layout all change, and
the v0.2.x log isn't something anyone is attached to (from
docs/design/0.3.0-sessions.md section 11, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Migrate `project.db` into `meow.db` on first open | A user's history carries over with nothing to install (reasoned from docs/design/0.3.0-sessions.md section 11, low) | Every part of the log changes shape, so a migration is a converter between two models for a history nobody asked to keep (from docs/design/0.3.0-sessions.md section 11, high) |
| Read a v0.2.x log read-only, as a newer binary reads an older 0.3 log | Old runs stay visible in `meow session show` without being rewritten (reasoned from docs/design/0.3.0-sessions.md section 11, low) | It keeps a second event model alive in the reader for the same history (reasoned from docs/design/0.3.0-sessions.md section 11, low) |

Doing nothing is the chosen option here: not migrating is the absence of work.

## What it costs

A user moving from v0.2.x loses `meow session` access to earlier runs from the
new binary, and keeps `v0.2.1` installed to read them (from
docs/design/0.3.0-sessions.md section 11, high). The old `.meowg1k/` directory
stays on disk until the user deletes it (reasoned from
crates/meow-store/src/lib.rs:63, low).

## What would reverse it

- A user reports needing a v0.2.x session in the 0.3 binary, for audit or to
  continue it (reasoned from docs/design/0.3.0-sessions.md section 11, low).

## Consequences

- `meow-store`'s migrations start at version 1 and create the schema from
  nothing (from crates/meow-store/src/migrations.rs:21-60, high).

## How I will know it was realised

1. `the_database_lives_at_one_known_path` in crates/meow-store/tests/spec.rs
   passes, and `grep -rn meowg1k/.data crates` finds nothing (from
   crates/meow-store/tests/spec.rs:26, high).

## What this does not settle

- What happens to a v0.2.x `project.db` copied by hand to
  `.meow/.data/meow.db`: its `meta` table would be read as a 0.3 schema version
  (reasoned from crates/meow-store/src/migrations.rs:175-195, low).
