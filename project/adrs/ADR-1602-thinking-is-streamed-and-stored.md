---
id: ADR-1602
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1622, REQ-1623]
supersedes: []
---

# 1602. Thinking content is streamed and stored

## Decision

Thinking content arrives as `Thinking` stream events and stays on the assistant
message it belongs to [REQ-1622] [REQ-1623] (from docs/spec/llm.md, high).

In the code, Anthropic reads `thinking` blocks and `thinking_delta` events,
OpenAI-shaped servers read `reasoning_content` or `reasoning`, and Gemini never
reads reasoning, so its `thinking` is always absent (from
crates/meow-llm/src/anthropic.rs:203-205, 413-414, high;
crates/meow-llm/src/openai.rs:222-229, high;
https://github.com/meowshed/meowg1k/pull/133, high). The engine copies the
response's thinking onto the assistant message it keeps in the run's history,
and Anthropic sends it back first on the next turn (from
crates/meow-agent/src/engine.rs:271-274, high;
crates/meow-llm/src/anthropic.rs:101-106, high).

## Why

Anthropic requires thinking blocks to be sent back on a later turn that
continues a tool call, so discarding them breaks resume and multi-turn tool use
on the provider the tool is built against first. Storage is the cheaper problem,
and content addressing plus retention already handle it (from docs/spec/llm.md,
high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Stream thinking and never store it, the spec's first answer | Thinking is bulky and only useful to watch, so dropping it keeps the record small (from docs/spec/llm.md, high) | Wrong on a fact: Anthropic requires thinking blocks back on a later turn that continues a tool call, so discarding them breaks resume and multi-turn tool use (from https://github.com/meowshed/meowg1k/pull/111, high) |

Neither the code nor the forge history names a second option, and doing nothing
is the row above: the spec's first answer was the prior state (from
https://github.com/meowshed/meowg1k/pull/111, high).

## What it costs

Thinking is bulky, so storing it costs space, which content addressing and
retention already handle (from docs/spec/llm.md, medium). It also costs context:
compaction counts a message's thinking with its text, so a model that thinks
at length reaches the compaction threshold sooner (from
crates/meow-agent/src/compaction.rs:45, high).

## What would reverse it

- Anthropic stops requiring thinking blocks on a turn that continues a tool
  call, and no other provider the engine targets requires them (reasoned from
  docs/spec/llm.md [R-LLM-024], low).

## Consequences

`meow session export` redacts thinking by default, because it is the part of a
transcript least likely to be meant for an audience (from docs/spec/llm.md,
high). The export filters `Note` events at level `thinking` unless
`--thinking` is passed (from crates/meow-session/src/export.rs:206-216, high;
crates/meow-cli/src/wire.rs:715, high).

The session log doesn't store thinking yet: the `Assistant` event has no
thinking field, and no production code writes a `Note` at level `thinking`, so
a resumed session has none to send back (from
crates/meow-core/src/event.rs:86-92, high; crates/meow-star/src/run.rs:639-647,
high; `grep -rn '"thinking"' crates/*/src`, high).

## How I will know it was realised

1. `thinking_is_kept_and_sent_back_on_the_next_turn` in
   `crates/meow-llm/tests/spec.rs` passes: the parsed response carries the
   thinking, and the next request's first assistant block is of type
   `thinking` (from crates/meow-llm/tests/spec.rs:312-345, high).
2. `reasoning_is_omitted_unless_it_is_asked_for` in
   `crates/meow-session/tests/fork.rs` passes for the export (from
   crates/meow-session/tests/fork.rs:358-390, high).
3. For the "stored" half: a resumed session's first request carries the
   previous run's thinking, which no test checks today (reasoned from
   crates/meow-core/src/event.rs:86-92, medium).

## What this does not settle

- How thinking reaches the session log, which the `Assistant` event can't carry
  today (from crates/meow-core/src/event.rs:86-92, high).
- The `signature` Anthropic returns with a thinking block: the provider neither
  reads nor sends it back, and whether the live API accepts a block without it
  is unverified here (from crates/meow-llm/src/anthropic.rs:101-106, 203-205,
  high; Anthropic's API requirement, low).
- Reading Gemini's reasoning, which is a model setting rather than a request
  field (from https://github.com/meowshed/meowg1k/pull/133, high).
