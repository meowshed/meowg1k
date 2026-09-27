---
id: BUG-0010
artifact: bug
status: approved
severity: major
violates: REQ-1610
found: 2026-09-27
revised: 2026-09-27
issue:
---

# The OpenAI-shaped provider assigns no identifier to a tool call without one

A non-streamed response from an OpenAI-shaped server whose tool calls lack `id`
keeps only the first call, with an empty identifier. The deduplication by
identifier runs on the empty string, so every later call without an id is
dropped as a repeat of the first.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Give the OpenAI provider a recorded non-streamed answer whose message holds
   two `tool_calls` entries with different function names and no `id` field.
2. Call `generate`.

The response carries one tool call, whose `id` is `""`.

## What the system does

`crates/meow-llm/src/openai.rs:239` takes `call["id"]` or the empty string, and
`crates/meow-llm/src/openai.rs:240-243` skips any call whose id is already in
the list. No code in the provider assigns an id. The streamed path at
`crates/meow-llm/src/openai.rs:543-554` starts a call only when it carries a
non-empty id, so a streamed call without one is never reported at all. The
Gemini provider does assign ids (`crates/meow-llm/src/gemini.rs:217-222`).

## What it should do, and why

REQ-1610: "A provider MUST assign an identifier to every tool call it returns
that lacks one." REQ-1611 then needs them unique, and REQ-1612 deduplicates only
calls that arrive with the same identifier. ADR-1604 records the decision to
deduplicate at the provider.

## Triage

The defect enters in `meow-llm`'s OpenAI-shaped provider, which also serves
`openrouter` and `llama`. It is major because tool calls are silently lost, and
a tool result sent back under an empty id can't be tied to its call. It shows
only with a server that omits ids; the official API sends them.

## Closed by

A test in `crates/meow-llm/tests/spec.rs`, named for example
`openai_calls_without_ids_each_get_one`, that records two calls without ids and
expects two calls with distinct, non-empty ids, for both `generate` and
`generate_stream`.
