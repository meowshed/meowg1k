---
id: ADR-1201
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1211, REQ-1212]
supersedes: []
---

# 1201. No command shows a stored credential, in full or in part

## Decision

No command prints a credential, in full or in part; `meow auth list` says that a
credential is present and when it was stored. (from docs/spec/auth.md
[R-AUTH-013] [R-AUTH-014], high)

## Why

Showing a stored credential puts a secret on a terminal, into scrollback and
into whatever recorded the session. The store is a file the user owns, and `cat`
is available to them. (from docs/spec/auth.md Decisions, high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no listing at all, as v0.2.x had | Nothing to leak, because nothing is listed (from docs/spec/auth.md Changes from v0.2.x, high) | A person can't tell which providers have a credential; [R-AUTH-013] requires `meow auth list` to say so (from docs/spec/auth.md [R-AUTH-013], high) |
| Show the user what they stored | It answers a reasonable request (from docs/spec/auth.md Decisions, high) | It puts a secret on a terminal, into scrollback and into whatever recorded the session (from docs/spec/auth.md Decisions, high) |
| Show part of the credential, such as its last characters | Lets a person tell two keys apart (reasoned from docs/spec/auth.md [R-AUTH-014], low) | The requirement forbids printing a credential "in part" as well as in full (from docs/spec/auth.md [R-AUTH-014], high) |

## What it costs

A user who wants to see a stored credential reads `~/.meow/auth.json` with `cat`
in place of a `meow` command. (from docs/spec/auth.md Decisions, medium) That
remedy holds only while the store is a file: issue 166 moves credentials to the
platform secret store, where `cat` no longer reaches them (from
https://github.com/meowshed/meowg1k/issues/166, high).

## What would reverse it

- People can't tell whether the stored key is the one they meant, for example
  after rotating a key, and the time of storing the listing shows doesn't
  answer it (reasoned from crates/meow-cli/src/auth.rs:87-104, low).

## Consequences

`Credential::describe` returns only a static kind name, and `stored` returns
the time it was stored, so nothing a listing reads can return part of a secret
(from crates/meow-cli/src/auth.rs:87-104, high). A person typing a key sees it
echoed, because portable echo suppression needs a terminal crate and raw mode,
and the prompt says the key will be visible (from
https://github.com/meowshed/meowg1k/pull/153, high).

## How I will know it was realised

1. `nothing_ever_prints_the_credential` in crates/meow-cli/tests/auth.rs
   passes; it checks five commands, not only `auth list` (from
   crates/meow-cli/tests/auth.rs:99-130 and
   https://github.com/meowshed/meowg1k/pull/153, high).
2. `a_credential_can_be_stored_listed_and_removed` in the same file passes
   (from crates/meow-cli/tests/auth.rs:67-98, high).

## What this does not settle

- A secret a handler put into a prompt, which lands in the session log. That is
  a separate problem with a separate answer, redaction on export (from
  https://github.com/meowshed/meowg1k/issues/166, high).
- The `--key` flag of `meow auth login`, which puts the key in shell history
  and exists for scripts that already hold it somewhere safer (from
  https://github.com/meowshed/meowg1k/pull/153, high).
