---
id: REQ-1012
artifact: requirement
topic: agent
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1012

An axis left unset on the engine's `Budget` MUST be unbounded. A Starlark agent
declaration fills every axis before it reaches the engine, as REQ-2528 states
(from crates/meow-star/src/agent.rs, high).

(from docs/spec/agent.md [R-AGENT-012], high)
