---
id: REQ-3004
artifact: requirement
topic: architecture
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: static
---

# REQ-3004

`meow-llm` MUST NOT depend on a workspace crate other than `meow-core`.

(from CLAUDE.md crate_boundaries, docs/design/0.3.0-architecture.md section 4
and crates/meow-llm/Cargo.toml, high)
