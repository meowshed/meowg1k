---
id: ADR-0116
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2257, REQ-2258, REQ-2826, REQ-2827]
supersedes: []
---

# 0116. The live JSON stream and the session export share one event type

## Decision

`ViewEvent::Logged` carries an `EventKind` unchanged, and serde's untagged
representation makes [R-TUI-032] hold by construction (from
docs/design/0.3.0-plan.md M8, high). The export writes the same `ViewEvent`
values the live `--format json` renderer writes, behind one `SCHEMA_VERSION`
(from crates/meow-session/src/export.rs:38-67 and
crates/meow-core/src/view.rs:19-25, high).

## Why

A parallel vocabulary for the live stream would have to be kept level by hand,
which the source calls the shape of every schema that drifts (from
docs/design/0.3.0-plan.md M8, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no machine-readable mode, as in v0.2.x | Nothing to version and no schema to keep stable for other tools (reasoned from docs/design/0.3.0-tui.md section 3, low) | The tool can't be composed with anything, and `meow review \| jq` is impossible (from docs/design/0.3.0-tui.md section 3, high) |
| A separate event vocabulary for the live stream | The live stream could shape its events for a reader without touching the log's schema (reasoned from crates/meow-core/src/view.rs:1-12, low) | It must be kept level with the log by hand, and that is how a schema drifts (from docs/design/0.3.0-plan.md M8, high) |
| Two types that agree, checked by a test or by review | Each crate owns its own type and changes it on its own schedule (reasoned from https://github.com/meowshed/meowg1k/pull/128, low) | Export shares the live renderer's types in place of agreeing with them, so the requirement holds by construction and not by review (from https://github.com/meowshed/meowg1k/pull/128, high) |

## What it costs

A change to a persisted event kind is a change to the live stream's schema too,
and the version goes up when a kind is added, removed or changes meaning (from
crates/meow-core/src/view.rs:19-25, high). The live-only kinds must never share
a name with a persisted kind, or the stream becomes ambiguous to `jq` (from
crates/meow-core/src/view.rs:56-59, high).

## What would reverse it

- A requirement that the live stream carry a persisted kind in a different
  shape from the export, which would contradict REQ-2827 (reasoned from
  docs/requirements/REQ-2827-persisted-kinds-serialise-identically.md, low).

## Consequences

`--format json` on a stored session emits the same event schema as a live
`--format json` run (from docs/design/0.3.0-sessions.md section 9, high). The
design's claim that `meow session show` reuses the live renderers doesn't hold
in the code: `show` prints the markdown export (from
crates/meow-cli/src/wire.rs:665-674, high).

## How I will know it was realised

1. `a_persisted_kind_serialises_the_same_in_both` in
   crates/meow-ui/tests/renderers.rs passes: a `ToolResult` serialises to the
   same JSON on its own and through `ViewEvent::Logged` (from
   crates/meow-ui/tests/renderers.rs:225, high).
2. `a_json_export_uses_the_live_schema` in crates/meow-session/tests/fork.rs
   passes: the export opens with the live schema version and carries only
   persisted kinds (from crates/meow-session/tests/fork.rs:278, high).
3. `live_and_persisted_kinds_do_not_collide` in
   crates/meow-ui/tests/renderers.rs passes (from
   crates/meow-ui/tests/renderers.rs:243, high).

## What this does not settle

- Whether `meow session show` replays a session through the live renderers, as
  docs/design/0.3.0-sessions.md section 9 describes; today it prints markdown
  (from crates/meow-cli/src/wire.rs:665-674, high).
- The markdown export's layout, which is `format!` calls in one function with
  no template (from https://github.com/meowshed/meowg1k/pull/128, high).
