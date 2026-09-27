---
id: ADR-2800
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2817, REQ-2818, REQ-2819, REQ-2820, REQ-2838, REQ-2839]
supersedes: []
---

# 2800. The live region is a fixed three rows, and an approval prompt goes into the transcript

## Decision

The live region is three rows high and never changes height, and an approval
prompt is committed to the transcript instead of drawn in the live region (from
docs/spec/tui.md [R-TUI-016], high).

Once this holds, a prompt appears below everything already in scrollback, stays
readable after it is answered, and the live region says an answer is awaited
while it is open (from crates/meow-ui/src/tty.rs:401-419, high). A prompt can't
be taken back once it is in scrollback, and it scrolls up with the transcript
like any other line, which is why the live region has to carry the waiting
notice (from crates/meow-ui/src/tty.rs:55-60, high).

## Why

A region that grows with content reflows the terminal while you're reading it,
and one that changes height for a prompt reflows it twice at the moment you're
reading the command you're asked to approve (from docs/spec/tui.md, high).

`ratatui` fixes an inline viewport's height when the terminal is constructed and
offers no way to take the backend back, so changing the height means rebuilding
over a backend that can't be recovered (from docs/spec/tui.md [R-TUI-016]
amendment of 2026-09-20, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A region that grows with content | It shows everything the run has to say without truncating, where three rows can hold only the tool, the step and the budget (reasoned from docs/design/0.3.0-tui.md section 5, low) | It reflows the terminal while you're reading it (from docs/spec/tui.md, high) |
| Do nothing: keep the approved [R-TUI-016] of 2026-09-19, a second larger fixed height while a prompt is open | The prompt sits in a bordered box pinned to the bottom of the screen and leaves no trace in the transcript once answered, as docs/design/0.3.0-tui.md section 7 draws it (from docs/design/0.3.0-tui.md section 7, high); it was what PR #124 built as `Tty::prompting` (from https://github.com/meowshed/meowg1k/pull/124, high) | It reflows the terminal twice as you read the command you're approving (from docs/spec/tui.md, high), and a permission decision that disappears when the prompt closes can't be checked afterwards (from https://github.com/meowshed/meowg1k/pull/127, high) |
| Rebuild the terminal with a taller inline viewport each time a prompt opens | It would meet the original [R-TUI-016] to the letter (reasoned from docs/spec/tui.md [R-TUI-016] amendment, low) | `ratatui` offers no way to take the backend back, so the rebuild would be over a backend that can't be recovered (from docs/spec/tui.md [R-TUI-016] amendment of 2026-09-20, high) |

## What it costs

Every prompt's text stays in the transcript for good, so a run with many
approvals leaves a longer scrollback (reasoned from
crates/meow-ui/src/tty.rs:401-414, low). The live region can show no more than
three rows at any time, whatever the run is doing (from
crates/meow-ui/src/tty.rs:25, high). The prompt is answered by typing a line and
Enter, not by a single keypress, because raw mode could leave the terminal
broken after a panic (from https://github.com/meowshed/meowg1k/pull/127, high).

## What would reverse it

A requirement to show more in the live region at once than three rows hold,
such as one line for each concurrent sub-agent, would reopen the height;
`ratatui` gaining a way to resize an inline viewport would not on its own,
because the committed prompt is kept for its own merit (reasoned from
docs/spec/tui.md [R-TUI-016] amendment of 2026-09-20, low).

## Consequences

The prompt costs no reflow, and the question and its answer stay in the
transcript where somebody can find them afterwards (from docs/spec/tui.md,
high). The requirement that a prompt overwrite nothing and stay readable after
it's answered was amended alongside, as [REQ-2838] and [REQ-2839] state (from
docs/spec/tui.md [R-TUI-060], high). The plain and JSON renderers have no live
region, so their prompt goes to stderr and never into a stdout that may be
carrying a result (from crates/meow-ui/src/lib.rs:40-56, high).

## How I will know it was realised

1. `the_live_region_keeps_one_height_and_says_when_it_is_waiting` in
   crates/meow-ui/tests/renderers.rs passes: the live area is `RUN_ROWS` (3)
   rows before, during and after a prompt, and reads "waiting for your answer"
   while one is open.
2. `a_prompt_does_not_overwrite_the_transcript` in the same file passes: the
   prompt text lands below the committed line, not in the live region, and is
   still on screen after `prompt_close`.

## What this does not settle

- What the prompt shows and which answers it offers, which [REQ-2840] and
  [REQ-2841] fix (from docs/spec/tui.md [R-TUI-061], high).
- What the live region shows while no prompt is open, which [REQ-2811] fixes
  (from docs/spec/tui.md [R-TUI-012], high).
- Whether a line committed by another thread while a prompt is open lands
  between the prompt and its answer: the renderer sits behind one mutex, but
  nothing holds it for the whole prompt (reasoned from
  crates/meow-cli/src/render.rs:15-46, low).
