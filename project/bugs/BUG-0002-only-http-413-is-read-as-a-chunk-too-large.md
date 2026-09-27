---
id: BUG-0002
artifact: bug
status: approved
severity: minor
violates: REQ-1420
found: 2026-09-27
revised: 2026-09-27
issue:
---

# Only HTTP 413 is read as a chunk too large, so other refusals don't name the file

The index's embedder maps only an HTTP 413 to `Rejected::TooLarge`. A provider
that refuses an oversized input with another status, such as a 400 whose body
says the input exceeds the model's context, reaches the caller as
`Rejected::Failed`, and the error names the workspace root, not the file and
the lines.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Give `crate::index::Embedder` in `crates/meow-cli/src/index.rs` a recorded
   provider that answers any request holding a text over some length with an
   `LlmError::Http { status: 400, .. }` whose message says the input is too
   long, as OpenAI's embeddings endpoint does.
2. Run `index.embed` through it over a workspace holding one file with one
   oversized chunk.

The error reads `could not walk <root>: ...` and names neither the file nor the
line range.

## What the system does

`crates/meow-cli/src/index.rs:61` turns only `status == 413` into
`Rejected::TooLarge`; every other failure becomes `Rejected::Failed`. The
splitter in `crates/meow-index/src/embed.rs:81-93` halves a batch only on
`TooLarge`, so a 400 is never narrowed to one chunk, and
`crates/meow-index/src/index.rs:272-275` reports `Failed` as
`IndexError::Walk { root, message }`, a walk error for what is an embedding
refusal.

## What it should do, and why

REQ-1420: "A single chunk that a provider rejects as too large MUST fail with an
error naming the file and the chunk's line range, not a generic size error."
ADR-1404 records the decision. The requirement is about the provider rejecting
the chunk, whatever status it uses to say so.

## Triage

The defect enters in `meow-cli`'s `Embedder`, which classifies the refusal, and
in `Index::embed`, which labels an embedding failure as a walk failure. It is
minor: a provider that sends 413 gets the right error, and for the rest the run
still fails, only without the location. The fix is to let each provider classify
its own size refusal, for example a field on `LlmError::Http` filled from the
vendor's documented error code.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`a_provider_refusing_a_long_input_with_400_names_the_file`, that drives the
embedder with a recorded 400 size refusal and expects `ChunkTooLarge` with the
file and lines.
