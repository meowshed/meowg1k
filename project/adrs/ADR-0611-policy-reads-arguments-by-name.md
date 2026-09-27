---
id: ADR-0611
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2050, REQ-2051, REQ-2052, REQ-2053]
supersedes: []
---

# 0611. The policy learns what a tool call touches from its argument names

## Decision

A tool call reaches the policy with the arguments named `path`, `file`, `paths`
and `files` as its paths, `command` as its command line and the host of `url` as
its host, and as a write when the tool's name contains `write`, `remove`,
`append` or `mkdir` (from https://github.com/meowshed/meowg1k/pull/127 and
crates/meow-star/src/run.rs:525-566, high). A tool that wants to be governed
uses those names (from https://github.com/meowshed/meowg1k/pull/127, high).

Once this is accepted, every call an agent makes is described this way before
the decision, and each path is resolved against the workspace root with
symbolic links followed, or joined lexically when it doesn't exist yet (from
crates/meow-star/src/run.rs:571-582, high).

## Why

The engine doesn't know which argument is a path and which is a command line,
and a declaration has no way to say so, so the name is the only signal left
(from crates/meow-star/src/run.rs:520-524 and
https://github.com/meowshed/meowg1k/pull/127, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no description of the call, as before #127 | No convention for a tool author to learn (reasoned from https://github.com/meowshed/meowg1k/pull/127, low) | `describe_call` was `None`, so no policy decision was ever taken (from https://github.com/meowshed/meowg1k/pull/127, high) |
| The declaration marks which argument is a path, a command or a URL | A tool is governed whatever its authors call its arguments (from https://github.com/meowshed/meowg1k/pull/127, high) | A declaration has no way to say it yet, and that would need a requirement of its own (from https://github.com/meowshed/meowg1k/pull/127, high) |

## What it costs

A tool whose path argument is named anything else, `target` for example,
reaches the policy with no paths, and a tool that writes under another name,
`save` for example, reaches it as a read (reasoned from
crates/meow-star/src/run.rs:527-540, low). A path that doesn't exist yet is
judged lexically and not with symbolic links resolved, the one place
REQ-2005's guarantee is weaker than it reads (from
https://github.com/meowshed/meowg1k/pull/127, high).

## What would reverse it

- A tool declaration gains a way to mark an argument as a path, a command or a
  URL (from https://github.com/meowshed/meowg1k/pull/127, high).

## Consequences

- A rule with a `paths` selector matches only calls whose tools use one of the
  four path names (reasoned from crates/meow-star/src/run.rs:540-555, low).
- ADR-0401's resolved paths come from this description (from
  docs/adrs/ADR-0401-policy-evaluation-gets-resolved-paths.md, medium).

## How I will know it was realised

1. `the_prompt_carries_everything_the_reader_needs` in
   crates/meow-star/tests/approval.rs passes; its rule asks only for a call
   with a `paths` match, so it passes only when `path` is read (reasoned from
   crates/meow-star/tests/approval.rs:164-190 and :274-300, low).
2. No test covers `command`, `url` or the write names; one per name, asserting
   the `Call` the policy receives, would.

## What this does not settle

- How a tool declares which of its arguments the policy should judge, which
  #127 says deserves a requirement (from
  https://github.com/meowshed/meowg1k/pull/127, high).
