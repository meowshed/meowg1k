---
id: ADR-0601
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1078, REQ-2530, REQ-2262]
supersedes: []
---

# 0601. A model's price lives in its declaration

## Decision

`meow.model` takes an input and an output price per million tokens, a call
against a priced model records its cost from the usage the provider returns,
and a budget that caps cost for an agent whose model has no price fails when
the workspace loads (from https://github.com/meowshed/meowg1k/issues/168, high).
The owner chose the issue's first option on 2026-09-27.

Once this is accepted, none of it is built yet: `meow.model` takes seven named
arguments and no price, every provider sets `cost_micros: None`, and an agent's
`budget` accepts `cost_micros` for any model (from
crates/meow-star/src/declare.rs:183, crates/meow-llm/src/anthropic.rs:255,
crates/meow-llm/src/gemini.rs:281 and crates/meow-star/src/agent.rs:38, high).
The ledger already checks a cost cap and the session log already totals cost,
so a priced call is the missing source and not missing plumbing (from
https://github.com/meowshed/meowg1k/issues/168 and
crates/meow-agent/src/budget.rs:214, high).

## Why

A cap that silently never fires is worse than no cap, and the declaration is
the one place meowg1k can learn a price: the number belongs to a contract
meowg1k isn't party to, so the user states it and keeps it current (from
https://github.com/meowshed/meowg1k/issues/168, high). Principle 10, Predictable
Cost, asks for spend to be a parameter of the workflow and not a surprise
(from https://github.com/meowshed/meowg1k/issues/168, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep the axis with no price source | No change to the declaration or the budget (from https://github.com/meowshed/meowg1k/issues/168, high) | A workspace can declare `cost_micros` and it never stops a run, and `meow session show` reports an absent cost for every session (from https://github.com/meowshed/meowg1k/issues/168, high) |
| Remove the cost axis from `Budget`, keep cost in `Usage` for a provider that reports one, and say spend is bounded by tokens | Smaller, and removes a promise that isn't kept (from https://github.com/meowshed/meowg1k/issues/168, high) | Less useful: tokens bound spend only indirectly, and a user who wants to cap money gets nothing (from https://github.com/meowshed/meowg1k/issues/168, high) |
| Take the price from what the provider reports | The number would follow the vendor's price list with nothing for the user to keep current (reasoned from https://github.com/meowshed/meowg1k/issues/168, low) | No provider reports one today, so the axis would stay dead for every model (from crates/meow-llm/src/anthropic.rs:255 and crates/meow-llm/src/gemini.rs:281, high) |

## What it costs

The user keeps each price current by hand, and a stale price records a wrong
cost without any warning (from https://github.com/meowshed/meowg1k/issues/168,
high). `meow.model` grows from seven named arguments to nine, past the point
where clippy already needs an allow (from crates/meow-star/src/declare.rs:178,
high). A cap is checked before each call against what has been spent, so a run
can overshoot it by one call's cost; `Budget::default` leaves cost unbounded
for that reason (from crates/meow-agent/src/budget.rs:36 and :214, high).

## What would reverse it

- A provider that meowg1k ships starts reporting a per-call cost in its
  response, so the declared price and the reported one can disagree (reasoned
  from https://github.com/meowshed/meowg1k/issues/168, low).

## Consequences

- REQ-1078 makes REQ-1077 concrete: the price source is the model declaration
  and the refusal comes at load. REQ-1077 isn't contradicted, so it stays in
  force (from docs/requirements/REQ-1077-cost-cap-needs-a-price-source.md,
  high).
- REQ-1009's four axes and REQ-1019's unbounded default cost hold unchanged
  (from docs/requirements/REQ-1009-budget-bounds-four-axes.md and
  docs/requirements/REQ-1019-default-budget-values.md, high).
- REQ-2216's "price table in effect at that moment" is the declared prices at
  the time the `Usage` event is written, and REQ-2218 keeps an unpriced model's
  cost absent (reasoned from
  docs/requirements/REQ-2216-cost-computed-at-write-time.md and
  docs/requirements/REQ-2218-unpriced-model-records-absent-cost.md, low).

## How I will know it was realised

1. A test in crates/meow-star/tests/loading.rs loads a workspace whose agent
   caps `cost_micros` over an unpriced model and gets a load error.
2. A test drives one call against a model declared with `input_per_mtok` and
   `output_per_mtok` and finds a `Usage` event whose cost equals the prompt
   and completion tokens at those prices.

## What this does not settle

- How cached prompt tokens are priced: `Usage` carries them apart, and #168
  names only an input and an output price (from crates/meow-core/src/usage.rs
  and https://github.com/meowshed/meowg1k/issues/168, high).
- The currency, and whether `cost_micros` counts millionths of it (from
  crates/meow-star/src/agent.rs:37, high).
- Whether a cost cap over a priced agent is refused when a sub-agent or the
  compaction model it uses has no price (reasoned from REQ-1014, low).
