---
id: ADR-1601
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1613, REQ-1614, REQ-1615, REQ-1616]
supersedes: []
---

# 1601. Cache control is an optional hint on a message

## Decision

A message can carry a cache hint, which a provider with explicit breakpoints
translates and a provider that caches automatically ignores [REQ-1613]
[REQ-1614] [REQ-1615] (from docs/spec/llm.md, high). The hint never changes the
message's content [REQ-1616] (from docs/spec/llm.md, high).

The hint is a `cache_hint: bool` on `Message`, set by `with_cache_hint` (from
crates/meow-llm/src/message.rs:64-100, high). Anthropic turns it into
`cache_control: {"type": "ephemeral"}` on a system block; OpenAI and Gemini
never read it (from crates/meow-llm/src/anthropic.rs:82-88, high; `grep -rn
cache_hint crates/meow-llm/src`, high).

## Why

Anthropic needs explicit breakpoints and OpenAI caches on its own, so the trait
carries the weaker of the two ideas and lets each provider do what it can with
it (from docs/spec/llm.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: leave cache control out of the trait | No field on `Message` and no translation in any provider (reasoned from crates/meow-llm/src/message.rs:64-67, low) | It would forfeit the largest cost saving available to a long agent run (from docs/spec/llm.md, high) |
| Carry explicit breakpoints in the trait | Says everything Anthropic's API can express, not only yes or no (reasoned from crates/meow-llm/src/anthropic.rs:86-88, low) | OpenAI caches on its own, so the trait carries the weaker of the two ideas in its place (from docs/spec/llm.md, medium) |

Neither the code nor the forge history names a third option.

## What it costs

Every `Message` carries a field that two of the three chat providers ignore
(from crates/meow-llm/src/message.rs:67, high). Anthropic translates the hint
only on a system message; a hint on a user, assistant or tool message is
dropped without a breakpoint (from crates/meow-llm/src/anthropic.rs:79-120,
high).

## What would reverse it

- A provider is added whose API needs more than yes or no on a message, such as
  a lifetime per breakpoint, and the boolean can't express it (reasoned from
  crates/meow-llm/src/message.rs:67, low).

## Consequences

- Nothing outside the tests sets a hint: the engine builds its request with no
  call to `with_cache_hint`, so no production request carries a breakpoint yet
  and the saving the Why names isn't realised (from
  crates/meow-agent/src/engine.rs:231-233, high; `grep -rn with_cache_hint
  crates`, high).
- Anthropic's request body gains a `cache_control` object and keeps the text
  unchanged (from crates/meow-llm/src/anthropic.rs:85-88, high).

## How I will know it was realised

1. `a_cache_hint_becomes_a_breakpoint_without_changing_the_message` in
   `crates/meow-llm/tests/spec.rs` passes: the system text is unchanged and
   carries `cache_control.type == "ephemeral"` (from
   crates/meow-llm/tests/spec.rs:178-203, high).
2. In use, an Anthropic run's second step reports `usage.cached` above zero,
   which needs a caller that sets the hint first (reasoned from
   crates/meow-llm/src/anthropic.rs:251-254, low).

## What this does not settle

- Which messages the engine marks, and when: nothing marks one yet (from
  crates/meow-agent/src/engine.rs:231-233, high).
- Whether a hint on a non-system message becomes a breakpoint on Anthropic,
  which [REQ-1614] requires and the code doesn't do (from
  crates/meow-llm/src/anthropic.rs:79-120, high).
- Gemini's separate context caching, which the provider doesn't use (from
  crates/meow-llm/src/gemini.rs, medium).
