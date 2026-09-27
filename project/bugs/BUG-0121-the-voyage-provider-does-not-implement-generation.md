---
id: BUG-0121
artifact: bug
status: approved
severity: minor
violates: REQ-1600
found: 2026-09-27
revised: 2026-09-27
issue: 244
---

# The Voyage provider doesn't implement generation

`Voyage::generate` and `generate_stream` return `LlmError::Unsupported`, and its
capabilities declare `structured: Structured::None`, which REQ-1601's native or
emulated choice doesn't allow.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Read `impl Provider for Voyage` at `crates/meow-llm/src/voyage.rs:66-100`.
2. Call `generate` on it. It returns `Unsupported { capability: "generation" }`.

## What the system does

`capabilities` returns `streaming: false`, `tools: false`,
`structured: Structured::None` and `embeddings: true`,
`crates/meow-llm/src/voyage.rs:72-79`. Both generation methods refuse,
`voyage.rs:81-100`. ADR-0444 added the `None` state for this provider.

## What it should do, and why

REQ-1600: "A provider MUST implement generation." REQ-1601: a provider "MUST
declare ... whether its structured output is native or emulated."

## Triage

The requirement is what's wrong here, not the code. Voyage serves embeddings
only, and the vision's goal 8 lists it among the provider kinds, so REQ-1600
should say a provider implements generation or embeddings or both, and REQ-1601
should allow "none" for structured output. The fix enters at the requirements
step as an amendment to both. It's minor because nothing a user runs fails: a
Voyage model is declared with `kind = "embedding"` and the index is its only
caller.

## Closed by

An amendment to REQ-1600 and REQ-1601, and the existing refusal covered by a
test in `crates/meow-llm/tests/`, named for example
`an_embedding_only_provider_refuses_generation_by_name`.
