---
id: REQ-2420
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2420

`re.match`, `re.find_all`, `re.replace`, and `re.split` MUST fail at the call
with the pattern and the reason when the pattern does not compile. (from
docs/spec/starlark.md [R-STAR-013], high)
