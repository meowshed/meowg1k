---
id: BUG-0202
artifact: bug
status: draft
severity: major
violates: REQ-2530
found: 2026-09-27
revised: 2026-09-27
issue: 168
---

<!-- Written to the writing standard meow-prose ships: lead with the answer,
give each rule its reason in the same sentence, and show the failing case. -->

# A declared cost cap never stops a run, because no model carries a price

## Reproduction

At `d67ae50`:

1. Declare an agent with `budget = {cost_micros: 1}`.
2. Run it for several steps.

The run is never stopped by cost, and `meow session show` reports no cost (from
https://github.com/retran/meowg1k/issues/168, high).

## What the system does

`Budget` has a `cost_micros` axis and the ledger checks it, but every provider
sets `cost_micros: None`, so the axis never fires. `meow.model` takes no price,
so nothing can supply one (from https://github.com/retran/meowg1k/issues/168 and
crates/meow-llm/src, high).

## What it should do, and why

REQ-2530: `meow.model` MUST accept an input and an output price per million
tokens, with REQ-2262 recording a priced call's cost and REQ-1078 refusing at
load a cost cap over an unpriced model. ADR-0601 chose a price in the
declaration over removing the axis.

## Triage

It enters where the budget gained a cost axis with no source for it. Major: a
declared cap silently does nothing, which the issue calls worse than not
offering it.

## Closed by

A test that declares a priced model and a cost cap and shows the run stopped by
cost, and one that declares a cap over an unpriced model and shows the load
refused. Issue #168 closes with the fix.
