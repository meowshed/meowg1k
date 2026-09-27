---
id: ADR-0444
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1601, REQ-1602]
supersedes: []
---

# 0444. `Structured` has a third state for a provider that produces no text

## Decision

`Structured` gained a third state for a provider that produces no text at all
(from https://github.com/meowshed/meowg1k/pull/133, high).

What works now: `Structured` is `Native`, `Emulated` or `None`; Voyage declares
`None`, and `check_supported` refuses a request carrying an output schema to a
provider declaring `None`, naming the provider and "structured output", before
anything is sent (from crates/meow-llm/src/provider.rs:18,
crates/meow-llm/src/voyage.rs:76 and crates/meow-llm/src/provider.rs:121,
high). What still doesn't: `R-LLM-001` names only two states, native or
emulated, so the requirement text lags the type (from docs/spec/llm.md
`R-LLM-001`, high).

## Why

Saying Voyage emulates a schema would claim it asks for JSON in a prompt it
never sends, and `R-LLM-002` would then accept a schema it can't satisfy (from
https://github.com/meowshed/meowg1k/pull/133, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep two states and declare such a provider's structured output emulated | Matches the two states `R-LLM-001` names, with no new variant for every match to handle (from docs/spec/llm.md `R-LLM-001`, medium) | `R-LLM-002` would accept a schema the provider can't satisfy (from https://github.com/meowshed/meowg1k/pull/133, high) |
| Declare generation itself as a capability and leave `Structured` at two states | One flag would cover every text feature an embedding-only provider lacks, where today Voyage refuses generation only when called (reasoned from crates/meow-llm/src/voyage.rs:82, low) | The source doesn't weigh it; `R-LLM-001` says every provider implements generation, so a generation flag would contradict the requirement outright (reasoned from docs/spec/llm.md `R-LLM-001`, low) |

A third option isn't named by the code, the history or the design documents,
so the table stops at two.

## What it costs

Every `match` on `Structured` handles a third arm, and each provider has to
choose among three states when it declares its capabilities (reasoned from
crates/meow-llm/src/provider.rs:18, low).

## What would reverse it

An embedding-only provider stops existing among the supported kinds, or the
capabilities gain a separate declaration that a provider generates no text,
either of which leaves `None` with nothing to describe (reasoned from
crates/meow-llm/src/voyage.rs:76, low).

## Consequences

- A schema sent to Voyage fails with `LlmError::Unsupported` naming "structured
  output" at the capability check, before any request (from
  crates/meow-llm/src/provider.rs:121, high).
- Generation on Voyage fails with `LlmError::Unsupported` naming "generation"
  from inside `generate`, which is a refusal no declared capability predicts
  (from crates/meow-llm/src/voyage.rs:82, high).

## How I will know it was realised

`crates/meow-llm/tests/providers.rs`
`an_embedding_only_provider_refuses_generation` passes and the recorded
transport holds no body (from crates/meow-llm/tests/providers.rs:423, high).
No test yet sends an output schema to a provider declaring `Structured::None`
and checks the "structured output" refusal (from grep of
crates/meow-llm/tests for `Structured::None`, medium).

## What this does not settle

- `R-LLM-001` says a provider MUST implement generation and names two
  structured-output states, while Voyage refuses generation and declares a
  third state; the requirement hasn't been amended to match (from
  docs/spec/llm.md `R-LLM-001` and crates/meow-llm/src/voyage.rs:82, high).
