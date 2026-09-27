---
id: ADR-2604
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2613, REQ-2617, REQ-2618]
supersedes: []
---

# 2604. Event payloads are content-addressed

## Decision

The store addresses every event payload by the BLAKE3 hash of its content, and
writing a hash already present stores no second copy [REQ-2613] [REQ-2617]
[REQ-2618] (from docs/spec/store.md, high).

## Why

Storing each payload inline duplicates it every time it recurs (from
docs/spec/store.md, high). A single long run re-reads the same files
constantly, and a naive log is dominated by duplicated tool output, so the
sessions design calls content addressing "not premature" (from
docs/design/0.3.0-sessions.md section 3, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: store each tool result inline, as v0.2.x does | One row per event and no reference count to keep, so deleting a session is a plain delete (reasoned from docs/spec/store.md [R-STORE-040], low) | An agent that reads the same file at three steps stores it three times (from docs/spec/store.md, Changes from v0.2.x, high) |
| Address only payloads over 512 bytes, and keep smaller ones inline in the event, as the first draft of [R-STORE-010] said | Small payloads skip the blob table and its reference count (reasoned from `git show f8e58ea:docs/spec/store.md` [R-STORE-010], low) | The event types always carry a hash, so a payload stored only sometimes by hash contradicted them; the spec was amended so every payload has a hash and inlining is an invisible storage choice (from <https://github.com/meowshed/meowg1k/pull/110>, high) |

The Go-era plan in issue 7 keyed a content store by SHA-256 of the file
content (from <https://github.com/meowshed/meowg1k/issues/7>, high). No source
says why BLAKE3 replaced SHA-256, so the table has no row for the choice of
hash.

## What it costs

- The store keeps a reference count per blob and deletes a blob only when the
  count reaches zero, so every path that adds or drops a referent has to
  count: a fork increments the count of every blob it shares, or collecting
  the origin deletes data the fork still points at (from docs/spec/store.md
  [R-STORE-012] and commit e39cd2f, high).
- Every payload is hashed on write (from crates/meow-store/src/blob.rs:51-59,
  high).

## What would reverse it

Logs that seldom repeat a payload would remove the reason, since the argument
rests on a long run re-reading the same files (reasoned from
docs/design/0.3.0-sessions.md section 3, low).

## Consequences

- `Store::put_blob` hashes the bytes with BLAKE3 and inserts with
  `ON CONFLICT(hash) DO UPDATE SET refcount = refcount + 1`, so a second write
  of the same bytes stores no copy, doesn't fail, and counts a second referent
  (from crates/meow-store/src/blob.rs:36-59, high).
- The store declines the permission [R-STORE-010] gives to keep a payload
  under 512 bytes inline, which ADR-0406 records (from
  crates/meow-store/src/blob.rs:44-50, high).
- The session log doesn't use the blob table yet. `Sessions::append` serialises
  the whole `EventKind`, a `ToolResult`'s `output` included, into the event's
  `body` and calls `Store::append`, which passes no payload hash; outside tests,
  nothing calls `put_blob` or `append_with_payload` (from
  crates/meow-session/src/log.rs:141-147, crates/meow-store/src/rows.rs:77-79
  and crates/meow-core/src/event.rs:103-111, high).

## How I will know it was realised

1. `a_payload_round_trips_by_hash_whatever_its_size` and
   `writing_the_same_payload_twice_stores_it_once` in
   crates/meow-store/tests/spec.rs pass (from
   crates/meow-store/tests/spec.rs:128-155, high).
2. A run that reads the same file at three steps leaves one row in `blobs`
   for that content and three events pointing at its hash. No test checks
   this, and the third consequence says it doesn't hold today.

## What this does not settle

- Which event fields count as the payload and go to the blob table, and which
  stay in the event body; `append_with_payload` keeps the body so a reader can
  say what happened without fetching anything (from
  crates/meow-store/src/rows.rs:81-86, high).
- Why BLAKE3 and not SHA-256, which issue 7 used (from
  <https://github.com/meowshed/meowg1k/issues/7>, high).
