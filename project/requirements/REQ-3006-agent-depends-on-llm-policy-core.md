---
id: REQ-3006
artifact: requirement
topic: architecture
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: static
---

# REQ-3006

`meow-agent` MUST NOT depend on a workspace crate other than `meow-llm`,
`meow-policy` and `meow-core`.

(from CLAUDE.md crate_boundaries, docs/design/0.3.0-architecture.md section 4
and crates/meow-agent/Cargo.toml, high)
