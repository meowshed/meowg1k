---
id: REQ-2528
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2528

An agent declaration's `budget` that omits an axis MUST take that axis from the
default budget, whether the agent is declared in Starlark or in markdown
frontmatter.

(from https://github.com/meowshed/meowg1k/pull/122 and
crates/meow-star/src/agent.rs:45, high)
