---
id: ADR-0413
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1039, REQ-1413]
supersedes: []
---

# 0413. Token counts are estimated at four characters a token

## Decision

The engine estimates four characters to a token when it decides whether to
compact (from https://github.com/retran/meowg1k/pull/120, high). The chunker's
`max_chars` uses the same estimate as a stand-in for a token limit, so the two
can't disagree about how big something is (from
https://github.com/retran/meowg1k/pull/130, high). `estimate_tokens` divides the
length of every message's content and thinking by four, and
`DEFAULT_MAX_CHARS` is 8000 (from crates/meow-agent/src/compaction.rs:37-48 and
crates/meow-index/src/chunk.rs:20-26, high).

Once this is accepted, compaction fires when the estimate reaches the
configured fraction of the context window, 0.8 by default, and no chunk is
longer than 8000 characters (from crates/meow-agent/src/compaction.rs:30 and
crates/meow-agent/src/compaction.rs:56-63, high). The two don't count the same
unit: compaction measures bytes with `len()`, and the chunker counts
characters with `chars().count()`, so they agree only on ASCII text (from
crates/meow-agent/src/compaction.rs:45 and crates/meow-index/src/chunk.rs:131,
high).

## Why

The estimate is close enough to decide when to summarise, and a real tokenizer
is a dependency a threshold doesn't need (from
https://github.com/retran/meowg1k/pull/120, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no estimate, as in v0.2.x, where compaction lived in a userland `.star` library each command had to remember to call | The runtime carries no size heuristic at all (from docs/spec/agent.md, Changes from v0.2.x, high) | `R-AGENT-040` makes the engine compact before a call that would exceed the threshold, which needs a size (from docs/spec/agent.md [R-AGENT-040], high) |
| A real tokenizer | It would be exact (from https://github.com/retran/meowg1k/pull/130, high) | It is another dependency per model family, which a threshold doesn't need (from https://github.com/retran/meowg1k/pull/130, high) |
| Rely on the provider's own limit and react to its rejection | The provider's limit is the authority, and `R-INDEX-021` already uses it for a chunk it rejects (from https://github.com/retran/meowg1k/pull/130, high) | A rejection arrives after the call, and `R-AGENT-040` asks for compaction before it (reasoned from docs/spec/agent.md [R-AGENT-040], low) |

## What it costs

The estimate is wrong enough that nothing should bill from it (from
https://github.com/retran/meowg1k/pull/120, high). Text with more characters to
a token than four, or non-ASCII text measured in bytes, moves the threshold, so
compaction can fire early or late (reasoned from
crates/meow-agent/src/compaction.rs:45-47, low).

## What would reverse it

- A provider rejects a request for exceeding its context window in a run that
  compaction should have caught, which shows the estimate undercounts (reasoned
  from crates/meow-agent/src/compaction.rs:56-63, low).
- An embedding provider rejects a chunk under 8000 characters as too large
  (reasoned from crates/meow-index/src/chunk.rs:20-26, low).

## Consequences

- The engine's summary records the tokens it saved, measured with the same
  estimate before and after (from crates/meow-agent/src/engine.rs:180 and
  crates/meow-agent/src/engine.rs:212, high).
- `estimate_tokens` is public, exported from `meow_agent` (from
  crates/meow-agent/src/lib.rs:24, high).
- A line longer than `max_chars` is cut and reported, the one case where a
  chunk boundary falls inside a line (from
  https://github.com/retran/meowg1k/pull/130, high).

## How I will know it was realised

1. `compaction_triggers_on_the_threshold_and_keeps_the_recent_ones` in
   crates/meow-agent/tests/spec.rs passes (from
   crates/meow-agent/tests/spec.rs:839-842, high).
2. `no_chunk_exceeds_the_limit` in crates/meow-index/tests/walk.rs passes (from
   crates/meow-index/tests/walk.rs:222-224, high).

## What this does not settle

- Where a chunk is too large, `R-INDEX-021` makes the provider's own limit the
  authority, not this estimate (from https://github.com/retran/meowg1k/pull/130,
  high).
- Whether the estimate counts bytes or characters: the two uses differ today
  (from crates/meow-agent/src/compaction.rs:45 and
  crates/meow-index/src/chunk.rs:131, high).
