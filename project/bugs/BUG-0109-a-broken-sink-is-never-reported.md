---
id: BUG-0109
artifact: bug
status: approved
severity: minor
violates: [REQ-1065, REQ-1068]
found: 2026-09-27
revised: 2026-09-27
issue: 232
---

# A broken sink's error is kept and never reported

When a sink returns an error, the engine stores it in `Delivery.broken` and
stops sending, and nothing reads the stored error, so the failure leaves no
`Note` event and no trace in the outcome.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `cargo test -p meow-agent a_broken_sink_does_not_destroy_the_run`. It
   passes.
2. Read the test at `crates/meow-agent/tests/spec.rs:633-661`. It claims
   REQ-1065 and REQ-1068 and checks only that the run finishes and the sink was
   called once.
3. Extend it to look for a `Note` naming "the renderer fell over" in the
   session, or for the text in `outcome.detail`. Neither exists.

## What the system does

`Delivery::send` sets `self.broken = Some(e)` and returns,
`crates/meow-agent/src/engine.rs:504-511`. `broken` is read only to skip later
sends and deltas, `engine.rs:505` and `:514`. No production sink returns an
error today: `Relay::event` in `crates/meow-star/src/run.rs:677-781` always
returns `Ok(())`, so the path is dormant.

## What it should do, and why

REQ-1065: "When the sink returns an error, the engine MUST record a `Note`
event naming the error." REQ-1068: "A broken sink MUST NOT fail silently." The
error should reach the session log, or the outcome, where a person can find it.

## Triage

The defect enters in the engine's `Delivery`. It's minor because no sink the
binary wires can fail yet, so no user sees it, but the test claims the
requirement it doesn't check and the first sink that can fail will fail
silently.

## Closed by

The test `a_broken_sink_does_not_destroy_the_run`, extended to expect the
sink's error in a `Note` event or in `outcome.detail`.
