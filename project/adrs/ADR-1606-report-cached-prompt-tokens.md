---
id: ADR-1606
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1638, REQ-1639]
supersedes: []
---

# 1606. A response reports cached prompt tokens as a separate count

## Decision

A response reports cached prompt tokens as a separate optional count, left
absent when the provider doesn't report caching [REQ-1638] [REQ-1639] (from
docs/spec/llm.md, high).

The count is `cached: Option<u32>` on `meow_core::Usage` (from
crates/meow-core/src/usage.rs:23, high). Anthropic reads
`cache_read_input_tokens`, the OpenAI-shaped provider reads
`prompt_tokens_details.cached_tokens` and Gemini reads
`cachedContentTokenCount`, and each leaves the count absent when its field is
(from crates/meow-llm/src/anthropic.rs:243-256, high;
crates/meow-llm/src/lib.rs:49-67, high; crates/meow-llm/src/gemini.rs:277-278,
high).

## Why

Cached prompt tokens are the main cost lever of a well-built agent, and a record
that doesn't report them makes that lever invisible (from docs/spec/llm.md,
high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: report no cached prompt tokens, as v0.2.x does | One field fewer to read from each vendor (reasoned from crates/meow-llm/src/lib.rs:62-65, low) | It makes the main cost lever of a well-built agent invisible (from docs/spec/llm.md, high) |
| Report the count as zero when the provider says nothing, as an earlier draft of the spec did | A plain integer that adds up without an absent case (reasoned from crates/meow-core/src/usage.rs:48-53, low) | "No cache hits" can't be told from "this provider doesn't say"; the draft contradicted the rule two lines above it (from https://github.com/retran/meowg1k/pull/111, high) |

Neither the code nor the forge history names a third option.

## What it costs

Every provider maps its own field, and every reader of usage handles an absent
count: the session log stores a nullable column and Starlark shows `None`
(from crates/meow-session/src/log.rs:156, high;
crates/meow-star/src/value.rs:146-150, high). Anthropic's
`cache_creation_input_tokens` isn't read, so tokens written to the cache aren't
reported apart (from `grep -rn cache_creation crates/meow-llm/src`, high).

## What would reverse it

- No reader uses the count: nothing in a script, the transcript or an export
  shows `cached` (reasoned from crates/meow-star/src/value.rs:146-150, low).

## Consequences

- A script sees `usage.cached` as a number or `None` (from
  crates/meow-star/src/value.rs:140-150, high).
- Adding an absent count to a reported one gives the reported one, so a total
  across providers can omit part of what was cached; the cost total takes the
  other rule and becomes unknown, which ADR-0408 records (from
  crates/meow-core/src/usage.rs:44-58, high).

## How I will know it was realised

1. `an_unreported_cache_count_stays_absent_rather_than_zero` in
   `crates/meow-llm/tests/spec.rs` passes: no field gives `None`, a reported 0
   gives `Some(0)` (from crates/meow-llm/tests/spec.rs:534-563, high).
2. `usage_is_read_or_absent` in `crates/meow-llm/tests/providers.rs` passes
   for OpenAI's `cached_tokens` (from
   crates/meow-llm/tests/providers.rs:352-396, high).

## What this does not settle

- Whether a total with an unreported count should itself be absent, as cost is
  (from crates/meow-core/src/usage.rs:44-58, high).
- Setting cache hints so the count rises, which ADR-1601 covers (from
  crates/meow-agent/src/engine.rs:231-233, high).
- A usage block that is missing altogether, which [REQ-1640] covers (from
  docs/spec/llm.md [R-LLM-041], high).
