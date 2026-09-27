---
id: ADR-1200
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1200]
supersedes: []
---

# 1200. A credential resolves from three places in one fixed order

## Decision

A credential resolves from the declaration's `api_key`, then the global store,
then the provider's environment variable, and the first present and not empty
wins. (from docs/spec/auth.md [R-AUTH-001], high)

## Why

Credentials belong to the machine, and that is the whole point of the store.
(from docs/spec/auth.md Decisions, high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: every provider reads an environment variable, as v0.2.x did apart from Copilot | No file on disk holds a secret (reasoned from https://github.com/meowshed/meowg1k/issues/166, low) | A key in an environment variable is in every child process, in `ps` and often in shell history (from https://github.com/meowshed/meowg1k/issues/166, high) |
| Let a declaration say where its key comes from | A repository can pick the source that suits its own setup (reasoned from docs/spec/auth.md Decisions, low) | It moves the decision into the repository, and credentials belong to the machine (from docs/spec/auth.md Decisions and https://github.com/meowshed/meowg1k/pull/150, high) |

The specification, the code and the history name no third order.

## What it costs

A declared `api_key` beats a key stored with `meow auth login`, so a workspace
that sets one hides the store from the person running it (from
crates/meow-cli/src/wire.rs:1418-1425, high). An environment variable comes
last, so it can't override a stored key for one run; the code calls it the
escape hatch (from crates/meow-cli/src/wire.rs:1418-1450, medium). The store is
read on every provider build, which is a file read per run (from
https://github.com/meowshed/meowg1k/pull/153, high).

## What would reverse it

- People need an environment variable to override a stored key for one run,
  for example in a job that sets its own key on a machine with a stored one,
  and the fixed order blocks it (reasoned from
  crates/meow-cli/src/wire.rs:1418-1450, low).

## Consequences

One function, `credential`, resolves in this order, and a store that won't open
doesn't stop a run that has a key elsewhere (from
crates/meow-cli/src/wire.rs:1413-1450, high). When all three are empty, one
function names all three places and spells out the variable, for example
`$ANTHROPIC_API_KEY`, so the message can't list the places differently from
where the lookup looked (from crates/meow-cli/src/wire.rs:1452-1495, high).

## How I will know it was realised

1. These tests in crates/meow-cli/tests/auth.rs pass:
   `the_store_supplies_a_key_the_declaration_does_not`,
   `a_declared_key_is_preferred_to_the_stored_one` and
   `a_missing_credential_names_where_it_was_looked_for` (from
   crates/meow-cli/tests/auth.rs:131-295, high).
2. A test where the store and the environment variable both hold a key and the
   store's wins. No such test exists yet (from crates/meow-cli/tests/auth.rs,
   high).

## What this does not settle

- Where the store lives and what backs it. [R-AUTH-010] names
  `~/.meow/auth.json`, and issue 166 asks for the platform secret store with
  the file as a fallback (from docs/spec/auth.md [R-AUTH-010] and
  https://github.com/meowshed/meowg1k/issues/166, high).
- How an OAuth grant becomes the key a provider sends. The resolver hands over
  the stored access token and the provider does the exchange (from
  crates/meow-cli/src/wire.rs:1436-1441, high).
