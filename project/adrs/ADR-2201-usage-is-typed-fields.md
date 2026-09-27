---
id: ADR-2201
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2215]
supersedes: []
---

# 2201. A Usage event carries tokens and cost as typed fields

## Decision

A `Usage` event carries prompt tokens, completion tokens, cached prompt tokens
and cost as separate typed fields (from docs/spec/session.md [R-SESSION-020],
high).

Once this holds, a session's totals are a SQL sum over a typed `usage` table,
and cached tokens are visible per call and per session (from
crates/meow-session/src/log.rs:149-158 and crates/meow-store/src/rows.rs:172,
high). Cached tokens and cost are optional, so a provider that doesn't report
caching and a model with no price read as unknown, never as zero (from
crates/meow-core/src/usage.rs:17-31, high).

## Why

v0.2.x wrote usage as metadata strings such as `llm_tokens_0_1762...` =
`"prompt=91,completion=12,total=103"`, which can't be summed without parsing
twice and carried no cached-token count (from docs/spec/session.md, high).
Prompt-cache hits are most of the cost difference between a well-built agent
and a badly built one, and a design that can't show them can't be tuned (from
docs/design/0.3.0-sessions.md section 3, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Usage as metadata strings, as in v0.2.x (the do-nothing option) | Any new counter goes into the one flat metadata namespace with no schema change (reasoned from docs/design/0.3.0-sessions.md section 2, low) | The strings can't be summed without parsing twice, and they carried no cached-token count (from docs/spec/session.md, high) |
| Cost as a floating-point or decimal currency field, as the design sketched with `cost_usd: Decimal` | Reads directly as money, with no unit conversion (from docs/design/0.3.0-sessions.md section 3, medium) | Integer millionths are used so that summing a thousand events doesn't drift (from crates/meow-core/src/usage.rs:24-31, high) |
| Typed fields only in the serialised event body, with no side table | One place holds each value (reasoned from crates/meow-session/src/log.rs:141-173, low) | Totals would decode every event on every read; the typed side table exists for what is read on every rebuild (from crates/meow-session/src/log.rs:149, medium) |

## What it costs

A new counter needs a change to the `Usage` type, the `usage` table and a
schema migration, where v0.2.x added a string (reasoned from
crates/meow-store/src/migrations.rs:89 and crates/meow-core/src/usage.rs, low).
Each value is stored twice, in the event body and in the side table, and fork
copies both (from crates/meow-session/src/log.rs:141-158 and
crates/meow-store/src/session.rs:162-167, high).

## What would reverse it

- A provider reporting a usage quantity that has to be summed and that the four
  fields can't carry, such as a separately priced reasoning-token count
  (reasoned from crates/meow-core/src/usage.rs, low).

## Consequences

- Totals include children, and a total including one unpriced call has no cost
  (from crates/meow-session/src/log.rs:330-342 and
  crates/meow-core/src/usage.rs:40-57, high).
- Markdown export prints the totals and says "cost unknown" when a cost is
  absent (from crates/meow-session/src/export.rs:79-101, high).
- `meow session list` shows each session's token total (from
  crates/meow-cli/src/session.rs:173-186, high).

## How I will know it was realised

1. `usage_is_typed_and_cached_tokens_are_their_own_field` in
   crates/meow-session/tests/spec.rs passes (from
   crates/meow-session/tests/spec.rs:277-299, high).
2. `an_unpriced_call_records_no_cost_rather_than_a_free_one` and
   `a_parent_reports_what_its_children_spent` in the same file pass (from
   crates/meow-session/tests/spec.rs:301-365, high).

## What this does not settle

- When cost is computed and from which price table, which [R-SESSION-021]
  settles (from docs/spec/session.md, high).
- How a provider reports cached tokens, which the LLM specification settles
  (from crates/meow-core/src/usage.rs:17-23, medium).
- What currency the cost is in: the type says "a unit of currency" and names
  none (from crates/meow-core/src/usage.rs:24, high).
