---
id: ADR-2411
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2502, REQ-2503, REQ-2504, REQ-2505]
supersedes: []
---

# 2411. Markdown agents compose by inclusion only

## Decision

A markdown agent composes its system prompt from `include` alone, with no
substitution, conditional or loop (from docs/spec/starlark.md [R-STAR-054],
high).

## Why

A shared prompt is the real need, and substitution and conditionals are how a
configuration format turns into a bad programming language (from
docs/spec/starlark.md, Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Substitution and conditionals in markdown agents, as the v0.2.x `template` module offered through Go's `text/template` | One markdown file could serve several variants of an agent without Starlark (reasoned from v0.2.1:internal/core/starlark/module_template.go:11, low). | They turn a configuration format into a bad programming language (from docs/spec/starlark.md, Decisions, high). The `template` module was dropped because Starlark's `%` and string methods cover the need (from docs/design/0.3.0-starlark-api.md, section 10, high). |
| No composition at all, as the first draft of the spec had it | A markdown agent is one file that holds its whole prompt (reasoned from https://github.com/retran/meowg1k/pull/111, low). | Frontmatter "could not carry `include`, which it needs": the spec review listed it as a contradiction and fixed it (from https://github.com/retran/meowg1k/pull/111, high). |

Doing nothing isn't an option: markdown agents are new in v0.3.0, so there was
no prior form to keep (from docs/spec/starlark.md, Changes from v0.2.x, high).

## What it costs

An agent whose prompt needs a value or a branch has to be a Starlark agent
(from docs/spec/starlark.md, Decisions, high). An `include` entry may name only
a markdown file inside `.meow/` (from crates/meow-star/tests/agents.rs,
`an_include_may_not_leave_the_config_directory` and
`include_belongs_to_a_markdown_agent`, high).

## What would reverse it

If the workspaces the binary serves keep several markdown agents that differ
only in a value substitution would fill, the duplication would outweigh the
risk Why names (reasoned from docs/spec/starlark.md, Decisions, low).

## Consequences

- The system prompt is each included file in the order given, then the body,
  separated by a blank line (from crates/meow-star/src/markdown.rs:114-115 and
  crates/meow-star/tests/agents.rs, `include_prepends_each_prompt_in_order`,
  high).
- A Starlark agent can't carry `include` and builds its prompt with string
  operations (from crates/meow-star/src/agent.rs:212-217, high).
- The shipped `.meow/` shares its prompt fragments as `.meow/lib/*.md` files
  (from https://github.com/retran/meowg1k/pull/137, high).

## How I will know it was realised

The tests `include_prepends_each_prompt_in_order`,
`an_include_may_not_leave_the_config_directory` and
`include_belongs_to_a_markdown_agent` in crates/meow-star/tests/agents.rs pass
(high). No test asserts that a placeholder such as `{{name}}` in the body
reaches the model unchanged, so REQ-2505 rests on
crates/meow-star/src/markdown.rs containing no substitution code (reasoned
from crates/meow-star/src/markdown.rs, medium).

## What this does not settle

- Whether an included file may itself include another; it is read as plain
  text, so nothing in it is interpreted (reasoned from
  crates/meow-star/src/agent.rs:180-225, low).
- How a handler reaches a markdown agent, which ADR-0446 settles (from
  https://github.com/retran/meowg1k/pull/137, high).
