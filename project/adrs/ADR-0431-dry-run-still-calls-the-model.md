---
id: ADR-0431
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2845, REQ-2846, REQ-2847]
supersedes: []
---

# 0431. `--dry-run` still calls the model and hands it a placeholder result

## Decision

`--dry-run` decides policy, records what each call would have been and tells the
model the call succeeded, and it says once that everything after the first
placeholder is a plausible run rather than the run (from
https://github.com/retran/meowg1k/pull/127, high). It plans tool calls, not
model calls, so it still calls the model (from
https://github.com/retran/meowg1k/pull/127, high).

Once this is accepted, a dry run replaces every declared tool with a planned
one that writes `would run <tool>(<arguments>)` and returns a placeholder
telling the model to continue as if the call succeeded (from
crates/meow-star/src/run.rs:387 and 846, high). Policy still decides before
each call, so a call policy denies is denied and an `ask` still asks a person
(from crates/meow-agent/src/engine.rs:357 to 385, high). The divergence warning
is stated once per planned tool rather than once per run: each planned tool
carries its own flag, so a run that plans two different tools warns twice (from
crates/meow-star/src/run.rs:393, medium).

## Why

What gets called is what the model asks for, so skipping the model would mean
planning nothing (from https://github.com/retran/meowg1k/pull/127, high). What
the model does next depends on a result it never received, which is why the
divergence is stated (from https://github.com/retran/meowg1k/pull/127, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no `--dry-run` | No run can be mistaken for the real one, because every result is real (reasoned from REQ-2847, low) | REQ-2845 asks for a run that evaluates policy and plans tool calls without executing any (from docs/spec/tui.md [R-TUI-072], high) |
| Skip the model during a dry run | Spends no tokens and sends nothing to a provider (reasoned from crates/meow-star/src/run.rs:387, low) | It would plan nothing, because what gets called is what the model asks for (from https://github.com/retran/meowg1k/pull/127, high) |

## What it costs

Every model call in a dry run is real, so a dry run spends tokens and money as
a real run does (reasoned from crates/meow-star/src/run.rs:387, which replaces
only the tools, low). A person still answers each `ask` for a call that won't
run (from crates/meow-agent/src/engine.rs:370, high). Everything after the
first placeholder is a plausible run rather than the run (from
https://github.com/retran/meowg1k/pull/127, high).

## What would reverse it

- A provider offers a way to plan tool calls without sampling a full turn, so
  the plan could be had without paying for the model calls (reasoned from
  https://github.com/retran/meowg1k/pull/127, low).

## Consequences

- The runtime builds a planned tool in place of each declared tool when the
  dry-run port is set, and sub-agents are still built and run (from
  crates/meow-star/src/run.rs:387 to 415, high).
- The transcript carries a `policy:` line, a `note: would run` line per call
  and a `warn: this is a dry run` line (from
  crates/meow-star/tests/approval.rs `a_dry_run_plans_without_running`, high).

## How I will know it was realised

1. crates/meow-star/tests/approval.rs `a_dry_run_plans_without_running` and
   `the_divergence_is_stated_once` pass (from those tests, high).
2. A dry run that plans two different tools states the divergence once; no
   test covers it, and the code states it once per tool (from
   crates/meow-star/src/run.rs:393, medium).

## What this does not settle

- Whether a dry run should skip the approval prompt for a call it won't make
  (from crates/meow-agent/src/engine.rs:370, high).
- Whether the model calls a dry run makes count against the budget and the
  request ceilings as a real run's do (reasoned from
  crates/meow-star/src/run.rs:387, low).
