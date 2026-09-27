---
id: REQ-1424
artifact: requirement
topic: index
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1424

The content hash MUST cover the chunking parameters as well as the content, so
changing chunk size or overlap does not leave stale chunks that look current.

(from docs/spec/index.md [R-INDEX-030], high)
