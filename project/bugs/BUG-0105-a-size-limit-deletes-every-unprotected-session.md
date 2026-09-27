---
id: BUG-0105
artifact: bug
status: approved
severity: major
violates: REQ-2255
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A retention size limit deletes every unprotected session

Deleting rows doesn't shrink an SQLite file, and `size_bytes` measures the
file, so a sweep with a size limit below the current size keeps deleting until
no unprotected session is left, and the file stays the same size.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. In a scratch crate depending on `meow-session`, `meow-store` and
   `meow-core` by path, open a store with `Store::open_at` in a temporary
   directory.
2. Start 50 sessions, each with one `Assistant` event of 20,000 bytes, and
   finish each.
3. Sum the sizes of `meow.db`, `meow.db-wal` and `meow.db-shm`, and sweep with
   `Retention { max_bytes: Some(size * 9 / 10), ..Retention::default() }`.

The run printed `before=5221856 limit=4699670 deleted=50 after=5221856`.

## What the system does

`crates/meow-session/src/retention.rs:80-94` deletes the oldest session, then
checks `self.store().size_bytes()` again. `Store::size_bytes`,
`crates/meow-store/src/lib.rs:127-137`, sums the file lengths, which a delete
doesn't reduce: freed pages go to SQLite's free list, and the write-ahead log
grows. Nothing in the tree runs `VACUUM` or sets `auto_vacuum`. So the loop
runs to the end of the list. BUG-0104 keeps this path unreachable today.

## What it should do, and why

REQ-2255 asks for retention by total database size, and ADR-0434 has it delete
oldest first and measure after each deletion until the database fits. A limit
10% under the current size should delete about 10% of the sessions, not all of
them.

## Triage

The defect enters in `Sessions::sweep` together with how `meow-store` measures
size. It's major and dormant: it destroys every unnamed session the moment a
size limit reaches it, which is data loss, but BUG-0104 means no user can set
one yet. Measuring live pages (`page_count - freelist_count` times
`page_size`), or reclaiming space after each delete, would fix it.

## Closed by

A test in `crates/meow-session/tests/`, named for example
`a_size_limit_deletes_only_until_the_database_fits`, running the reproduction
above and expecting fewer than half the sessions deleted.
