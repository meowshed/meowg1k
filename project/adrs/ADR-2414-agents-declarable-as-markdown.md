---
id: ADR-2414
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2496, REQ-2497, REQ-2498, REQ-2499]
supersedes: []
---

# 2414. An agent can be declared as a markdown file

## Decision

A `.md` file under `.meow/agents/` declares an agent whose frontmatter carries
its settings and whose body is its system prompt, and it produces the same value
as a Starlark agent (from docs/spec/starlark.md [R-STAR-050] [R-STAR-051],
high).

## Why

In v0.2.x every agent is a Starlark file, and the shipped `lib/agent.star`
exists only to hide the boilerplate that makes one work (from
docs/spec/starlark.md, Changes from v0.2.x, high). Most agents are a prompt, a
tool list and a budget, and requiring a `.star` file for that is a tax on the
common case (from docs/design/0.3.0-starlark-api.md, section 5.6, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Do nothing: keep every agent a Starlark file, as v0.2.x did | One declaration form, and a handler can hold the agent value directly (from https://github.com/retran/meowg1k/pull/137, high). | Each agent needs boilerplate that the shipped `lib/agent.star` exists only to hide (from docs/spec/starlark.md, Changes from v0.2.x, high). |
| Starlark only, with `meow.agent(...)` as a single declaration and no handler | It removes the boilerplate, since `meow.agent` covers what `agent.star` did, and keeps one form (from https://github.com/retran/meowg1k/pull/137, high). | The prompt still lives in a Starlark string literal and the common case still needs a `.star` file, which is the tax section 5.6 names (from docs/design/0.3.0-starlark-api.md, section 5.6, high). |

Neither the code, the history nor the design documents name a further option.

## What it costs

- Two declaration forms have to stay equal. Both go through one builder,
  `Fields::build`, which refuses `system` in frontmatter and `include` in
  Starlark (from crates/meow-star/src/agent.rs:186-217, high).
- A handler can't hold a markdown agent's value, because the agent is read
  after the file that would hold it; `meow.agent_named` gives it by name (from
  https://github.com/retran/meowg1k/pull/137, high).

## What would reverse it

If most agents in the workspaces the binary serves need control flow, the
common case this decision serves stops being common (reasoned from
docs/design/0.3.0-starlark-api.md, section 5.6, low).

## Consequences

- A markdown agent can go in another agent's `tools` list and on the command
  line, and produces the same result struct (from
  docs/design/0.3.0-starlark-api.md, section 5.6, high).
- About ten thousand lines of v0.2.x Starlark became three markdown agents, two
  prompt fragments and one file of about two hundred lines (from
  https://github.com/retran/meowg1k/pull/137, high); the three agents are in
  `.meow/agents/` (from `ls .meow/agents`, high).

## How I will know it was realised

The tests `a_markdown_file_declares_an_agent`,
`frontmatter_may_not_carry_the_system_prompt` and
`the_two_ways_of_declaring_an_agent_agree` in crates/meow-star/tests/agents.rs
pass (high).

## What this does not settle

- How a markdown agent composes its prompt, which ADR-2411 settles (from
  docs/spec/starlark.md, Decisions, high).
- How a handler reaches a markdown agent, which ADR-0446 settles (from
  https://github.com/retran/meowg1k/pull/137, high).
- Whether tools and commands can be markdown too; only agents can (from
  docs/design/0.3.0-starlark-api.md, section 5.6, medium).
