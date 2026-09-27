---
id: ADR-0426
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2804, REQ-2805]
supersedes: []
---

# 0426. The mapping from engine events to view events lives in `meow-star`

## Decision

`Relay` lives in `meow-star`, not in `meow-ui` (from
https://github.com/retran/meowg1k/pull/124, high).

Once this is accepted, `Relay` implements the engine's `Sink`, turns each
`AgentEvent` into a `ViewEvent` and sends it through the `Events` port, and
`meow-ui` depends on `meow-core`, `ratatui` and `serde_json` alone (from
crates/meow-star/src/run.rs:605-616, crates/meow-star/src/run.rs:676 and
crates/meow-ui/Cargo.toml, high). What still doesn't work is
`RunStart.model`, which stays empty because the engine reports the agent and
not the model it resolved (from https://github.com/retran/meowg1k/pull/124 and
crates/meow-star/src/run.rs:685, high).

## Why

`meow-ui` isn't allowed to know the engine exists, and that boundary is what
lets a renderer be driven from a recorded log (from
https://github.com/retran/meowg1k/pull/124, high). `Relay` also keeps the state
the view needs and the engine doesn't: the name a tool call was made under, the
step count and what has been spent (from
https://github.com/retran/meowg1k/pull/124, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Put the mapping in `meow-ui` | The renderers and the mapping to what they read sit in one crate (reasoned from crates/meow-star/src/run.rs:605-610, low) | `meow-ui` would have to know the engine exists (from https://github.com/retran/meowg1k/pull/124, high) |
| Have the engine emit `ViewEvent` itself | No mapping layer at all, since `ViewEvent` lives in `meow-core`, which `meow-agent` can already reach (reasoned from crates/meow-core/src/view.rs:30 and CLAUDE.md, section "architecture", low) | The engine would carry state it has no reason to repeat on every event, such as the tool name per call, the step count and the spend (from crates/meow-star/src/run.rs:612-615, high) |

Doing nothing isn't an option here, because before this change no renderer
existed to receive the engine's events (from
https://github.com/retran/meowg1k/pull/124, high).

## What it costs

- `RunStart.model` is carried and not filled, and filling it means threading
  the model name through `AgentEvent`, an `R-AGENT-*` change (from
  https://github.com/retran/meowg1k/pull/124, high).
- `Relay` has since taken on writing the session log as well, so one type both
  shows a run and records it (from crates/meow-star/src/run.rs:648-655 and
  commit 549ffb0, high).
- A frontend that drives `meow-agent` without `meow-star` would have to write
  its own mapping, because `Relay` is private to `run.rs` (from
  crates/meow-star/src/run.rs:617, medium).

## What would reverse it

- The architecture allowing `meow-ui` to depend on `meow-agent`, which the
  boundary in CLAUDE.md forbids today (from CLAUDE.md, section
  "architecture", high).

## Consequences

- Every renderer test replays a recorded `Vec<ViewEvent>` with no engine
  behind it (from crates/meow-ui/tests/renderers.rs:17-72 and
  https://github.com/retran/meowg1k/pull/124, high).
- `Relay` is the only place that knows the current tool, the elapsed time, the
  step count and the budget consumed at once, so it builds the live region's
  progress event (from crates/meow-star/src/run.rs:657-674, high).

## How I will know it was realised

1. `all_three_take_the_same_recording` in crates/meow-ui/tests/renderers.rs
   drives all three renderers from one recording with no terminal attached
   (from crates/meow-ui/tests/renderers.rs:162-175, high).
2. `cargo tree -p meow-ui -e normal --depth 1` lists `meow-core` and no
   `meow-agent` (from running that command on 2026-09-27, high).

## What this does not settle

- Whether `Relay` should send a step's text to the renderers as well as to
  the log. Today it doesn't, which leaves the plain renderer with no model
  text, although its own `keep` says a record that disagrees with what
  somebody watched is a defect (from crates/meow-star/src/run.rs:639-655 and
  docs/adrs/ADR-0425-the-plain-renderer-writes-text-once.md, medium).
- No test drives `Relay` itself; the tests in crates/meow-star/tests filter
  the engine's events out (from crates/meow-star/tests/running.rs:33-38, high).
