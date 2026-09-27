---
id: ADR-0457
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1209]
supersedes: []
---

# 0457. The key prompt echoes what is typed and says so

## Decision

The prompt doesn't suppress echo and says the key will be visible, and `--key`
exists for scripts that already hold the key somewhere safer (from
https://github.com/retran/meowg1k/pull/153, high).

Once this is accepted, `meow auth login <provider>` without `--key` prints
"Key for `<provider>` (it will be visible as you type): " on standard error and
reads one line, and with no terminal on standard input it refuses and exits 2
(from crates/meow-cli/src/ask.rs:260 and crates/meow-cli/src/wire.rs:101,
high). The key a person types stays on the screen and in the terminal's
scrollback (reasoned from crates/meow-cli/src/ask.rs:269, low).

## Why

Portable suppression needs a terminal crate and raw mode, and a version that
echoes on one platform and not another is worse than not promising it (from
https://github.com/retran/meowg1k/pull/153, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Suppress echo while the key is typed | Nobody looking at the screen or the scrollback sees the key (reasoned from crates/meow-cli/src/ask.rs:252, low) | It needs a terminal crate and raw mode, and suppression that works on one platform only is worse than not promising it (from https://github.com/retran/meowg1k/pull/153, high) |
| Do nothing: take the key only from `--key`, with no prompt | Nothing is echoed, because nothing is typed at a prompt (reasoned from crates/meow-cli/src/surface.rs:139, low) | `--key` puts the key in shell history, so it isn't the path a person is steered to (from https://github.com/retran/meowg1k/pull/153, high) |
| Read the key from piped standard input | A script could pass a key without a terminal and without shell history (reasoned from crates/meow-cli/src/ask.rs:263, low) | A pipeline that silently read a blank key would store one, so the prompt refuses when standard input isn't a terminal (from crates/meow-cli/src/ask.rs:258, high) |

## What it costs

`--key` puts the key in shell history, which is why it isn't the path a person
is steered to (from https://github.com/retran/meowg1k/pull/153, high). The typed
key is visible to anyone watching the screen and stays in scrollback (reasoned
from crates/meow-cli/src/ask.rs:269, low).

## What would reverse it

A terminal crate that suppresses echo on every supported platform being in the
build and used directly by `meow-cli`. `crossterm` 0.29 is already compiled in
through `ratatui`, though no code in the tree calls it (from Cargo.lock,
crates/meow-cli/Cargo.toml:33 and `grep -rn "crossterm::" crates`, high), so
the cost the pull request named is smaller than it said (reasoned from
crates/meow-cli/Cargo.toml:33, low).

## Consequences

- The prompt says what will happen, so the person decides whether to type the
  key where it can be seen (from crates/meow-cli/src/ask.rs:269, high).
- The key is trimmed, and an empty one is refused with exit 2 and "an empty key
  is not a credential" (from crates/meow-cli/src/wire.rs:112, high).

## How I will know it was realised

Every test in `crates/meow-cli/tests/auth.rs` logs in with `--key`, for example
`a_credential_can_be_stored_listed_and_removed` (from
crates/meow-cli/tests/auth.rs:69, high). No test drives the prompt itself or
checks its "visible as you type" wording; one would need a pseudo-terminal
(reasoned from crates/meow-cli/src/ask.rs:263, low).

## What this does not settle

- Whether echo suppression is added later, now that `crossterm` is in the build
  (from Cargo.lock, high).
- A way for a script to hand over a key without shell history or a terminal,
  such as a file or an environment variable read by `meow auth login` (reasoned
  from crates/meow-cli/src/ask.rs:263, low).
