---
id: ADR-1202
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1226, REQ-1229]
supersedes: []
---

# 1202. Trust is per workspace and asked again when its declarations change

## Decision

Trust is recorded per workspace path, and a workspace whose declarations changed
asks again, because what is recorded is what was shown. (from docs/spec/auth.md
[R-AUTH-032] [R-AUTH-034], high)

## Why

Trusting a path once and forever makes the question a formality, because the
repository at that path is not the repository the question was answered about.
Asking again on change is the smallest thing that keeps the answer meaningful.
(from docs/spec/auth.md Decisions, high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no trust question, as v0.2.x had | No friction: `meow` runs in a freshly cloned repository with no question asked (from docs/spec/auth.md Changes from v0.2.x, high) | It ran whatever the workspace's `.meow/` declared, and a `.meow/` is executable code with tool access (from docs/spec/auth.md Changes from v0.2.x and [R-AUTH-030], high) |
| Trust a path once and forever | One question per workspace, ever (reasoned from docs/spec/auth.md Decisions, low) | The question becomes a formality, because the repository at that path is no longer the one the question was answered about (from docs/spec/auth.md Decisions, high) |
| Ask again on every edit to `.meow/` | Catches every change, including a handler's body (reasoned from https://github.com/retran/meowg1k/pull/154, low) | The prompt becomes a formality people click through; a handler's body can't widen authority without widening a recorded declaration (from https://github.com/retran/meowg1k/pull/154 and crates/meow-cli/tests/trust.rs:180-203, high) |
| Key trust by content, not path | Moving a workspace keeps its trust (from https://github.com/retran/meowg1k/pull/154, high) | A repository trusted anywhere would be trusted everywhere (from https://github.com/retran/meowg1k/pull/154, high) |

## What it costs

Every workspace, including the one being developed, needs `meow trust` once,
with no per-machine opt-out. Moving a workspace loses its trust, and two
checkouts of one repository at two paths are two agreements. (from
https://github.com/retran/meowg1k/pull/154, high)

## What would reverse it

- A handler's body turns out to widen what a workspace may do without changing
  an agent, a package, a tool, a command or a policy rule, so that the
  fingerprint misses a change of authority (reasoned from
  crates/meow-cli/src/trust.rs:77-94, low).

## Consequences

The record is a SHA-256 fingerprint over the sorted agent, package, tool,
command and policy lines, which are the lines the person was shown; reordering
a declaration isn't a change (from crates/meow-cli/src/trust.rs:22-94, high).
The describing commands, `check`, `doctor`, `policy show`, `models` and
`providers`, run without trust, because they are how a person decides whether
to trust a workspace (from https://github.com/retran/meowg1k/pull/154, high).
Agreeing records trust and doesn't then run the command (from
https://github.com/retran/meowg1k/pull/154, high).

## How I will know it was realised

1. These tests in crates/meow-cli/tests/trust.rs pass:
   `widening_the_policy_withdraws_the_agreement`,
   `a_new_tool_withdraws_the_agreement`, `editing_a_handler_does_not_ask_again`,
   `adding_a_package_asks_again` and `trust_can_be_listed_and_withdrawn` (from
   crates/meow-cli/tests/trust.rs:124-300, high).

## What this does not settle

- A path that won't canonicalise, which is used as given (from
  https://github.com/retran/meowg1k/pull/154, high).
- A `--trust-everything` switch or a per-machine opt-out, which needs a
  requirement first (from https://github.com/retran/meowg1k/pull/154, high).
