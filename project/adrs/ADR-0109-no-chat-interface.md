---
id: ADR-0109
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2807, REQ-2850, REQ-2851]
supersedes: []
---

# 0109. meowg1k has no chat interface, and iteration is re-invocation

## Decision

meowg1k is driven entirely from the shell, with no conversational pane, no
message history to scroll and no prompt to type into (from
docs/design/0.3.0-tui.md section 1.1, high). Continuity, where it's wanted,
comes from sessions: `meow review --continue` and `meow session resume <id>`
(from docs/design/0.3.0-tui.md section 1.1, high).

## Why

A REPL owns the terminal for the duration, produces output that exists only
inside itself and can't appear in a pipeline (from docs/design/0.3.0-tui.md
section 1.1, high). Once the tool has a chat window, the pressure is to put
everything inside it, and the command line becomes a second-class way to reach
the same features (from docs/design/0.3.0-tui.md section 1.1, high). A run is a
task with a beginning, an end and a reason it stopped, and the loop doesn't wait
for a person (from docs/philosophy.md section 2, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A full-screen chat REPL, the shape the source says every terminal coding agent converges on | Follow-up questions without re-invoking a command, and a familiar shape for users of other agents (reasoned from docs/design/0.3.0-tui.md section 1.1, low) | It owns the terminal, keeps its output inside itself, can't sit in a pipeline, and turns the command line into a second-class surface (from docs/design/0.3.0-tui.md section 1.1, high) |
| An optional chat command beside the shell commands | Keeps every shell command and still offers a conversation to those who want one (reasoned from docs/design/0.3.0-tui.md section 1.1, low) | Once the tool has a chat window, the pressure is to put everything inside it (from docs/design/0.3.0-tui.md section 1.1, high) |

Doing nothing is this decision: v0.2.1 had no chat command either, only `auth`,
`init`, `version` and the Starlark commands (from `git ls-tree v0.2.1 cmd/`,
high), so the choice was whether to add one.

## What it costs

A user who wants to refine an answer re-invokes the command with `--continue`
or resumes the session, in place of typing a reply (from
docs/design/0.3.0-tui.md section 1.1, high). An agent that needs a person
mid-run gets only bounded prompts, `ctx.ask.text`, `confirm` and `select`, which
fail when there is no terminal (from docs/design/0.3.0-tui.md section 6, high).

## What would reverse it

- The pipeline stops being the way people use the tool: if `.meow/` commands in
  practice are run only by hand and never piped, redirected or run in CI, the
  reason the source gives stops applying (reasoned from
  docs/design/0.3.0-tui.md section 1.1, low).

## Consequences

- No alternate screen buffer, ever: the transcript goes to normal scrollback so
  it survives the process (from docs/design/0.3.0-tui.md section 1.1, high).
- The only interactive moments are an approval and a direct question from an
  agent, and both are single bounded prompts that degrade to a refusal when
  there is no terminal (from docs/design/0.3.0-tui.md section 1.1, high).
- `--continue` resumes the most recent session of the command being invoked and
  fails when that command has none, in place of starting a fresh run (from
  docs/philosophy.md section 2, high).
- The stop reason is the exit code, so a script can tell a finished run from a
  budget stop (from docs/design/0.3.0-tui.md section 2.4, high).

## How I will know it was realised

1. `continue_with_nothing_to_continue_fails`,
   `continue_adds_to_the_most_recent_session_of_that_command` and
   `continue_does_not_resume_another_commands_session` in
   crates/meow-cli/tests/surface.rs pass (from
   crates/meow-cli/tests/surface.rs:371-420, high).
2. `the_terminal_renderer_stays_on_the_main_screen` in
   crates/meow-ui/tests/renderers.rs passes (from
   crates/meow-ui/tests/renderers.rs:361-364, high).
3. The command line has no built-in that reads a free-form reply from the
   terminal in a loop; the built-ins are the ones section 2.2 lists (from
   docs/design/0.3.0-tui.md section 2.2, medium).

## What this does not settle

- How a resumed session seeds the engine with its own messages, which belongs to
  the session log (from https://github.com/retran/meowg1k/pull/127, high).
- Whether a new run starts a fresh session or continues one by default, which
  ADR-0111 records (from docs/design/0.3.0-tui.md section 2.3, high).
