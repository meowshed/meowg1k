---
id: BUG-0045
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# Compaction estimates tokens from bytes, and the chunker counts characters

ADR-0413 has the engine and the chunker use one estimate, four characters to a
token, "so the two can't disagree about how big something is". Compaction
measures `String::len`, which is bytes, and the chunker measures
`chars().count()`. On text that isn't ASCII they disagree by up to a factor of
four.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Call `meow_agent::estimate_tokens` on one message holding 4000
   Cyrillic letters. It returns 2000, from 8000 bytes.
2. Split a file holding the same 4000 letters with the default chunker. It
   counts 4000 characters, well under `max_chars`.

This was traced through the code and not run.

## What the system does

`crates/meow-agent/src/compaction.rs:42-48` sums `content.len()` and
`thinking.len()`, and `crates/meow-index/src/chunk.rs:131` compares
`piece.chars().count()` with `max_chars`. ADR-0413's own Decision section
already records that the two agree only on ASCII text.

## What it should do, and why

No requirement fixes the unit. ADR-0413's stated reason for one estimate is
that the two can't disagree, and they do. For a workspace in Russian or Chinese,
compaction fires at about half or a third of the context the user configured.

## Triage

The defect enters in `meow-agent`'s `estimate_tokens`. It is minor: the
estimate is a threshold, never a bill, and early compaction costs a summary
call. The fix is to count characters in `estimate_tokens`.

## Closed by

A test in `crates/meow-agent/tests/`, named for example
`estimate_tokens_counts_characters_not_bytes`, that expects 1000 for 4000
Cyrillic letters.
