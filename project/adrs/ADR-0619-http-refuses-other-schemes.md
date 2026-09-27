---
id: ADR-0619
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2544]
supersedes: []
---

# 0619. `http` refuses a URL whose scheme isn't `http` or `https`

## Decision

`http` refuses a URL that doesn't begin with `http://` or `https://` before it
makes any request, naming the URL (from
https://github.com/retran/meowg1k/pull/145 and
crates/meow-star/src/capability_http.rs:159-166, high).

Once this is accepted, it works for all four verbs, which go through the one
check (from crates/meow-star/src/capability_http.rs:159-166, medium).

## Why

A URL handed to a client that might follow it is how a fetch becomes a file
read: `file://` reaching a redirect follower (from
https://github.com/retran/meowg1k/pull/145, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: hand every URL to the client | No list of schemes to keep (reasoned from crates/meow-star/src/capability_http.rs:159, low) | A `file://` URL could become a file read outside the workspace's `fs` checks (from https://github.com/retran/meowg1k/pull/145, high) |
| A host allowlist in the module | Limits where a handler can reach, not only how (from https://github.com/retran/meowg1k/pull/145, high) | A host allowlist belongs to the policy layer for calls a model decided, and a handler's own calls are its author's program (from https://github.com/retran/meowg1k/pull/145 and ADR-2409, high) |

## What it costs

The check compares the prefix exactly, so `HTTPS://example.com` is refused
although the scheme is valid (reasoned from
crates/meow-star/src/capability_http.rs:162, low).

## What would reverse it

- A handler needs a scheme other than these two, such as `ws`, from the same
  module (reasoned from crates/meow-star/src/capability_http.rs:162, low).

## Consequences

- Redirects still follow up to ten, which is `reqwest`'s policy and not a
  promise of REQ-2447 to REQ-2455 (from
  https://github.com/retran/meowg1k/pull/145, high).

## How I will know it was realised

1. `a_scheme_that_is_not_http_is_refused` in crates/meow-star/tests/running.rs
   passes: `file:///etc/hosts` is refused by name (from
   crates/meow-star/tests/running.rs:2383, high).

## What this does not settle

- Whether a redirect to a scheme other than these two is refused by `reqwest`
  or reaches the module (reasoned from
  https://github.com/retran/meowg1k/pull/145, low).
