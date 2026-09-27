---
id: ADR-2806
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2815, REQ-2816]
supersedes: []
---

# 2806. Diagnostics go into scrollback and never through the live frame

## Decision

Diagnostics from the logging layer are inserted into scrollback in order and
never drawn over the live region (from docs/spec/tui.md [R-TUI-015], high).

Once this holds, a diagnostic reaches the terminal renderer as an event and is
committed above the live region with `insert_before`, after which the live
region is painted again (from crates/meow-ui/src/tty.rs:127-150, high). No
crate but `meow-cli` can print, because the workspace lints deny `println!` and
`eprintln!` and only `meow-cli` declares its own lints without them (from
Cargo.toml:32-37 and crates/meow-cli/Cargo.toml:49-57, high).

## Why

In v0.2.x ten `log.Printf` calls in `module_llm.go` write straight through the
live frame (from docs/spec/tui.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: log straight to the terminal, as v0.2.x's `log.Printf` calls do | Any code can report a problem in one line, with no event type and no renderer in between (reasoned from docs/design/0.3.0-tui.md section 3, low) | It writes straight through the live frame and corrupts it (from docs/spec/tui.md and docs/design/0.3.0-tui.md section 3, high) |
| Write diagnostics to stderr while the frame is drawn on stdout | stdout stays clean for a pipe, which is what `--format json` needs (from docs/spec/tui.md [R-TUI-033], high) | Both streams reach the same terminal, so `eprintln!` still writes through the live frame (from CLAUDE.md, no_stdout_behind_the_tui, high) |

No third option appears in the design documents or the pull request history
(from docs/design/0.3.0-tui.md section 5.1 and
https://github.com/retran/meowg1k/pull/124, high).

## What it costs

Every diagnostic has to become an event and pass through the one mutex in front
of the renderer (from crates/meow-cli/src/render.rs:15-21, high). The terminal
renderer repaints the live region after each committed line, or a diagnostic
leaves a blank strip where the progress was (from
crates/meow-ui/src/tty.rs:133-139, high). `meow-cli` is exempt from the lint,
so its own `eprintln!` calls are safe only while no live region is drawn, and
nothing checks that (reasoned from crates/meow-cli/Cargo.toml:49-57 and
crates/meow-cli/src/wire.rs:1160-1180, low).

## What would reverse it

A diagnostic that has to appear while the renderer can't take events, such as a
failure inside the renderer itself, would need a path around it; today such a
failure is dropped, because the place it would be reported is the thing that
failed (from crates/meow-cli/src/render.rs:56-65, high).

## Consequences

A warning appears in the transcript in order with the steps around it, and it
survives in scrollback after the process exits (from docs/design/0.3.0-tui.md
section 5.1, high). Under `--format json` the same rule sends diagnostics to
stderr, which [ADR-2804] records (from docs/spec/tui.md [R-TUI-033], high).

## How I will know it was realised

1. `a_diagnostic_does_not_tear_through_the_live_region` in
   crates/meow-ui/tests/renderers.rs passes: a `Note` at level `warn` is absent
   from the live region, which still shows the current tool.
2. `mise run check` passes, which runs clippy with the workspace's
   `print_stdout` and `print_stderr` denials.

## What this does not settle

- What the logging layer is: the tree has no `tracing` subscriber, and no
  `-v` or `-q` flag of the kind docs/design/0.3.0-tui.md section 2.3 lists, so
  diagnostics today are `Note` events and `ctx.out.warn` calls (from a search
  of crates/ for `tracing` and `verbose`, medium).
- Whether `meow-cli` may print while a live region is drawn: the lint exempts
  the crate as a whole (from Cargo.toml:32-35, high).
