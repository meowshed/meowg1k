---
id: ADR-2200
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2209, REQ-2210, REQ-2212, REQ-2213]
supersedes: []
---

# 2200. Compaction supersedes a range of events without deleting it

## Decision

A `Compaction` event names the inclusive sequence range it supersedes and leaves
the events in that range in place, and rebuilding the log for display or export
returns the original events with no summary substituted (from
docs/spec/session.md [R-SESSION-010] [R-SESSION-012], high).

Once this holds, a compacted run replays in full for a person and for
`meow session export`, while the rebuild for a model still sees the summary in
place of the range (from docs/design/0.3.0-sessions.md section 3, high). It
doesn't yet hold end to end: the binary writes the engine's message positions
as the range, not session sequence numbers (see Consequences).

## Why

In v0.2.x, `ctx.session.mark_obsolete(ids)` mutated the log, so a compacted run
could no longer be replayed in full (from docs/spec/session.md, high). The full
run surviving compaction is what lets a person continue or inspect a long agent
loop after it compacted, not only before (from docs/design/0.3.0-sessions.md
section 3, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Mark compacted events obsolete in the log, as v0.2.x's `ctx.session.mark_obsolete(ids)` did (the do-nothing option) | A smaller log, and one rebuild that needs no range lookup, because the obsolete events are gone (reasoned from docs/design/0.3.0-sessions.md section 2, low) | Mutating the log means a compacted run can no longer be replayed in full (from docs/spec/session.md, high) |
| Delete the superseded events and keep only the summary | The smallest log, and a model rebuild that is a plain read (reasoned from crates/meow-session/src/log.rs:237-267, low) | It breaks the append-only rule every other property depends on, and the original messages are gone (from docs/design/0.3.0-sessions.md section 3, high) |
| One rebuild function with a flag choosing summary or originals | One walk to maintain in place of two (from https://github.com/retran/meowg1k/pull/116, high) | A shared function with a boolean is how the two readings would drift back together (from crates/meow-session/src/log.rs:234-236, high) |

(from docs/spec/session.md, high)

## What it costs

The log keeps every superseded event, so it grows by the summary on top of the
originals, and retention by age, count and size is what bounds it (from
docs/design/0.3.0-sessions.md section 8, medium). Each model rebuild reads the
compaction side table and walks every event to substitute summaries (from
crates/meow-session/src/log.rs:237-267, high).

## What would reverse it

- A session database whose superseded events dominate its size under the
  configured retention limits, so that keeping originals costs more than a
  replay is worth (reasoned from docs/design/0.3.0-sessions.md section 8, low).

## Consequences

- `Sessions::events` returns every event as written and
  `Sessions::events_for_model` substitutes each summary where its range stood
  (from crates/meow-session/src/log.rs:209-267, high).
- A fork can't start inside a superseded range, because it would copy events
  the origin's own model rebuild no longer shows (from
  crates/meow-session/src/fork.rs:61-75, high).
- The binary records the engine's `Compacted` event through `Sessions::append`,
  converting the engine's message positions straight into the range, so the
  range recorded is message positions, not session sequence numbers, and the
  overlap check in `Sessions::compact` is bypassed (from
  crates/meow-star/src/run.rs:757-768, crates/meow-agent/src/event.rs:73 and
  crates/meow-cli/src/session.rs:139-142, high).

## How I will know it was realised

1. `compaction_supersedes_without_deleting` in
   crates/meow-session/tests/spec.rs passes: after a compaction, the rebuild
   for a person still returns the superseded events (from
   crates/meow-session/tests/spec.rs:177-206, high).
2. `the_model_rebuild_skips_the_superseded_range` in the same file passes (from
   crates/meow-session/tests/spec.rs:208-250, high).
3. A run of the binary that compacts records a `Compaction` event whose range
   names session sequence numbers; no test checks this yet (reasoned from
   crates/meow-star/src/run.rs:757-768, low).

## What this does not settle

- When to compact, on which model, and how the summary is written; the engine
  decides those (from docs/spec/session.md Scope, high; see ADR-1001 and
  ADR-1002).
- Whether one compaction may overlap another, which [R-SESSION-013] settles
  (from docs/spec/session.md, high).
