---
id: ADR-0440
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1412]
supersedes: []
---

# 0440. Chunk overlap is counted in lines

## Decision

The overlap is counted in lines for the same reason the boundaries are (from
https://github.com/meowshed/meowg1k/pull/130, high).

Once this is accepted, two neighbouring chunks share `overlap` lines, 10 by
default, and a workspace sets the number with `overlap` in `meow.index(...)`
(from crates/meow-index/src/chunk.rs:18 and crates/meow-cli/src/index.rs:87,
high). A chunk too long for `max_chars` is cut inside its window on line
boundaries, and the overlap then holds only between windows, not between the
pieces of one window (reasoned from crates/meow-index/src/chunk.rs:139, low).

## Why

A boundary is a line, and lines and tokens can't both be exact (from
https://github.com/meowshed/meowg1k/pull/130, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Count the overlap in tokens | The overlap would match what the embedding model counts, so it would cost a known share of the model's input (reasoned from crates/meow-index/src/chunk.rs:20, low) | Lines and tokens can't both be exact, and a boundary is a line (from https://github.com/meowshed/meowg1k/pull/130, high) |
| Do nothing: count it in characters, as v0.2.x did with `overlapRunes` | A size in characters bounds the chunk's length directly, whatever the line lengths are (from `git show v0.2.1:internal/core/chunker/plain_text_strategy.go`, medium) | v0.2.x's overlap was the last `overlapRunes` characters of a chunk, so the next chunk began inside a line (from `git show v0.2.1:internal/core/chunker/plain_text_strategy.go`, medium), and REQ-1414 requires boundaries on lines (from docs/spec/index.md [R-INDEX-014], high) |

The question admits no third unit the code or the history names (from
https://github.com/meowshed/meowg1k/pull/130, medium).

## What it costs

The overlap's size in characters varies with the lines it covers, so ten long
lines can take a large share of a chunk's `max_chars` (reasoned from
crates/meow-index/src/chunk.rs:118, low). An overlap equal to or larger than
`chunk_lines` makes the window advance one line at a time, so a file gives
about one chunk per line (from crates/meow-index/src/chunk.rs:117, high).

## What would reverse it

If chunk boundaries stopped falling on lines, for example a splitter that cuts
on syntax nodes, the reason for counting in lines would go with them (reasoned
from https://github.com/meowshed/meowg1k/pull/130, low). An amendment to REQ-1412
would also reverse it.

## Consequences

- The overlap is part of the staleness fingerprint, so changing it re-chunks
  every file (from crates/meow-index/src/chunk.rs:84, high).
- A definition that straddles a boundary is whole in one of the two chunks when
  it is no longer than the overlap (from crates/meow-index/tests/walk.rs:216,
  high).

## How I will know it was realised

`neighbouring_chunks_overlap` in `crates/meow-index/tests/walk.rs` passes: with
four lines a chunk and an overlap of two, the second chunk starts on line 3
(from crates/meow-index/tests/walk.rs:200, high).

## What this does not settle

- What happens to the overlap when a window is over `max_chars` and is cut into
  pieces (reasoned from crates/meow-index/src/chunk.rs:139, low).
- Whether an overlap of `chunk_lines` or more should be refused when the index
  is declared, since today it is accepted and the window advances one line at a
  time (from crates/meow-index/src/chunk.rs:117 and
  crates/meow-cli/src/index.rs:87, high).
