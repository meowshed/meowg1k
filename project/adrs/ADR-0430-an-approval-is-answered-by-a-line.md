---
id: ADR-0430
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2841]
supersedes: []
---

# 0430. An approval prompt is answered by a line, not a keypress

## Decision

The prompt is answered by a line, not a keypress (from
https://github.com/retran/meowg1k/pull/127, high).

Once this is accepted, the terminal approver puts the question up, reads one
line from standard input and takes the question down, and the first character
of the line picks the answer: `o` or `y` once, `a` always, `d` or `n` deny, `s`
or `q` stop (from crates/meow-cli/src/ask.rs:89 and
crates/meow-policy/src/prompt.rs:121, high). A line that starts with anything
else, an empty line and a closed standard input all deny (from
crates/meow-cli/src/ask.rs:176 to 187, high).

## Why

Raw mode would let `o` approve without Enter, and it would also put the terminal
into a mode a panic could leave it in; a security question is a reasonable place
to press Enter (from https://github.com/retran/meowg1k/pull/127, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: the engine treats `ask` as a refusal | Never blocks a run and never waits on a person (from https://github.com/retran/meowg1k/pull/127, high) | Right when nobody can be asked and wrong when somebody can (from https://github.com/retran/meowg1k/pull/127, high) |
| Raw mode, answering with one keypress | `o` approves without Enter (from https://github.com/retran/meowg1k/pull/127, high) | A panic could leave the terminal in raw mode (from https://github.com/retran/meowg1k/pull/127, high) |

## What it costs

Every approval takes a second key, Enter (from
https://github.com/retran/meowg1k/pull/127, high). Only the first character is
read, so a line such as `okay, but not that file` approves once (reasoned from
crates/meow-cli/src/ask.rs:181 and crates/meow-policy/src/prompt.rs:121, low).

## What would reverse it

- The binary starts using raw mode for another reason and restores the
  terminal on a panic. The risk that decided this would then be paid already;
  no crate enables raw mode today (from a search for `enable_raw_mode` and
  `raw_mode` under crates/, high).

## Consequences

- The binary needs no raw-mode handling and no panic hook to restore the
  terminal (from the same search, high).
- A key that isn't an answer is not an approval, and the prompt and the parser
  share one key table so they can't disagree (from
  crates/meow-cli/src/ask.rs tests `an_unrecognised_key_is_not_an_approval` and
  `all_four_answers_are_offered`, high).
- The secret prompt for `meow auth login` reads a line for the same reason, a
  portable alternative needs raw mode (from crates/meow-cli/src/ask.rs:252 to
  260, high).

## How I will know it was realised

1. crates/meow-cli/src/ask.rs tests `all_four_answers_are_offered` and
   `an_unrecognised_key_is_not_an_approval` pass (from those tests, high).
2. `meow --yes who` refuses with "the question was not answered", and `meow who
   < /dev/null` refuses with "needs a terminal" and doesn't block (from
   https://github.com/retran/meowg1k/pull/127, high).

## What this does not settle

- Where the prompt is drawn; the transcript placement is ADR-0110's decision
  (from https://github.com/retran/meowg1k/pull/127, high).
- Whether a secret typed at the key prompt is echoed; ADR-0457 decides that.
- How long "always" lasts; REQ-2842 fixes it at the current process.
