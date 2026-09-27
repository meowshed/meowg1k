---
id: ADR-0101
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2445]
supersedes: []
---

# 0101. Handlers stay imperative Starlark, and only concurrency is taken away from them

## Decision

A command's behaviour is a Starlark handler that can read, branch and format;
Starlark stays imperative, and only the concurrency is taken away from it (from
docs/design/0.3.0-architecture.md section 3.3, high). The handler receives a
context carrying only what is per-invocation, and imports every capability
(from docs/design/0.3.0-starlark-api.md section 1, high).

## Why

The useful automations are glue. Reading a diff, branching on whether tests pass
and formatting a report are imperative, and pushing them into declarative
configuration reinvents a worse language (from docs/design/0.3.0-architecture.md
section 3.3, high). This repository's own `review` command shows it: the
handler returns early on an empty diff, branches on whether the run finished,
loops over the findings and turns the verdict into an exit status (from
.meow/meow.star:133-165, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep the v0.2.x handler and its 26-member context | No migration of an existing `.meowg1k/` tree (reasoned from docs/design/0.3.0-starlark-api.md section 11, low) | The context was assembled by hand in two places that had already diverged, so a module added to one was missing inside an agent loop (from docs/spec/starlark.md Changes from v0.2.x, high) |
| Pure declarative configuration, no handlers | The source calls it attractive (from docs/design/0.3.0-architecture.md section 3.3, high); I read that as a tool set and a behaviour knowable before a run, which is what `meow policy explain` depends on (reasoned from docs/spec/starlark.md Decisions, low) | The useful automations are imperative glue, and pushing them into declarative configuration reinvents a worse language (from docs/design/0.3.0-architecture.md section 3.3, high) |
| Markdown agents only, with no Starlark at all | An agent is a paragraph and not a program (from docs/design/0.3.0-starlark-api.md section 1, high) | Automations that branch, fan out, post-process structured output or chain agents need Starlark (from docs/design/0.3.0-starlark-api.md section 5.6, high) |

## What it costs

A `.meow/` directory in a repository somebody just cloned is executable code
with tool access, so the first run in an untrusted workspace has to show what is
declared and ask (from docs/design/0.3.0-starlark-api.md section 4.1, high).
A handler's own calls, such as `fs.write` through `load("@std//fs", ...)`, are
not judged by the policy, because policy governs what a model decided and not
what a script did (from docs/spec/policy.md Decisions, high).

## What would reverse it

- Every command in this repository's `.meow/` could be written as a markdown
  agent with no handler, which would remove the evidence the Why section rests
  on (reasoned from .meow/meow.star:133-165, low).

## Consequences

An agent that needs no control flow is declared as markdown, and Starlark is for
automations that branch, fan out, post-process structured output or chain agents
(from docs/design/0.3.0-starlark-api.md section 5.6, high). The context has six
members plus `cancelled()`, and the other twenty v0.2.x members became imported
modules (from docs/design/0.3.0-starlark-api.md section 6.2, high).

## How I will know it was realised

1. `the_context_has_six_members_and_cancelled` in
   crates/meow-star/tests/running.rs passes (from
   crates/meow-star/tests/running.rs:375-399, high).
2. `meow review` in this repository runs its handler and exits non-zero when
   the reviewer's verdict is `hold` (from .meow/meow.star:165, medium).

## What this does not settle

- Which capabilities a handler may reach and how the tool set stays knowable:
  a handler can't declare a tool, which is a separate decision (from
  docs/spec/starlark.md Decisions, high).
- How concurrency is offered in place of what was taken away, which ADR-0102
  records (from docs/design/0.3.0-starlark-api.md section 5.4, high).
