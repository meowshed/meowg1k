---
id: ADR-0459
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1222, REQ-1223]
supersedes: []
---

# 0459. The describing commands and `meow pkg` run without trust

## Decision

`check`, `doctor`, `policy show`, `models` and `providers` are exempt from the
trust gate (from https://github.com/meowshed/meowg1k/pull/154, high). `meow pkg`
is exempt too (from https://github.com/meowshed/meowg1k/pull/163, high).

Once this is accepted, `gate_on_trust` lets `check`, `doctor`, `policy`,
`models`, `providers`, `trust`, `version` and `pkg` through without asking, and
every other command that loads a workspace is asked (from
crates/meow-cli/src/wire.rs:155, high). `meow pkg` also reads a workspace whose
declared packages are absent, through `load_for_packages`, which no other
command does (from crates/meow-cli/src/lib.rs:79, high). No requirement names
the exemption: REQ-1222 and REQ-1223 say the first invocation asks, without an
exception for these commands (from docs/spec/auth.md [R-AUTH-030], high).

## Why

The describing commands are how a person decides whether to trust a workspace,
and requiring trust first makes the decision impossible to inform (from
https://github.com/meowshed/meowg1k/pull/154, high). `meow pkg` is how a person
sees what a workspace would pull in before deciding, and evaluating the
workspace to learn what to download reaches nothing (from
https://github.com/meowshed/meowg1k/pull/163, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Require trust before any command | One rule with no list of exceptions to keep current, and no command touches an untrusted `.meow/` beyond the question (reasoned from crates/meow-cli/src/wire.rs:155, low) | The decision to trust becomes impossible to inform (from https://github.com/meowshed/meowg1k/pull/154, high) |
| Keep `meow pkg` behind the gate, where it was before #163 (from https://github.com/meowshed/meowg1k/pull/163, medium) | An untrusted workspace can't make the binary download an archive from a source it names (reasoned from crates/meow-cli/src/wire.rs:165, low) | `meow pkg` is how a person sees what a workspace would pull in before deciding whether to trust it (from https://github.com/meowshed/meowg1k/pull/163, high) |
| Do nothing: no trust gate, as v0.2.x had | No friction: every command runs in a fresh clone (from docs/spec/auth.md Changes from v0.2.x, high) | A `.meow/` directory is executable code with tool access, and nothing about `git clone` asks whether you meant to run it (from https://github.com/meowshed/meowg1k/pull/154, high) |

## What it costs

The exempt commands load and inspect an untrusted `.meow/`, which `R-STAR-084`
says is safe (from https://github.com/meowshed/meowg1k/pull/154, high).
`meow pkg update` in an untrusted workspace downloads an archive from a source
the workspace names and writes a lockfile, so an untrusted workspace can make
the machine reach a host of its choosing (from crates/meow-cli/src/wire.rs:165,
high). The list is a `matches!` on subcommand names, so a new describing command
has to be added to it by hand (from crates/meow-cli/src/wire.rs:155, high).

## What would reverse it

A way found for an exempt command to run workspace code beyond declaration, for
example a module that `R-STAR-084` fails to refuse, would make loading unsafe
and the exemption with it (reasoned from
docs/requirements/REQ-1228-trust-prompt-runs-only-declaration.md, low).

## Consequences

- `meow check`, `meow doctor` and the rest never print "has not been run on this
  machine" (from crates/meow-cli/tests/trust.rs:240, high).
- `meow trust` is exempt, so agreeing is possible at all (from
  crates/meow-cli/src/wire.rs:163, high).
- `auth`, `init`, `completions` and `version` run before any workspace loads,
  so the gate never sees them (from crates/meow-cli/src/lib.rs:59, high).

## How I will know it was realised

`describing_a_workspace_needs_no_trust` in `crates/meow-cli/tests/trust.rs`
passes for `check`, `doctor`, `policy show`, `models` and `providers` (from
crates/meow-cli/tests/trust.rs:240, high). No test runs `meow pkg` in an
untrusted workspace: the helper in `crates/meow-cli/tests/pkg.rs` runs
`meow trust` before every command (from crates/meow-cli/tests/pkg.rs:24, high).

## What this does not settle

- Whether the exemption belongs in REQ-1222 and REQ-1223, which today require
  every first invocation to ask (from docs/spec/auth.md [R-AUTH-030], high).
- Whether `meow pkg` should reach the network for a workspace nobody has agreed
  to (from crates/meow-cli/src/wire.rs:165, high).
- Which other commands only describe, such as `meow session list` or
  `meow index`, which today need trust (from crates/meow-cli/src/wire.rs:155,
  high).
