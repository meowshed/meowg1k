---
id: BUG-0024
artifact: bug
status: approved
severity: major
violates: REQ-2613
found: 2026-09-27
revised: 2026-09-27
issue: 197
---

# Event payloads are stored inline, not addressed by their BLAKE3 hash

`Sessions::append` serialises the whole event into the `body` column. Nothing in
production calls `put_blob` or `append_with_payload`, so no event payload goes
through the blob table and none is addressed by its hash.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `grep -rn "put_blob\|append_with_payload" crates`. Outside the store's
   own definitions and tests, nothing calls them.
2. Run any agent, open `.meow/.data/meow.db` with `sqlite3`, and select from the
   events table. Each row's `body` holds the full event JSON and its `payload`
   column is null.

Step 2 was not run; it follows from step 1 and the code below.

## What the system does

`crates/meow-session/src/log.rs:141-147` serialises `kind` into `body` and calls
`self.store.append`, which is `append_with_payload(..., None)`
(`crates/meow-store/src/rows.rs:78`).

## What it should do, and why

REQ-2613: "Every event payload MUST be addressed by the BLAKE3 hash of its
content." ADR-2604 records the decision; its reason is that a file quoted by
many tool results is stored once.

## Triage

The defect enters in `meow-session`'s `append`, which never splits a payload
out. It is major because the requirement's main behaviour doesn't happen. Its
cost is hidden today because the relay records every `ToolResult` with an empty
`output` (`crates/meow-star/src/run.rs:727-732`), so tool output, the bulk the
blob table exists for, never reaches the log at all.

## Closed by

A test in `crates/meow-session/tests/`, named for example
`a_tool_result_payload_is_stored_once_by_hash`, that appends two events with the
same large output and expects one blob row and two events whose `payload`
names it.
