---
id: ADR-1400
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1414, REQ-1415, REQ-1416, REQ-1417]
supersedes: []
---

# 1400. Chunking is line-oriented

## Decision

The chunker splits on line boundaries and doesn't parse syntax (from
docs/spec/index.md [R-INDEX-014], high). A line longer than the whole chunk
limit is the one exception: the chunker cuts it and reports the cut (from
docs/spec/index.md [R-INDEX-014], high).

Once this is accepted, every file the walk returns is chunked the same way,
with no grammar per language, and each chunk carries whole lines that a result
can cite (from crates/meow-index/src/chunk.rs:94-98, high). What still doesn't
work is choosing a boundary by meaning: a function longer than 60 lines is cut
at line 60 wherever it falls, and the 10-line overlap is what keeps it
retrievable from either side (from crates/meow-index/src/chunk.rs:10-18,
high).

## Why

The line splitter is the baseline to measure against. A syntax-aware splitter
costs a grammar per language, a build dependency and a fallback for every
language without one (from docs/spec/index.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep v0.2.x's splitter, which cuts on blank-line paragraphs, then lines, and counts size and overlap in runes | Paragraphs of prose stay whole, and a chunk's size is exact in characters (from `git show v0.2.1:internal/core/chunker/plain_text_strategy.go`, medium) | Overlap in runes and boundaries on lines can't both be exact, and an overlong line is cut without being reported, so a minified file makes the line rule and the size rule quietly unsatisfiable (from docs/spec/index.md [R-INDEX-012] [R-INDEX-014] and https://github.com/meowshed/meowg1k/pull/130, medium) |
| A syntax-aware splitter using tree-sitter | Better chunk boundaries (from docs/spec/index.md, high) | It costs a grammar per language, a build dependency and a fallback for every language without one (from docs/spec/index.md, high) |
| Count chunk size in tokens with a real tokenizer | An exact fit to the embedding model's input limit (from https://github.com/meowshed/meowg1k/pull/130, high) | It's another dependency per model family; the chunker estimates four characters to a token, the same estimate compaction uses, and the provider's own refusal is the authority (from https://github.com/meowshed/meowg1k/pull/130, high) |

## What it costs

Chunk boundaries are worse than a syntax-aware splitter would give (from
docs/spec/index.md, medium). A chunk can start in the middle of a definition,
so a result can show the tail of one function and the head of the next
(reasoned from crates/meow-index/src/chunk.rs:10-18, low). Retrieval quality
has never been measured against an alternative, so the size of that cost is
unknown (from https://github.com/meowshed/meowg1k/pull/111, high).

## What would reverse it

- A recall number that justifies the tree-sitter dependency (from
  docs/spec/index.md, high). The pull request that closed the open questions
  lists the chunking strategy among four decisions that want a measurement
  rather than an argument (from https://github.com/meowshed/meowg1k/pull/111,
  high).

## Consequences

Replacing the line splitter is an amendment to [R-INDEX-014] (from
docs/spec/index.md, high). The chunking parameters (60 lines, 10 lines of
overlap, 8000 characters) enter the staleness hash, so changing any of them
re-chunks every file on the next update (from
crates/meow-index/src/chunk.rs:10-26 and crates/meow-index/src/index.rs:492-499,
high). `meow-index` depends on no parser crate (from
crates/meow-index/Cargo.toml, high).

## How I will know it was realised

1. `chunks_fall_on_line_boundaries` in crates/meow-index/tests/walk.rs passes:
   every chunk begins and ends on a line boundary.
2. `an_overlong_line_is_split_and_reported` in
   crates/meow-index/tests/walk.rs passes: a line over the limit is cut and
   the chunk reports `split_line`.
3. `neighbouring_chunks_overlap` in crates/meow-index/tests/walk.rs passes:
   neighbours share the configured number of lines.

## What this does not settle

- The chunk size and the overlap. They are defaults, and a workspace changes
  them with `meow.index(chunk_lines=..., overlap=...)`, which [R-INDEX-012]
  governs (from crates/meow-star/src/declare.rs:372-373 and
  crates/meow-cli/src/index.rs:86-87, high).
- How a chunk is sized against a provider's limit in tokens. The character
  limit is a stand-in, and a provider's refusal is handled by
  [R-INDEX-020] and [R-INDEX-021] (from
  https://github.com/meowshed/meowg1k/pull/130, high).
- Whether a syntax-aware splitter would improve recall on this repository.
  Nothing measures recall yet (from https://github.com/meowshed/meowg1k/pull/131,
  high).
