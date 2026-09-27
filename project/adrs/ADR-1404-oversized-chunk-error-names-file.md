---
id: ADR-1404
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1420]
supersedes: []
---

# 1404. An oversized chunk's error names the file and the line range

## Decision

A single chunk that a provider rejects as too large fails with an error naming
the file and the chunk's line range (from docs/spec/index.md [R-INDEX-021],
high). The batch splitter halves a refused batch down to one text and returns
that text's position, and the index turns the position into
`IndexError::ChunkTooLarge` with the path and the first and last lines (from
crates/meow-index/src/embed.rs:51-94 and crates/meow-index/src/index.rs:260-275,
high).

Once this is accepted, `meow index build` stops at the refused chunk with a
message of the form `src/big.rs:2-2 is 1200 characters, and the model takes
8000`, and the vectors already stored stay stored (from
crates/meow-index/src/error.rs:55-73 and crates/meow-index/src/index.rs:236-290,
high). What still doesn't work is a refusal the binary doesn't recognise as a
size refusal: only an HTTP 413 reaches the splitter as `Rejected::TooLarge`,
so a provider that refuses an oversized input with another status fails the
build with the provider's own message and no file name (from
crates/meow-cli/src/index.rs:53-64, high).

## Why

v0.2.x's `llmEmbed` gives a generic failure message for a single oversized chunk
(from docs/spec/index.md, high). A caller told "input too large" and nothing
else has to bisect their own repository to find out where (from
crates/meow-index/src/error.rs:58-60 and
https://github.com/meowshed/meowg1k/pull/131, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: a generic size error, as v0.2.x's `llmEmbed` gives | No mapping from a batch position back to a chunk row (reasoned from crates/meow-index/src/index.rs:260-275, low) | It doesn't name the file, so the caller has to bisect the repository (from docs/spec/index.md and crates/meow-index/src/error.rs:58-60, high) |
| Trust the chunker's character limit to keep every chunk small enough, so no chunk is ever refused | No refusal path at all (reasoned from crates/meow-index/src/chunk.rs:20-26, low) | The limit is a stand-in of four characters to a token, and the provider's own limit is the authority (from https://github.com/meowshed/meowg1k/pull/130, high) |

The history names no third option; these two are all that the specification,
the code and the pull requests record.

## What it costs

Finding the offender costs extra requests: a refused batch of 64 is halved
until the one refused text is alone, so up to about 2 x log2(64) = 12 more
requests before the error (reasoned from crates/meow-index/src/embed.rs:47-94,
low). The error's `limit` is the chunker's configured `max_chars`, not the
provider's limit, so the message can say a chunk under that number was too
large (from crates/meow-index/src/index.rs:265-271, high).

## What would reverse it

- A provider that reports which input in a batch was too large, which would
  make the halving unnecessary though not the naming (reasoned from
  crates/meow-index/src/embed.rs:51-55, low).

## Consequences

`meow-index` has its own `Rejected` type with a `TooLarge` case, and the
binary's embedder has to translate a provider error into it (from
crates/meow-index/src/embed.rs:24-34 and crates/meow-cli/src/index.rs:53-64,
high). A refused chunk stops the build, and a later build resumes from the
chunks that still have no vector (from crates/meow-index/src/index.rs:236-290,
high).

## How I will know it was realised

1. `an_oversized_chunk_names_the_file_and_the_lines` in
   crates/meow-index/tests/index.rs passes: the error is `ChunkTooLarge` for
   `big.rs` with a valid line range, and its message contains `big.rs`.
2. `a_refused_batch_is_split_and_retried` in crates/meow-index/tests/index.rs
   passes.

## What this does not settle

- Which provider statuses mean "too large". Only 413 is mapped today (from
  crates/meow-cli/src/index.rs:61, high).
- Whether a build should skip a refused chunk and go on. It stops (from
  crates/meow-index/src/index.rs:260-275, high).
