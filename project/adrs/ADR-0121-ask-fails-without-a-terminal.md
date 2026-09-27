---
id: ADR-0121
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2836, REQ-2837]
supersedes: []
---

# 0121. `ctx.ask` fails instead of blocking when nobody can answer

## Decision

`ctx.ask.text`, `confirm` and `select` fail loudly when stdin isn't a terminal
or `--yes` is set (from docs/design/0.3.0-tui.md section 6, high). The binary
decides once per run whether anybody can be asked: `--yes` means unattended even
in a terminal, and otherwise stdin decides (from
crates/meow-cli/src/ask.rs:31-45, high). With no terminal the call fails with
an error that names it (from crates/meow-star/src/port.rs:46-50, high).

## Why

An unattended run must never hang on a prompt (from docs/design/0.3.0-tui.md
section 6, high). Both interactive moments degrade to a refusal when there is no
terminal (from docs/design/0.3.0-tui.md section 1.1, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: read a line from stdin with no terminal check, as v0.2.x's `ui.Confirm` did | A script or a test can pipe answers into stdin (reasoned from v0.2.1:internal/ui/input.go:15-44, low) | An unattended run hangs on the prompt while stdin stays open (from docs/design/0.3.0-tui.md section 6, high); v0.2.x also printed the prompt with `fmt.Print`, straight through the live frame (from docs/design/0.3.0-tui.md section 3, high) |
| Take the call's `default` when nobody can answer | The run carries on, and a handler that supplied a default gets it (reasoned from docs/design/0.3.0-tui.md section 6, low) | Silently taking the default turns an unattended run into a wrong answer (from crates/meow-star/src/port.rs:41-45, high) |
| Block waiting for an answer | A person who arrives late can still answer (reasoned from docs/design/0.3.0-tui.md section 6, low) | An unattended run would hang on the prompt (from docs/design/0.3.0-tui.md section 6, high) |

## What it costs

A handler that asks can't run in a pipeline or in CI at all, even when it
offers a default, so an automation meant to run unattended must avoid
`ctx.ask` (from crates/meow-cli/tests/surface.rs:320-343, high). `--yes` makes
every question fail too, since it means unattended and never means yes (from
crates/meow-cli/src/ask.rs:31-55, high).

## What would reverse it

- A requirement to answer `ctx.ask` from a file or from stdin in a pipeline,
  which REQ-2836 forbids today (reasoned from
  docs/requirements/REQ-2836-ctx-ask-fails-without-terminal.md, low).

## Consequences

The same rule holds for an approval: an `ask` decision becomes a denial when
there is no terminal or `--yes` is set, so neither route blocks and neither
becomes more permissive (from crates/meow-cli/src/ask.rs:15-20, high). A
handler that fails this way exits non-zero with `needs a terminal` on stderr
(from crates/meow-cli/tests/surface.rs:320-343, high).

## How I will know it was realised

1. `asking_with_no_terminal_fails_rather_than_waiting` in
   crates/meow-cli/tests/surface.rs passes against the built binary with a
   closed stdin, and a hang would hang the suite (from
   crates/meow-cli/tests/surface.rs:320, high).
2. `yes_is_unattended_and_is_not_consent` and
   `no_terminal_refuses_and_names_the_call` in crates/meow-cli/src/ask.rs pass
   (from crates/meow-cli/src/ask.rs:338 and 350, high).

## What this does not settle

- How a prompt is drawn when there is a terminal: it is committed to the
  transcript, because the inline viewport can't grow, which is REQ-2838 and
  REQ-2839 (from docs/design/0.3.0-plan.md M8, high).
- Whether a handler can find out in advance that a question is possible:
  `ctx.ask` holds only `text`, `confirm` and `select`, so a handler learns only
  by failing (from crates/meow-star/src/context.rs:259-300, high).
