---
id: SPC-2600
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-2600, REQ-2601, REQ-2602, REQ-2603, REQ-2604, REQ-2605, REQ-2606, REQ-2607, REQ-2608, REQ-2609, REQ-2610, REQ-2611, REQ-2612, REQ-2613, REQ-2614, REQ-2615, REQ-2616, REQ-2617, REQ-2618, REQ-2619, REQ-2620, REQ-2621, REQ-2622, REQ-2623, REQ-2624, REQ-2625, REQ-2626, REQ-2627, REQ-2628, REQ-2629, REQ-2630, REQ-2631, REQ-2632, REQ-2633, REQ-2634, REQ-2635, REQ-2636, REQ-2637, REQ-2638, REQ-2639, REQ-2640, REQ-2641, REQ-2642, REQ-2643]
---

# The workspace store

## Scope

`meow-store` owns the one SQLite database a workspace keeps. It persists the
session event log, the content-addressed blobs those events point at, the
workspace key-value store, the response cache and the retrieval index (from
docs/spec/store.md, high).

It doesn't interpret what it stores. Deciding that a `Compaction` event
supersedes a range is `meow-session`'s job; the store only guarantees that the
event is written, ordered and still there later (from docs/spec/store.md, high).

## Boundary

| Surface | What it is |
| --- | --- |
| `.meow/.data/meow.db` | The single database file, resolved from the workspace root [REQ-2600] |
| WAL and shared-memory files | The side files SQLite manages beside the database, owner-only like it [REQ-2605] |
| `meow doctor` | Reports that the database is unencrypted, with its path [REQ-2604] |

(from docs/spec/store.md, high)

## Behaviour

### Database and schema

The store keeps all of one workspace's data in `.meow/.data/meow.db` [REQ-2600].
Every connection runs with write-ahead logging and foreign key enforcement
[REQ-2601], with `synchronous` at `NORMAL` [REQ-2602]. The database is plaintext
[REQ-2603], `meow doctor` says so and gives the path [REQ-2604], and the store
creates the database and its side files readable and writable by their owner
only, on every platform with file permissions [REQ-2605].

The store records the schema version in a `meta` table [REQ-2606]. On open it
applies every migration newer than that version in ascending order, each in its
own transaction [REQ-2607]. Migrations are forward-only [REQ-2609] and the store
has no downgrade path [REQ-2610].

### Blobs

The store addresses every event payload by the BLAKE3 hash of its content
[REQ-2613]. It can keep a payload under 512 bytes inline in place of the `blobs`
table [REQ-2614]; a reader can't tell which [REQ-2615], because the same hash
returns the same bytes either way [REQ-2616]. Writing a blob whose hash is
present stores no second copy [REQ-2617] and succeeds [REQ-2618]. The store
keeps a reference count per blob [REQ-2619] and deletes a blob only when its
count reaches zero [REQ-2620].

### Writing

The store commits each event on its own [REQ-2623] and never holds one
transaction open across a turn's tool execution [REQ-2624]. It allows one writer
at a time [REQ-2627] and readers alongside that writer [REQ-2628]. A bulk write
such as an index build commits in bounded batches [REQ-2629] and doesn't hold
the write lock for the length of the operation [REQ-2630].

### Key-value store

The store provides a durable key-value table scoped to the workspace, with
`get`, `put`, `delete` and `keys` [REQ-2632], kept apart from session state so
that deleting a session leaves it intact [REQ-2633].

### Retention

The store deletes a session by removing its rows and decrementing the reference
count of every blob those rows referenced [REQ-2634], and deletes the whole
session or nothing [REQ-2635]. It reports the database's total on-disk size for
retention to act on [REQ-2636].

### Cache

The store provides a cache keyed by a hash of the request that produced the
entry [REQ-2637]. It caches embedding responses by default [REQ-2638] and a
generation response only when the caller asks [REQ-2639]. Each entry records its
model [REQ-2640], and a lookup for another model misses [REQ-2641]. The store
evicts entries by total size and age [REQ-2642], and eviction leaves every
session that quoted an entry unaffected [REQ-2643].

## Failure paths

| Condition | What happens |
| --- | --- |
| A migration fails | The database stays at the version it had before that migration [REQ-2608] |
| The recorded schema version is newer than the binary supports | Open fails with an error naming both versions [REQ-2611] and the file is left unmodified [REQ-2612] |
| A read names a hash with no row | It fails with an error naming the hash [REQ-2621] and never returns empty content [REQ-2622] |
| A write fails | The error reaches the caller [REQ-2625]; the store never logs it and continues [REQ-2626] |
| A write blocks on another writer | It waits up to the configured busy timeout, at least 5 seconds, then fails [REQ-2631] |
| A session deletion can't complete | The deletion fails and no part of the session is removed [REQ-2635] |
