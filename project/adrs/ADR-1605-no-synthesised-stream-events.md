---
id: ADR-1605
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1619]
supersedes: []
---

# 1605. A provider without native streaming declares streaming unsupported

## Decision

A provider with no native streaming endpoint declares streaming unsupported and
synthesises no events from a completed response [REQ-1619] (from
docs/spec/llm.md, high).

`Capabilities.streaming` declares it, and the default `generate_stream` fails
with `LlmError::Unsupported` naming the provider and `streaming` (from
crates/meow-llm/src/provider.rs:36-45, 63-84, high). Anthropic, the
OpenAI-shaped provider and Gemini declare streaming; Voyage, which only
embeds, doesn't (from crates/meow-llm/src/anthropic.rs:267, high;
crates/meow-llm/src/openai.rs:309, high; crates/meow-llm/src/gemini.rs:293,
high; crates/meow-llm/src/voyage.rs:74, high).

## Why

Events synthesised from a completed response report progress that never occurred
(from docs/spec/llm.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: synthesise stream events from a completed response, as v0.2.x's `llama.go` does | Every provider gives a caller events, so a display has one path (reasoned from `git show v0.2.1:internal/adapters/gateway/llama.go`, lines 304-321, low) | It reports progress that never occurred (from docs/spec/llm.md, high) |
| Require every provider to stream | A caller never meets an unsupported stream (reasoned from https://github.com/retran/meowg1k/pull/110, low) | A provider with no endpoint could meet it only by synthesising; the contradiction between [R-LLM-001] and [R-LLM-022] was resolved by making streaming a declared capability, like tool calling (from https://github.com/retran/meowg1k/pull/110, high) |

Neither the code nor the forge history names a third option.

## What it costs

A caller has to branch on the capability: the engine streams only when the
provider declares it and the sink wants deltas, and calls `generate` otherwise,
so a person watching sees nothing until the whole answer arrives (from
crates/meow-agent/src/engine.rs:235-243, high).

## What would reverse it

- A provider without a streaming endpoint is added for generation, and a
  display that can't tell it from a stalled stream is reported as hanging
  (reasoned from crates/meow-agent/src/engine.rs:235-243, low).

## Consequences

- `llama.cpp`'s server, which v0.2.x synthesised events for, is now served by
  the OpenAI-shaped provider and streams natively (from
  crates/meow-cli/src/wire.rs:1360, high; crates/meow-llm/src/openai.rs:6,
  high; https://github.com/retran/meowg1k/pull/133, high).
- `check_supported` refuses a streamed request to such a provider before any
  request is sent, per [REQ-1602] (from crates/meow-llm/src/provider.rs:107-114,
  high).

## How I will know it was realised

1. `a_provider_without_streaming_refuses_rather_than_synthesising_it` in
   `crates/meow-llm/tests/spec.rs` passes: the error is `Unsupported` for
   `streaming` and the sink received no event (from
   crates/meow-llm/tests/spec.rs:255-274, high).
2. `an_undeclared_capability_is_refused_without_sending_anything` in the same
   file passes (from crates/meow-llm/tests/spec.rs:86-115, high).

## What this does not settle

- What a display shows while a non-streaming provider works (reasoned from
  crates/meow-agent/src/engine.rs:235-243, low).
