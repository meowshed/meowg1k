---
id: BUG-0003
artifact: bug
status: approved
severity: minor
violates: REQ-1420
found: 2026-09-27
revised: 2026-09-27
issue: 176
---

# A chunk too large reports the chunker's limit as the model's

`IndexError::ChunkTooLarge` says "the model takes N", and N is the chunker's
`max_chars`, 8000 by default. The provider's limit is never read, so the number
the user is told to fit under is one the chunk already fits under.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Give the embedder a recorded provider that answers 413 to any text over 1000
   characters.
2. Index a file with one 3000-character chunk and run `index.embed`.

The error reads `<path>:<first>-<last> is 3000 characters, and the model takes
8000`.

## What the system does

`crates/meow-index/src/index.rs:265-271` builds `ChunkTooLarge` with
`limit: self.chunking.max_chars`. The chunker already caps every chunk at
`max_chars` (`crates/meow-index/src/chunk.rs:131`), so a chunk a provider
refuses is always at or under the number the message calls the model's limit.
The message format is in `crates/meow-index/src/error.rs:61`.

## What it should do, and why

REQ-1420 asks for an error naming the file and the line range, "not a generic
size error". An error that names the wrong limit tells the user to change
nothing, which is worse than naming none. ADR-1404 records the decision.

## Triage

The defect enters in `Index::embed`. It is minor because the file and lines are
right and only the number misleads. The fix is to report the provider's limit
where the provider states one and otherwise to say the provider refused the
size without a number.

## Closed by

A test in `crates/meow-index/tests/index.rs`, named for example
`a_refused_chunk_does_not_claim_the_chunker_limit_is_the_models`, that expects
the error to omit or correct the limit when the provider refused a chunk under
`max_chars`.
