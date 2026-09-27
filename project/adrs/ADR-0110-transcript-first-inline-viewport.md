---
id: ADR-0110
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2806, REQ-2807, REQ-2808, REQ-2809, REQ-2810, REQ-2813, REQ-2814, REQ-2815, REQ-2816]
supersedes: []
---

# 0110. The terminal renderer commits the transcript to scrollback and redraws only a small live region

## Decision

meowg1k doesn't take over the screen: the transcript is printed into normal
scrollback line by line as it is finalized, and only a small live region at the
bottom is redrawn (from docs/design/0.3.0-tui.md section 5, high). The mechanism
is `ratatui` with `Viewport::Inline(n)`, where `Terminal::insert_before` emits
finalized lines above the viewport (from docs/design/0.3.0-tui.md section 5.1,
high). `Tty::new` builds the terminal with `Viewport::Inline(RUN_ROWS)`, and
`Tty::commit` sends every finalized line through `insert_before` (from
crates/meow-ui/src/tty.rs:88-100 and 124-150, high).

## Why

When a full-screen alternate-buffer interface exits, the screen is restored and
the run is gone: no scrollback, nothing to paste into an issue and nothing for
`tee` to have captured (from docs/design/0.3.0-tui.md section 5, high). The
source says every agent run is something the user will want to read again (from
docs/design/0.3.0-tui.md section 5, high). v0.2.x ran three rendering stacks at
once, each owning the cursor, so which one won depended on timing (from
docs/design/0.3.0-tui.md section 3, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep v0.2.1's Bubble Tea program on the alternate screen | A full-screen layout the program controls, with mouse input, and the terminal restored cleanly on exit (from `git show v0.2.1:internal/adapters/output/tui/program.go`, lines 768-781, high) | On exit the screen is restored and the whole run is gone (from docs/design/0.3.0-tui.md section 5, high) |
| A full-screen alternate-buffer interface in Rust (`Viewport::Fullscreen`) | The whole screen to lay out, not three rows (reasoned from docs/design/0.3.0-tui.md section 5.1, low) | On exit the screen is restored and the whole run is gone (from docs/design/0.3.0-tui.md section 5, high) |
| A hand-rolled ANSI layer | No new dependency (reasoned from https://github.com/meowshed/meowg1k/pull/124, low) | `ratatui`'s inline viewport and `insert_before` are precisely the shape the design needs, and a hand-rolled layer would reimplement them (from docs/design/0.3.0-tui.md section 5.1 and https://github.com/meowshed/meowg1k/pull/124, high) |

## What it costs

`ratatui` fixes `n` when the terminal is constructed, so the live region is
three rows for the life of the process and an approval prompt goes into the
transcript (from docs/design/0.3.0-tui.md section 5.1, high). That cost forced
an amendment of [R-TUI-016] and [R-TUI-060] on 2026-09-20 (from
https://github.com/meowshed/meowg1k/pull/127, high). It also adds `ratatui` 0.30
as a dependency of `meow-ui` (from crates/meow-ui/Cargo.toml:12, high).

## What would reverse it

- `ratatui` drops `Viewport::Inline` or `Terminal::insert_before`, which is the
  shape the source chose it for (reasoned from docs/design/0.3.0-tui.md section
  5.1, low).
- The tool gains a surface that has to own the screen, which the no-chat
  decision in ADR-0109 rules out while it holds (reasoned from
  docs/design/0.3.0-tui.md section 1.1, low).

## Consequences

- Killing the process leaves valid output, because everything above the live
  region was already committed (from docs/design/0.3.0-tui.md section 5.1,
  high).
- A resize reflows only the live region (from docs/design/0.3.0-tui.md section
  5.1, high).
- The log writer uses `insert_before`, so a warning appears in the transcript in
  order (from docs/design/0.3.0-tui.md section 5.1, high); `Tty::commit`
  repaints the live region after each insert, because inserting scrolls it off
  its rows (from crates/meow-ui/src/tty.rs:124-139, high).
- `meow-ui` is generic over the backend, so tests drive `TestBackend` and the
  renderer is testable without a terminal (from
  https://github.com/meowshed/meowg1k/pull/124, high).

## How I will know it was realised

1. These tests in crates/meow-ui/tests/renderers.rs pass (from
   crates/meow-ui/tests/renderers.rs:361-495, high):
   `the_terminal_renderer_stays_on_the_main_screen`,
   `a_finalized_line_goes_into_scrollback`, `the_live_region_shows_progress`,
   `the_run_ends_with_its_stop_reason`, `a_resize_reflows_only_the_live_region`
   and `a_diagnostic_does_not_tear_through_the_live_region`.
2. Interrupting a run with Ctrl-C leaves the finalized transcript in the
   terminal's scrollback, ended by a line naming the `cancelled` stop reason
   (from docs/design/0.3.0-tui.md section 5.1, medium).

## What this does not settle

- What a piped or CI run prints: the `plain` renderer writes the same
  structure with no live region, no colour and no cursor movement (from
  docs/design/0.3.0-tui.md section 5.2, high).
- The height of the live region and where an approval prompt goes, which the
  fixed `n` settles as three rows and the transcript (from
  crates/meow-ui/src/tty.rs:19-25, high).
