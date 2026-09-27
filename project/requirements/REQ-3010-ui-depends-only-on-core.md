---
id: REQ-3010
artifact: requirement
topic: architecture
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: static
---

# REQ-3010

`meow-ui` MUST NOT depend on a workspace crate other than `meow-core`, so a
renderer runs from a recorded log.

(from CLAUDE.md crate_boundaries, docs/design/0.3.0-architecture.md section 4
and crates/meow-ui/Cargo.toml, high)
