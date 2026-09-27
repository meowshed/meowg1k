---
id: ADR-2001
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2015]
supersedes: []
---

# 2001. Network tools get a hosts selector now

## Decision

Network tools support a `hosts` selector matching the host of the request from
the first release of the policy layer, so a policy can allow one host without
allowing the network (from docs/spec/policy.md [R-POLICY-007], high).

Once this is accepted, a rule such as `allow http.get hosts ["docs.rs",
"*.crates.io"]` allows those hosts and a call to any other host falls through
to the default deny (from crates/meow-policy/tests/spec.rs:192-213, high). The
runtime finds the host only in an argument named `url`, so a tool that names
its address argument differently isn't narrowed by host (from
crates/meow-star/src/run.rs:521-524 and 559-564, high).

## Why

An all-or-nothing network rule forces the choice between no network and
unrestricted egress, and egress is where a prompt-injected agent does the most
damage (from docs/spec/policy.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Wait for an agent that needs a `hosts` selector | No selector is built or tested before a tool needs it (reasoned from docs/spec/policy.md, Decisions, low) | Until then only an all-or-nothing network rule exists, which forces the choice between no network and unrestricted egress (from docs/spec/policy.md, high) |
| Do nothing: an all-or-nothing network rule | A network rule is one line, matched on the tool name alone (reasoned from crates/meow-policy/src/policy.rs:297-340, low) | Egress is where a prompt-injected agent does the most damage (from docs/spec/policy.md, high) |
| Check the URL inside the tool the workspace writes, and give the model that tool | Needs nothing from the policy layer, which is what the `http` module's own pull request suggests for a handler's calls (from https://github.com/meowshed/meowg1k/pull/145, high) | A check inside each tool is a second enforcement point beside the engine, and two are two places for a decision to differ (reasoned from docs/spec/policy.md, Decisions, low) |

## What it costs

The runtime has to parse a host out of each call before the decision, and it
does so by a name convention on the arguments (from
crates/meow-star/src/run.rs:515-524 and 585-590, high). A selector is
checked against the tools that support it when the policy is built, so the
caller has to say which of its tools carry a host (from REQ-2009 and
crates/meow-policy/tests/spec.rs:134-150, which tests a `commands` selector,
medium).

## What would reverse it

- Evidence that the host judged before a call isn't the host the call reaches,
  for example through a redirect, so the selector gives confidence it can't
  back, would reopen the question (reasoned from
  crates/meow-star/src/capability_http.rs:45 and 187, low).

## Consequences

A policy can allow one host without allowing the network (from
docs/spec/policy.md [R-POLICY-007], high). A handler's own `http` calls aren't
narrowed by it, because the `http` module isn't behind the policy layer (from
crates/meow-star/src/capability_http.rs:19-23, high).

## How I will know it was realised

1. `a_host_selector_allows_one_host_without_allowing_the_network` in
   crates/meow-policy/tests/spec.rs passes: `docs.rs` and `static.crates.io`
   are allowed and `evil.example` is denied (from
   crates/meow-policy/tests/spec.rs:192-213, high).

## What this does not settle

- Redirects: the `http` module follows up to ten, and the selector judges only
  the host of the first request (from crates/meow-star/src/capability_http.rs:45
  and 187, and https://github.com/meowshed/meowg1k/pull/145, medium).
- How a tool whose address argument isn't named `url` declares its host (from
  crates/meow-star/src/run.rs:521-524, high).
