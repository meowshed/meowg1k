---
id: ADR-0406
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2614, REQ-2615, REQ-2616]
supersedes: []
---

# 0406. The store keeps every payload in the `blobs` table and never inline

## Decision

`R-STORE-010` permits inlining a payload under 512 bytes, and this store
declines the permission (from https://github.com/retran/meowg1k/pull/113, high).

Once this is accepted, `Store::put_blob` writes every payload, whatever its
size, to the `blobs` table, and `INLINE_LIMIT` stays only as the documented
boundary (from crates/meow-store/src/blob.rs:36-58 and
crates/meow-store/src/lib.rs:41-44, high). What still doesn't work is the
session log's use of it: every event kind carries its content as a `String`,
`Sessions::append` writes it into `events.body` with no payload, and no
production code calls `put_blob`, so in practice every event's content is
stored inline in the event row (from crates/meow-core/src/event.rs:82-107,
crates/meow-session/src/log.rs:141-147 and crates/meow-store/src/rows.rs:74-79,
medium).

## Why

A `BLOB` column already stores a small value in the row SQLite writes anyway, so
the second path buys nothing and costs a branch on every read (from
https://github.com/retran/meowg1k/pull/113, high). The requirement asks that the
choice be invisible, which it is either way (from
https://github.com/retran/meowg1k/pull/113, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Keep a payload under 512 bytes inline | A small payload is read with its event row, with no second lookup in `blobs` (reasoned from crates/meow-store/src/blob.rs:60-76, low) | It buys nothing, because a `BLOB` column already stores a small value in the row, and it costs a branch on every read (from https://github.com/retran/meowg1k/pull/113, high) |
| Keep every payload in the event row and have no blob table | One read per event and no reference counts to keep (reasoned from crates/meow-store/src/session.rs:144-180, low) | An agent that reads the same 40 KB file at steps 3, 7 and 11 would store it three times, and a naive log is dominated by duplicated tool output (from docs/design/0.3.0-sessions.md, lines 78-81, high) |

Doing nothing isn't an option here: `R-STORE-010` requires content addressing
and only leaves the inline path to the implementation, so the question is only
which of the two permitted layouts to use (from
docs/requirements/REQ-2613-payload-addressed-by-blake3.md and
docs/requirements/REQ-2614-small-payload-may-be-inline.md, medium).

## What it costs

A payload of a few bytes pays for a row in `blobs` and a reference count, which
the inline path would have saved (reasoned from
crates/meow-store/src/blob.rs:48-58, low). Every referent has to maintain that
count, so a fork adds a referent to each blob it copies in the same
transaction as the copy (from crates/meow-store/src/session.rs:128-180, high).

## What would reverse it

- Reading payloads through `blobs` shows up as a measured cost on the rebuild
  path, which a small value kept in the event row would avoid (reasoned from
  https://github.com/retran/meowg1k/pull/113, low).

## Consequences

- `data` holds the bytes whether a payload is small or large, so the
  invisibility the requirement asks for is structural rather than a rule to
  remember (from crates/meow-store/src/migrations.rs:30-38, high).
- A missing hash is an error naming the hash, never empty content (from
  crates/meow-store/src/blob.rs:60-76, high).

## How I will know it was realised

1. `a_payload_round_trips_by_hash_whatever_its_size` in
   crates/meow-store/tests/spec.rs passes: a 16-byte payload and a 40,000-byte
   payload both come back by hash (from crates/meow-store/tests/spec.rs:128-139,
   high).
2. Once the session log stores event content through `put_blob`, a test that
   appends the same tool output twice finds one row in `blobs` with a
   reference count of two; no such test exists yet (reasoned from
   crates/meow-store/tests/spec.rs:142-155, low).

## What this does not settle

- Which parts of an event are payloads. The design named `BlobRef` fields for
  messages, tool arguments, tool output and summaries, and the code stores all
  of them as strings in the event body (from docs/design/0.3.0-sessions.md,
  lines 51-57, and crates/meow-core/src/event.rs:82-107, medium).
