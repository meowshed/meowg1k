---
id: REQ-3001
artifact: requirement
topic: architecture
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: static
---

# REQ-3001

`meow-core` MUST NOT perform input or output.

(from CLAUDE.md crate_boundaries, docs/design/0.3.0-architecture.md section 4
and crates/meow-core/Cargo.toml, which depends on `serde` and `serde_json`
alone, high)
