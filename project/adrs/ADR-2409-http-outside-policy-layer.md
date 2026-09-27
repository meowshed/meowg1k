---
id: ADR-2409
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2447]
supersedes: []
---

# 2409. `http` is not behind the policy layer

## Decision

A handler calls `http` without a policy check, because policy governs what a
model decided and not what a script did (from docs/spec/starlark.md, Decisions,
high).

## Why

A handler is code the workspace's own author wrote, and gating it would put a
prompt in front of the author's own program (from docs/spec/starlark.md,
Decisions, high).

The decision applies ADR-2002, that policy governs what a model decided and not
what a script did, which the source cites as "the decision above" (from
docs/spec/starlark.md, Decisions, high; confirmed by
crates/meow-star/src/capability_http.rs:19-23, which cites the same reason).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Put `http` behind the policy layer | Every network call is judged, including one a handler makes with a URL a model supplied (reasoned from docs/spec/starlark.md, Decisions, low). | It puts a prompt in front of the author's own program (from docs/spec/starlark.md, Decisions, high). A check inside a capability module would also give permissions two places to be decided (from https://github.com/meowshed/meowg1k/pull/123, high). |
| A host allowlist inside the `http` module | A handler's call can't reach a host the workspace didn't name, whoever chose the URL (reasoned from https://github.com/meowshed/meowg1k/pull/145, low). | An allowlist belongs to the policy layer for model-driven calls, and a handler's own calls are the author's own program; a workspace that wants one writes a tool that checks the URL and gives the model that tool (from https://github.com/meowshed/meowg1k/pull/145, high). |

Doing nothing isn't an option: no Go file at v0.2.1 mentions a policy, so there
was no prior gate to keep (from `git grep -il policy v0.2.1 -- '*.go'`, high).
Neither the code, the history nor the design documents name a further option.

## What it costs

A workspace that gives a model a tool taking a URL from the model has handed the
model the network, and the rule for that tool has to reflect it (from
docs/spec/starlark.md, Decisions, high).

## What would reverse it

If the engine ever runs Starlark on a model's behalf other than through a tool
call, a model's decision could reach `http` without a rule matching it, and the
reason in Why would no longer hold (reasoned from docs/spec/policy.md,
Decisions, and ADR-2002, low).

## Consequences

- What a model decides still passes through policy: the model calls a tool, and
  the tool call is what a rule matches (from docs/spec/starlark.md, Decisions,
  high).
- The `http` builtins check only the phase and the URL scheme before sending, so
  `file://` is refused and nothing consults a policy (from
  crates/meow-star/src/capability_http.rs:156-167, high).

## How I will know it was realised

The tests `a_response_carries_what_the_server_sent` and
`a_404_is_a_response_rather_than_an_error` in crates/meow-star/tests/running.rs
declare no `meow.policy`, and their requests still reach the loopback server
(from crates/meow-star/tests/running.rs:349-352 and 2205-2270, high). A tool
call no rule matches is denied (ADR-0411), so a policy check on `http` would
make both tests fail (reasoned from ADR-0411, medium).

## What this does not settle

- Which rule a workspace should write for a tool that takes a URL from the
  model; the source leaves that to the workspace (from docs/spec/starlark.md,
  Decisions, high).
- The same question for `fs`, `shell` and `git`, which ADR-2002 settles for
  every capability (from docs/spec/policy.md, Decisions, high).
