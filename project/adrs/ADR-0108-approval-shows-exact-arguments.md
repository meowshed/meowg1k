---
id: ADR-0108
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2023, REQ-2024, REQ-2025, REQ-2840]
supersedes: []
---

# 0108. An approval prompt shows the exact call verbatim and names the rule that asked

## Decision

The approval prompt shows the exact command verbatim, never a summary, and names
the matching rule (from docs/design/0.3.0-tui.md section 7, high). `Prompt`
carries the tool, the arguments as the call's JSON with only sensitive values
redacted, the rule and where it was written, and the agent and step (from
crates/meow-policy/src/prompt.rs:32-78, high).

## Why

An approval prompt that paraphrases what it's approving is worse than no prompt
(from docs/design/0.3.0-tui.md section 7, high), because it asks for consent to
something the reader didn't see (from crates/meow-policy/src/prompt.rs:39-41,
high). Naming the rule tells the user why they're asked, so they can narrow the
policy afterwards (from docs/design/0.3.0-tui.md section 7, high) in place of
answering the same question every run (from
crates/meow-policy/src/prompt.rs:43-46, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no approval prompt, as in v0.2.x, which had no policy | Nothing interrupts a run (reasoned from docs/design/0.3.0-tui.md section 3, low) | v0.2.x had nowhere to put the one interaction that matters most (from docs/design/0.3.0-tui.md section 3, high) |
| Show a summary of the call | Short and readable for a large argument, such as the content of a file write (reasoned from crates/meow-policy/src/prompt.rs:66, low) | A prompt that paraphrases what it approves is worse than no prompt (from docs/design/0.3.0-tui.md section 7, high) |
| Show the call without the rule that asked | A shorter prompt (reasoned from docs/design/0.3.0-tui.md section 7, low) | The user couldn't tell why they were asked or which rule to narrow (from docs/design/0.3.0-tui.md section 7, high) |

## What it costs

The prompt shows the whole argument object, however long, because `Prompt::new`
renders it with no truncation (from crates/meow-policy/src/prompt.rs:66, high),
so a large write fills the transcript with its content (reasoned from
crates/meow-policy/src/prompt.rs:66, low). A value marked sensitive is hidden
even here, so the reader approves a call without seeing that one value (from
crates/meow-policy/tests/spec.rs:504-517, high).

## What would reverse it

- A class of tool whose verbatim arguments are too long to read in a terminal,
  shown by approvals given without reading, together with a summary that can be
  proven to say what the call does (reasoned from
  docs/design/0.3.0-tui.md section 7, low). Until a summary can be checked, the
  verbatim form stays.

## Consequences

Sensitive values are redacted in the prompt as they are in the transcript and in
every export (from docs/spec/policy.md [R-POLICY-060], high). The prompt is
committed to the transcript, so the question and its answer stay readable after
it closes (from docs/spec/tui.md [R-TUI-060] and
https://github.com/meowshed/meowg1k/pull/127, high). A rule records the file and
line it was written at, so the prompt can point there (from
crates/meow-policy/tests/spec.rs:502, high).

## How I will know it was realised

1. `an_approval_prompt_shows_the_call_verbatim_and_names_its_rule` in
   crates/meow-policy/tests/spec.rs passes (from
   crates/meow-policy/tests/spec.rs:473-518, high).
2. `the_prompt_shows_what_is_being_approved` in crates/meow-cli/src/ask.rs and
   `the_prompt_carries_everything_the_reader_needs` in
   crates/meow-star/tests/approval.rs pass (from
   crates/meow-cli/src/ask.rs:298-312 and
   crates/meow-star/tests/approval.rs:277, high).

## What this does not settle

- Which answers the prompt offers and how long `always` lasts, which ADR-0107
  records (from docs/spec/tui.md [R-TUI-061] and [R-TUI-062], high).
- Where the prompt is drawn: the live region can't grow, so it goes into the
  transcript (from docs/design/0.3.0-tui.md section 5.1, high).
