---
id: REQ-2544
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2544

`http` MUST refuse a URL that doesn't begin with `http://` or `https://`, naming
the URL, before it makes any request.

(from https://github.com/retran/meowg1k/pull/145 and
crates/meow-star/src/capability_http.rs:162, high)
