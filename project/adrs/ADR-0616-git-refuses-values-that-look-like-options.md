---
id: ADR-0616
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2541]
supersedes: []
---

# 0616. `@std//git` refuses a caller's value that begins with a dash

## Decision

Every revision and path a caller gives `@std//git` is refused, with a message
naming it, when it begins with `-` (from
https://github.com/meowshed/meowg1k/pull/136 and
crates/meow-star/src/capability_git.rs:45-57, high). A commit message is the
exception: it goes after `--message` as a word of its own, so a message
beginning with a dash stays a message (from
crates/meow-star/src/capability_git.rs:176-189, high).

Once this is accepted, it works for `diff`, `log` and `show`'s revision and for
the paths of `diff` and `stage` (from
crates/meow-star/src/capability_git.rs:78, :112, :128 and :210, high).

## Why

A branch called `--upload-pack=rm` isn't a branch, `git` would read it as one
more option, and a name that came from a model must not turn one command into
another; the words never touch a shell, so the argument list is the one place
to check (from https://github.com/meowshed/meowg1k/pull/136, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: pass values through | A file or a revision whose name begins with a dash works (reasoned from crates/meow-star/src/capability_git.rs:50, low) | `git` reads such a value as an option, so a model's input could change the command (from https://github.com/meowshed/meowg1k/pull/136, high) |
| Rely on `--` to end the options | Paths beginning with a dash stay usable (reasoned from crates/meow-star/src/capability_git.rs:99, low) | A revision comes before `--` and can't be protected that way (reasoned from crates/meow-star/src/capability_git.rs:74-80, low) |

## What it costs

A file whose name begins with a dash can't be staged or diffed through the
module, even where `--` already protects it (from
crates/meow-star/src/capability_git.rs:99-101 and :210, high).

## What would reverse it

- A repository in use has files whose names begin with a dash and a handler
  has to stage them (reasoned from crates/meow-star/src/capability_git.rs:210,
  low).

## Consequences

- The check sits in one function that every caller-supplied value goes
  through (from crates/meow-star/src/capability_git.rs:45-49, high).

## How I will know it was realised

1. `git_refuses_a_value_that_looks_like_an_option` in
   crates/meow-star/tests/running.rs passes (from
   crates/meow-star/tests/running.rs:1160, high).

## What this does not settle

- Whether `shell.run` should check its words the same way; it doesn't (from
  crates/meow-star/src/capability.rs:230-260, high).
