---
id: ADR-1604
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1612]
supersedes: []
---

# 1604. Tool calls are deduplicated at the provider boundary

## Decision

A provider removes tool calls that arrive more than once with the same
identifier before it returns the response, whatever the session or history
setting [REQ-1612] (from docs/spec/llm.md, high).

Anthropic and the OpenAI-shaped provider skip a call whose identifier is
already in the list they are building, and the stream aggregator does the same
for a repeated `ToolCallStart` (from crates/meow-llm/src/anthropic.rs:206-213,
high; crates/meow-llm/src/openai.rs:239-243, high;
crates/meow-llm/src/stream.rs:75-86, high). Gemini sends no identifiers, so it
builds each one from the tool's name and a count, and two of them can't
collide (from crates/meow-llm/src/gemini.rs:212-226, high).

## Why

The provider boundary is where the duplication happens, so deduplicating there
covers every caller (from docs/spec/llm.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: deduplicate in `module_llm.go` only inside `if useSession && ...`, as v0.2.x does | Keeps the check beside the session history it protected (reasoned from docs/spec/llm.md, low) | With `use_session=False` a duplicated streamed call executes twice (from docs/spec/llm.md, high) |
| Let the store tolerate a repeated identifier with `INSERT OR IGNORE`, as v0.2.x's agent change did | The session write never fails on a repeat (from https://github.com/meowshed/meowg1k/pull/89#discussion_r2895914429, high) | It drops rows without a word and hides the bug upstream; the reviewer asked for the identifiers to be deduplicated earlier (from https://github.com/meowshed/meowg1k/pull/89#discussion_r2895914429, high) |

Neither the code nor the forge history names a third option.

## What it costs

Each call is compared with every call before it, which is quadratic in the
calls of one response and negligible at the counts a model returns (reasoned
from crates/meow-llm/src/anthropic.rs:211, low). Two different calls that share
an identifier lose the second, because only the identifier is compared (from
crates/meow-llm/src/anthropic.rs:207-213, high).

The OpenAI-shaped provider compares the identifier before assigning a missing
one, so a non-streamed response whose calls carry no `id` keeps only the first
call (from crates/meow-llm/src/openai.rs:239-243, high).

## What would reverse it

- A vendor documents that a repeated identifier can carry a distinct call, or a
  recorded exchange shows two different calls sharing one (reasoned from
  crates/meow-llm/src/anthropic.rs:207-213, low).

## Consequences

- Every caller gets a list without repeats: the engine, a Starlark handler, a
  session on or off (from https://github.com/meowshed/meowg1k/pull/117, high).
- The streamed and non-streamed paths both deduplicate, so they agree, which
  [REQ-1618] needs (from crates/meow-llm/src/stream.rs:75-86, high).

## How I will know it was realised

1. `a_repeated_tool_call_identifier_produces_one_call` in
   `crates/meow-llm/tests/spec.rs` passes for Anthropic (from
   crates/meow-llm/tests/spec.rs:153-176, high).
2. `a_repeated_tool_call_identifier_is_one_call` in
   `crates/meow-llm/tests/providers.rs` passes for OpenAI (from
   crates/meow-llm/tests/providers.rs:104-131, high).
3. `gemini_calls_get_identifiers_that_do_not_collide` in the same file passes
   (from crates/meow-llm/tests/providers.rs:175-201, high).

## What this does not settle

- Assigning an identifier to a call that lacks one, which [REQ-1610] requires
  and the OpenAI-shaped provider doesn't do before it deduplicates (from
  crates/meow-llm/src/openai.rs:239-243, high).
- A repeated identifier across two responses in one session (reasoned from
  crates/meow-llm/src/anthropic.rs:192-213, low).
