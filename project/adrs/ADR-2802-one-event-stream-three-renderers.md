---
id: ADR-2802
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2804, REQ-2805]
supersedes: []
---

# 2802. One event stream feeds three renderers

## Decision

One event stream and three renderers replace the three rendering stacks v0.2.x
runs at once (from docs/spec/tui.md, high).

Once this holds, a run picks exactly one renderer, the terminal, plain or JSON
one, and every event reaches it through one `Renderer` trait (from
crates/meow-ui/src/lib.rs:28-74 and crates/meow-cli/src/render.rs:72-116,
high). The engine's own events and a script's `ctx.out` calls travel down the
same channel, so a renderer sees one vocabulary (from
https://github.com/meowshed/meowg1k/pull/124, high).

## Why

In v0.2.x a uilive progress logger, an 807-line Bubble Tea program and a set of
lipgloss widgets each own the cursor, and which one wins depends on timing (from
docs/spec/tui.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: three rendering stacks at once, as in v0.2.x | Each stack was already written and suited one job: a progress logger, a full interactive program and reusable widgets (from docs/design/0.3.0-tui.md section 3, medium) | Each owns the cursor and which one wins is timing-dependent (from docs/spec/tui.md, high) |
| One stream for the engine's events and a second for a script's output | Script output stays apart from engine events, so neither type grows for the other's sake (reasoned from https://github.com/meowshed/meowg1k/pull/124, low) | A renderer would see two vocabularies instead of one (from https://github.com/meowshed/meowg1k/pull/124, high) |

The design documents and the pull request history show no third option weighed
against one stream (from docs/design/0.3.0-tui.md section 4 and
https://github.com/meowshed/meowg1k/pull/124, high).

## What it costs

The engine's `AgentEvent` has to be mapped to a `ViewEvent` somewhere, and
`meow-ui` may not know the engine, so a `Relay` in `meow-star` does it and keeps
the state the view needs: the name a tool call was made under, the step count
and what has been spent (from https://github.com/meowshed/meowg1k/pull/124,
high). A renderer isn't `Sync`, so every event passes through one mutex (from
crates/meow-cli/src/render.rs:15-21, high). A field the engine doesn't report
stays empty, as `RunStart.model` does (from crates/meow-star/src/run.rs:685 and
https://github.com/meowshed/meowg1k/pull/124, high).

## What would reverse it

A renderer that has to read engine state the `ViewEvent` stream doesn't carry,
to show what [REQ-2811] or [REQ-2823] require, would break the `meow-ui`
boundary and reopen the design (reasoned from CLAUDE.md, architecture, low).

## Consequences

`meow-ui` depends on `meow-core` and on no other workspace crate (from
crates/meow-ui/Cargo.toml, high). Each renderer is tested by replaying one
recorded stream, the terminal one over `ratatui`'s `TestBackend` with no
terminal attached (from crates/meow-ui/tests/renderers.rs, high). A diagnostic
has one place to go, which [ADR-2806] relies on (from
crates/meow-ui/src/tty.rs:127-139, high).

## How I will know it was realised

1. `all_three_take_the_same_recording` in crates/meow-ui/tests/renderers.rs
   passes: the plain, JSON and terminal renderers each consume the same
   `recording()` with no terminal attached.
2. The `[dependencies]` of crates/meow-ui/Cargo.toml name no workspace crate
   other than `meow-core`.

## What this does not settle

- How the renderer is chosen, which [REQ-2800] to [REQ-2803] fix (from
  docs/spec/tui.md [R-TUI-001] to [R-TUI-004], high).
- Whether the live stream and an export share one type, which [ADR-0116]
  records (from docs/adrs/ADR-0116-live-stream-and-export-share-a-type.md,
  high).
- How `ctx.out` reaches the stream, which [ADR-0424] records (from
  docs/adrs/ADR-0424-ctx-out-port-takes-one-typed-event.md, high).
