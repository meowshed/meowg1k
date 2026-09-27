---
id: ADR-0115
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2211, REQ-2212, REQ-2213]
supersedes: []
---

# 0115. The two rebuild modes are two functions over one iterator

## Decision

Rebuilding the log for a model call and rebuilding it for display or export are
written as two functions over one iterator (from docs/design/0.3.0-plan.md M2,
high). `Sessions::events` returns every event as written, and
`Sessions::events_for_model` walks the output of `events` and substitutes each
summary where its range stood (from crates/meow-session/src/log.rs:209-270,
high).

## Why

The two rebuild modes are the pair most likely to be written as one function
with a flag and then to drift (from docs/design/0.3.0-plan.md M2, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: compaction rewrites the log, as v0.2.x's `mark_obsolete` did, so one reading serves both | One rebuild, with nothing to keep level (reasoned from docs/design/0.3.0-sessions.md section 2, low) | A compacted run can no longer be replayed in full, because the original messages are gone (from docs/design/0.3.0-sessions.md section 2, high) |
| One rebuild function with a flag | One walk over the log, so a change to how events decode lands once (reasoned from crates/meow-session/src/log.rs:230-236, low) | It is the shape most likely to drift (from docs/design/0.3.0-plan.md M2, high) |
| Two functions that each read the store themselves | Each reading is independent of the other and can change alone (reasoned from crates/meow-session/src/log.rs, low) | Decoding would be written twice, and the model reading would stop being a filter over the person's reading (reasoned from crates/meow-session/src/log.rs:237-239, low) |

## What it costs

The model rebuild reads every event and filters it, so a call pays for the whole
log even when most of it is superseded (from
crates/meow-session/src/log.rs:237-239, high). The substitution is easy to get
wrong: the first version put a summary after the messages that followed its
range, and the test asserted the wrong order for two milestones (from
https://github.com/meowshed/meowg1k/pull/129, high).

## What would reverse it

- A third reading of the log that needs both substitution and the originals,
  which a flag would then serve better than a third function (reasoned from
  crates/meow-session/src/log.rs:230-236, low).

## Consequences

Compaction supersedes a range without deleting it, so the person's rebuild
still returns every original event (from crates/meow-session/src/log.rs:209-213,
high). `meow session show` and the exports read `events`, and the engine's
resume path reads the model rebuild (from crates/meow-session/src/export.rs:52
and 111 and crates/meow-cli/src/session.rs:60 and 97-99, high).

## How I will know it was realised

1. `compaction_supersedes_without_deleting` in crates/meow-session/tests/spec.rs
   passes: the person's rebuild returns every original event (from
   crates/meow-session/tests/spec.rs:180, high).
2. `the_model_rebuild_skips_the_superseded_range` in the same file passes, with
   the summary standing where its range stood (from
   crates/meow-session/tests/spec.rs:210, high).

## What this does not settle

- Whether a range can be superseded twice, which is REQ-2214 (from
  crates/meow-session/tests/spec.rs:254, high).
- How a summary is produced, which is compaction's job in the engine (from
  crates/meow-agent/src/compaction.rs, high).
