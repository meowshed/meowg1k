---
id: REQ-1412
artifact: requirement
topic: index
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1412

Chunks MUST overlap by a configured number of lines, so a definition split
across a boundary is retrievable from either side. Overlap is counted in lines
rather than tokens because boundaries are lines, and the two units cannot both
be exact.

(from docs/spec/index.md [R-INDEX-012], high)
