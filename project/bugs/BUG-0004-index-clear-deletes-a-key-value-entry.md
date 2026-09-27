---
id: BUG-0004
artifact: bug
status: approved
severity: minor
violates: REQ-1439
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `meow index clear` deletes the `index.model` entry from the key-value store

`Index::clear` removes the key-value entry `index.model` along with the index
rows. REQ-1439 says `clear` leaves the key-value store untouched, and the test
that cites REQ-1439 asserts the entry is gone.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `cargo test -p meow-index clear_removes_the_index_and_nothing_else`.

The test passes, and its assertion
`assert!(index.model().unwrap().is_none())` at
`crates/meow-index/tests/index.rs:465` checks that the key-value entry was
deleted.

## What the system does

`crates/meow-index/src/index.rs:149` calls `self.store.kv_delete(MODEL_KEY)`,
where `MODEL_KEY` is `"index.model"` (`crates/meow-index/src/index.rs:17`).

## What it should do, and why

REQ-1439: "`clear` MUST leave sessions and the key-value store untouched."

## Triage

The defect enters where the index keeps its own metadata in the shared
key-value store. The code is likely right and the requirement too broad: the
recorded model belongs to the index, and keeping it after a clear would make
the next build's model check compare against vectors that no longer exist. The
fix is probably to amend REQ-1439 to exclude the index's own entry, or to move
the model name into an index table. It is minor because no user value in the
key-value store is lost.

## Closed by

Whichever way it is resolved, `clear_removes_the_index_and_nothing_else` in
`crates/meow-index/tests/index.rs` asserting what the amended REQ-1439 says.
