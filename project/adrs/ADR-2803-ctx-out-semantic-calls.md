---
id: ADR-2803
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2832, REQ-2833, REQ-2834]
supersedes: []
---

# 2803. `ctx.out` offers ten semantic calls and no layout builtins

## Decision

`ctx.out` exposes ten semantic calls and none that positions the cursor, draws a
frame or paginates (from docs/spec/tui.md [R-TUI-040] [R-TUI-041], high).

Once this holds, a script says what a thing is, a warning, a finding or a table,
and each renderer decides how it looks (from docs/design/0.3.0-tui.md section
6, high). A script can no longer draw a panel, a banner, a tree, a progress bar
or a pager of its own (from docs/design/0.3.0-tui.md section 6, high).

## Why

v0.2.x's `ctx.ui` exposes 22 layout builtins, so presentation is decided in
userland and can't be fixed centrally (from docs/spec/tui.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: 22 layout builtins in `ctx.ui`, as in v0.2.x | A script controls exactly how its output looks, with `panel`, `table`, `tree`, `pager`, `divider`, `banner` and `progress_bar` (from docs/design/0.3.0-tui.md section 3, high) | Presentation is decided in userland and can't be fixed centrally (from docs/spec/tui.md, high); it was inconsistent across commands (from docs/design/0.3.0-tui.md section 3, high) |
| The ten semantic calls plus `progress_bar` and `progress` | A script could show progress through phases the engine doesn't see (reasoned from docs/design/0.3.0-tui.md section 6, low) | The live region shows progress already, and a script driving its own bar fights the renderer (from docs/design/0.3.0-tui.md section 6, high) |
| The ten semantic calls plus `pager` | A long result could be read a page at a time without leaving meowg1k (reasoned from docs/design/0.3.0-tui.md section 6, low) | Paging is `less`'s job, and a pager is wrong in a pipe (from docs/design/0.3.0-tui.md section 6, high) |

## What it costs

Every v0.2.x script that drew its own layout had to be rewritten, and
`.meowg1k/lib/ui_helpers.star`, 384 lines of stream-handler code, was deleted
(from docs/design/0.3.0-tui.md section 10, high). A script that wants a layout
the ten calls don't offer has no way to draw it, because `print` is refused and
points at `ctx.out` (from docs/spec/starlark.md [R-STAR-082], high).

## What would reverse it

A workspace that needs a presentation the ten calls can't express, shown by
scripts that hand-draw it through `ctx.out.write`, would reopen the list
(reasoned from docs/design/0.3.0-tui.md section 6, low).

## Consequences

`ctx.out` has exactly `write`, `markdown`, `note`, `warn`, `error`, `step`,
`table`, `diff`, `finding` and `json`, fixed as `Output::CALLS` (from
crates/meow-core/src/view.rs:269, high). A layout defect is fixed once in a
renderer, not in every script (from docs/spec/tui.md, Changes from v0.2.x,
high). The workspace this repository uses on itself calls seven of the ten
(from .meow/, `grep -o "ctx.out.[a-z]*"`, high).

## How I will know it was realised

1. `ctx_out_has_exactly_ten_calls` in crates/meow-star/tests/running.rs passes:
   `dir(ctx.out)` inside a handler lists the ten calls and nothing else.
2. `ctx_out_has_exactly_ten_calls` and `every_out_call_reaches_every_renderer`
   in crates/meow-ui/tests/renderers.rs pass.

## What this does not settle

- How each renderer draws each call, for example `markdown` rendered in the
  terminal and raw in plain output (from docs/design/0.3.0-tui.md section 6,
  high).
- What `ctx.ask` offers, which [REQ-2835] fixes (from docs/spec/tui.md
  [R-TUI-050], high).
- How the port behind `ctx.out` is shaped, which [ADR-0424] records (from
  docs/adrs/ADR-0424-ctx-out-port-takes-one-typed-event.md, high).
