---
id: ADR-0408
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2218, REQ-2219]
supersedes: []
---

# 0408. A total that includes an unpriced call reports no cost

## Decision

An unpriced model records an absent cost rather than zero, and a total including
one unpriced call reports no cost at all (from
https://github.com/retran/meowg1k/pull/116, high). `Usage::cost_micros` is an
`Option<u64>` in millionths of a unit of currency, and `Usage::add` gives `None`
unless both sides carry a cost (from crates/meow-core/src/usage.rs:24-58, high).

Once this is accepted, a session's totals, its children included, report no
cost as soon as one call was unpriced, and the plain renderer and the export
print "cost unknown" for it (from crates/meow-ui/src/plain.rs:214-219 and
crates/meow-session/src/export.rs:99-101, high). No price table exists yet:
every provider sets `cost_micros: None`, so today every total reads "cost
unknown" (from crates/meow-llm/src/anthropic.rs:255,
crates/meow-llm/src/gemini.rs:281 and crates/meow-llm/src/lib.rs:66, high).

## Why

A number that silently omits part of the spend is worse than no number (from
https://github.com/retran/meowg1k/pull/116, high). Recording an unpriced call as
zero would make an unpriced model read as a free one (from
docs/spec/session.md [R-SESSION-021], high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: the design's `cost_usd: Decimal`, a number on every `Usage` event | Every total is a number, and summing needs no case for a missing value (from docs/design/0.3.0-sessions.md:56, high) | An unpriced model would record zero and read as a free one (from docs/spec/session.md [R-SESSION-021], high) |
| Sum the costs of the priced calls | A partly priced session still shows a figure, which is a lower bound on the spend (reasoned from crates/meow-core/src/usage.rs:54-57, low) | The number silently omits part of the spend (from https://github.com/retran/meowg1k/pull/116, high) |
| An absent cost that makes the whole total absent (chosen) | A reported figure is always the whole spend (from crates/meow-core/src/usage.rs:41-44, high) | It won |

## What it costs

A session with one unpriced call among many priced ones shows no cost at all,
so the user loses the figure for the priced part (from
crates/meow-core/src/usage.rs:54-57, high). Until a price table exists, every
run pays this, because no provider fills a cost (from
crates/meow-llm/src/anthropic.rs:255, high). Cost is also an `Option` in the
store, the view and the renderers, so each reader handles the missing case
(from crates/meow-session/src/log.rs:157 and
crates/meow-core/src/view.rs:129-130, high).

## What would reverse it

- `R-SESSION-021` is amended to let an unpriced call count as zero, or a
  partial total becomes a requirement (reasoned from
  crates/meow-core/src/usage.rs:24-30, low).
- `Usage` gains a count of unpriced calls, so a partial sum can say what it
  leaves out and stops omitting it silently (reasoned from the reason PR 116
  gives, https://github.com/retran/meowg1k/pull/116, low).

## Consequences

- `Usage::add` treats cost and cached tokens differently: a missing cached
  count is skipped, and a missing cost poisons the sum (from
  crates/meow-core/src/usage.rs:47-57, high).
- The session log stores a null cost for an unpriced event and reads it back as
  `None` (from crates/meow-session/src/log.rs:157 and
  crates/meow-session/src/log.rs:336, high).
- Every surface that prints a cost needs an "unknown" wording (from
  crates/meow-ui/src/plain.rs:214-219, high).

## How I will know it was realised

1. `an_unpriced_call_records_no_cost_rather_than_a_free_one` in
   crates/meow-session/tests/spec.rs passes: a priced and an unpriced `Usage`
   event give a session total with `cost_micros` of `None` (from
   crates/meow-session/tests/spec.rs:301-335, high).
2. `a_parent_reports_what_its_children_spent` in the same file passes: priced
   children add to the parent's cost (from
   crates/meow-session/tests/spec.rs:337-365, high).

## What this does not settle

- Where prices come from: no price table exists, and no provider fills a cost
  (from crates/meow-llm/src/anthropic.rs:255, high).
- How the cost budget treats an unpriced call. `Ledger::charge` adds
  `cost_micros.unwrap_or(0)`, so an unpriced call counts as free against a cost
  budget, which is the reading `R-SESSION-021` forbids for the log (from
  crates/meow-agent/src/budget.rs:229-233, high).
