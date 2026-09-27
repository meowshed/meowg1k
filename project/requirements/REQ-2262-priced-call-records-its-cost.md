---
id: REQ-2262
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2262

A call against a model that declares a price MUST record its cost from the usage
the provider returned: prompt tokens at the input price plus completion tokens
at the output price, each price per million tokens.

(from https://github.com/retran/meowg1k/issues/168, high)
