---
id: ADR-0467
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1230, REQ-1231, REQ-1232, REQ-1233]
supersedes: []
---

# 0467. Credentials live in the operating system's secret store, with the file as the fallback

## Decision

meowg1k persists nothing itself and asks the operating system to: Keychain on
macOS, Credential Manager on Windows and Secret Service on Linux, holding a
credential in memory only for the call (from
https://github.com/meowshed/meowg1k/issues/166, high). `~/.meow/auth.json` becomes
the fallback rather than the design, and says so when it is used (from
https://github.com/meowshed/meowg1k/issues/166, high). The issue is open, so this
records a proposal nobody has approved.

Nothing of this works yet. The binary has no secret-store dependency, and
`meow auth` still reads and writes `~/.meow/auth.json`, created `0600`, refused
when wider and written atomically (from crates/meow-cli/src/auth.rs:4 and
crates/meow-cli/src/auth.rs:123, high; `grep -rn keyring crates Cargo.toml`
finds nothing). Every provider's key is read once when the engines are built and
held by the provider for the whole run, not only for the call (from
crates/meow-cli/src/wire.rs:1398 and crates/meow-cli/src/wire.rs:1405, high).

## Why

Principle 9 forbids storing or persisting user secrets, and `~/.meow/auth.json`
is a file on disk with secrets in it however carefully it is written (from
https://github.com/meowshed/meowg1k/issues/166, high). OAuth forces persistence of
something, because a device flow is pointless if the refresh token dies with the
process, so "hold nothing" can't mean "persist nothing anywhere" (from
https://github.com/meowshed/meowg1k/issues/166, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: a plaintext `~/.meow/auth.json`, `0600`, atomic and never printed, as #153 and #155 built | It works on every platform with no service running, and ten tests drive it through the binary (from crates/meow-cli/tests/auth.rs:69, high) | It is a file on disk with secrets in it, which the principle forbids (from https://github.com/meowshed/meowg1k/issues/166, high) |
| Environment variables | Nothing is persisted by meowg1k at all, and a run already falls back to the conventional variable (from crates/meow-cli/src/wire.rs:1445, high) | A key in `ANTHROPIC_API_KEY` is in every child process, in `ps` and often in shell history (from https://github.com/meowshed/meowg1k/issues/166, high) |
| A token file per provider, as v0.2.x kept `~/.config/meowg1k/copilot_token` | It persists one token for one provider, a narrower breach of the principle (from https://github.com/meowshed/meowg1k/issues/166 and `git grep copilot_token v0.2.1`, high) | It still leaves a secret in a file meowg1k wrote (from https://github.com/meowshed/meowg1k/issues/166, high) |

## What it costs

A new dependency that talks to three platform services, and a behaviour that
differs by platform, with the file kept as a fallback that still needs the
`0600`, refusal and atomic-write rules (from
https://github.com/meowshed/meowg1k/issues/166, high). The binary tests set
`MEOW_HOME` to a temporary directory to isolate the file; a secret store is per
user and has no such switch, so the tests would need a way to force the fallback
or a fake store (reasoned from crates/meow-cli/src/auth.rs:132, low).

## What would reverse it

A platform where no secret store runs is the common case, for example
containers and CI runners, so the fallback file becomes what most users get
(reasoned from https://github.com/meowshed/meowg1k/issues/166, low). An amendment
dropping principle 9's "persists no secret of its own" would also remove the
reason (from docs/vision.md goal 1, high).

## Consequences

`R-AUTH-010` needs amending to name the store, with the file as what happens
when there is none, and `R-AUTH-011` and `R-AUTH-012` become properties of the
fallback only (from https://github.com/meowshed/meowg1k/issues/166, high).
REQ-1204 to REQ-1208 carry those requirements today. `R-AUTH-001`, `R-AUTH-004`,
`R-AUTH-013` and `R-AUTH-014` are unaffected: the order, the module ban, the
commands and never printing a credential hold whatever the backing store is
(from https://github.com/meowshed/meowg1k/issues/166, high). The vision's first
goal stays "partly met" until this lands (from docs/vision.md goal 1, high).

## How I will know it was realised

On a machine with a secret store, `meow auth` stores a key and
`~/.meow/auth.json` doesn't exist afterwards; with the store unavailable, the
same command writes the file and prints a line naming it, as REQ-1233 asks
(reasoned from docs/requirements/REQ-1233-fallback-store-use-is-announced.md,
low). No test exists yet: the tests in `crates/meow-cli/tests/auth.rs` exercise
the file only (from crates/meow-cli/tests/auth.rs:211, high).

## What this does not settle

- Secrets a handler put in a prompt and that land in a session are a different
  problem, with `Redaction` on export as its answer (from
  https://github.com/meowshed/meowg1k/issues/166, high).
- How a key read from the store is held only for the call, when each provider
  is built with its key before the run starts (from
  crates/meow-cli/src/wire.rs:1405, high).
- How an existing `~/.meow/auth.json` moves into the store (reasoned from
  crates/meow-cli/src/auth.rs:146, low).
