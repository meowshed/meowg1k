---
id: ADR-0429
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1201]
supersedes: []
---

# 0429. A provider with no credential is skipped at startup and fails when used

## Decision

A provider with no credential is skipped rather than refused at startup, and the
failure arrives when something asks for that provider, naming it (from
https://github.com/meowshed/meowg1k/pull/126, high).

Once this is accepted, a run whose providers all have a credential starts, and
`meow doctor` lists the providers with none and exits 6 (from
crates/meow-cli/src/wire.rs:501 and crates/meow-cli/tests/surface.rs
`a_missing_credential_exits_six`, high). The index path names all three places
it looked (from crates/meow-cli/tests/auth.rs
`a_missing_credential_names_where_it_was_looked_for`, high). The agent path
doesn't: a skipped provider has no engine, and asking for it reports
``provider `anthropic` is not declared`` and exits 9 (from
crates/meow-star/src/run.rs:263, and reproduced with `target/debug/meow`
built 2026-09-20, after which neither run.rs nor wire.rs changed, high).

## Why

A workspace may declare three providers and a run may need one, so failing on a
missing key for an unused provider would be wrong (from
https://github.com/meowshed/meowg1k/pull/126, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Refuse at startup when any declared provider has no credential | Fails before any model call spends money, and reports every missing key in one place (reasoned from crates/meow-cli/src/wire.rs:1394, low) | A run that needs one provider would fail on a key for a provider it doesn't use (from https://github.com/meowshed/meowg1k/pull/126, high) |
| Build each provider only when a model on it is first asked for, as v0.2.x deferred building its model services until a command ran (from `git show v0.2.1:cmd/starlark.go`, lines 62 to 64, high) | Builds nothing a run doesn't use, and the failure could name the three places at the point of use (reasoned, low) | The runtime takes a fixed map of engines built by the binary, and `meow-star` can't reach the credential store to build one later (reasoned from crates/meow-cli/src/wire.rs:1394 and REQ-1203, low) |

Doing nothing isn't an option here: before this change no binary existed to
decide it (from https://github.com/meowshed/meowg1k/pull/126, high).

## What it costs

The failure moves from startup to the middle of a run, after any earlier model
call on another provider has spent tokens (reasoned from
crates/meow-cli/src/wire.rs:1394, low). The failure is also reported by a
different path than the one that built the engines, and on the agent path it
reports the wrong cause today (from crates/meow-star/src/run.rs:263, high).

## What would reverse it

- The commands a workspace declares routinely use every provider it declares,
  so skipping one never lets a run finish that refusing would have stopped
  (reasoned from https://github.com/meowshed/meowg1k/pull/126, low).

## Consequences

- `meow doctor` has to report missing credentials itself, because startup no
  longer does (from crates/meow-cli/src/wire.rs:501, high).
- `meow check` builds no provider and reads no credential, so it works on a
  machine with no keys (from crates/meow-cli/tests/auth.rs
  `declaring_does_not_read_the_credential_store`, high).
- Every caller that looks a provider up has to turn "no engine" into the
  three-places message; `served_by` does it for the index and the agent path
  doesn't (from crates/meow-cli/src/wire.rs:1469 and
  crates/meow-star/src/run.rs:263, high).

## How I will know it was realised

1. A workspace declares two providers, one with a key and one without, and a
   command on the keyed provider exits 0 (reasoned from
   crates/meow-cli/src/wire.rs:1394, low; no test covers it today).
2. A command whose agent uses the provider with no key exits 6 and its message
   names `api_key`, `meow auth login` and the environment variable, as
   crates/meow-cli/tests/auth.rs
   `a_missing_credential_names_where_it_was_looked_for` already asserts for
   `meow index build` (from that test, high). This fails today.

## What this does not settle

- What happens when a credential is present and the provider rejects it; that
  is a provider failure, not a missing credential (reasoned from
  crates/meow-cli/src/wire.rs:1417, low).
- How an OAuth grant that has expired is renewed; ADR-0462 covers a refused
  renewal.
