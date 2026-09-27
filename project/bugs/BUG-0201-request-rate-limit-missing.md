---
id: BUG-0201
artifact: bug
status: approved
severity: major
violates: REQ-1073
found: 2026-09-27
revised: 2026-09-27
issue: 167
---

<!-- Written to the writing standard meow-prose ships: lead with the answer,
give each rule its reason in the same sentence, and show the failing case. -->

# A workspace can't declare a request ceiling, so nothing limits request rate

## Reproduction

At `d67ae50`:

1. Run `grep -r requests_per crates/`.

It returns no match: no declaration, counter or refusal exists (from
https://github.com/meowshed/meowg1k/issues/167, high).

## What the system does

A workspace can bound steps, tokens and wall time for one run, and nothing
limits how many requests it makes over time. The Go implementation kept a
per-minute and per-day ceiling in its database, and the rewrite dropped it (from
https://github.com/meowshed/meowg1k/issues/167, high).

## What it should do, and why

REQ-1073: a workspace MUST be able to declare request ceilings per minute and
per day, with REQ-1074 to REQ-1076 requiring the count to be shared across
invocations through the workspace database, an over-ceiling request refused
before the call, and a refusal that names the limit and when it frees up. The
vision's sixth goal, predictable cost, names a hard ceiling on usage.

## Triage

It enters in the rewrite, which carried over the cache and dropped the limiter.
Major: a requirement's whole behaviour is missing, and a loop over `meow` spends
without bound.

## Closed by

A test that runs two processes against one workspace and shows the second
refused by what the first spent, as #167 asks, because a single-process test
can't tell a shared limiter from a per-process one. Issue #167 closes with the
fix.
