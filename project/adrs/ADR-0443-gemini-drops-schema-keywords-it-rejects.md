---
id: ADR-0443
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2511]
supersedes: []
---

# 0443. Schema keywords Gemini rejects are dropped before sending

## Decision

`additionalProperties`, `default` and `x-meow` are dropped rather than sent to
Gemini (from https://github.com/meowshed/meowg1k/pull/133, high).

What works now: the Gemini provider removes `additionalProperties`, `$schema`,
`x-meow` and `default` at every depth of a schema, both from the response schema
it sends as `generationConfig.responseSchema` and from each tool's parameters,
so one `meow.arg` declaration reaches Gemini, OpenAI and Anthropic alike (from
crates/meow-llm/src/gemini.rs:122, crates/meow-llm/src/gemini.rs:140 and
crates/meow-llm/src/gemini.rs:249, high). The code drops `$schema` too, which
the pull request doesn't list (from crates/meow-llm/src/gemini.rs:250, high).
What still doesn't work: the filter matches a key at any level, so a property
whose own name is `default` or `additionalProperties` is removed from
`properties` as well (reasoned from crates/meow-llm/src/gemini.rs:256, medium).

## Why

Gemini rejects a whole request for a JSON Schema keyword it doesn't know, and
dropping them is what lets one `meow.arg` declaration serve every provider, per
`R-STAR-061` (from https://github.com/meowshed/meowg1k/pull/133, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: send the schema unchanged | The model sees every constraint, including `additionalProperties: false` and each default (reasoned from crates/meow-llm/tests/providers.rs `gemini_gets_a_schema_it_can_read`, low) | Gemini rejects the whole request (from https://github.com/meowshed/meowg1k/pull/133, high) |
| Strip the keywords in `meow-star` for every provider, as `schema::for_model` already strips `x-meow` | One schema for every provider, built in one place (from crates/meow-star/src/schema.rs:27, medium) | The source doesn't weigh it; the providers that accept `additionalProperties` and `default` would lose constraints only Gemini needs removed (reasoned from crates/meow-llm/src/openai.rs:153, low) |

A third option, translating the schema into the subset Gemini documents, isn't
named by the code, the history or the design documents, so the table stops at
two.

## What it costs

Gemini doesn't see `additionalProperties: false`, so it may return keys the
schema forbids, and the local validator doesn't check that keyword either, so
those keys reach the script (reasoned from crates/meow-llm/src/gemini.rs:326
and crates/meow-llm/src/schema.rs:14, low). Gemini also never sees a default,
so it can't fill one in (reasoned from crates/meow-llm/src/gemini.rs:250, low).

## What would reverse it

Gemini's API accepting `additionalProperties`, `default` and unknown extension
keys without rejecting the request, observable as a recorded exchange that
sends them and gets a 200 (reasoned from
https://github.com/meowshed/meowg1k/pull/133, low).

## Consequences

- The list of refused keywords lives in one constant, `REFUSED` in
  `gemini.rs`, and grows only by editing it (from
  crates/meow-llm/src/gemini.rs:250, high).
- The response is still parsed and checked against the original, unstripped
  schema, in one attempt (from crates/meow-llm/src/gemini.rs:323, high).

## How I will know it was realised

`crates/meow-llm/tests/providers.rs` `gemini_gets_a_schema_it_can_read`
passes: a schema carrying `additionalProperties`, `x-meow` and a nested
`default` reaches the recorded request without any of the three and keeps the
`type` it can read (from crates/meow-llm/tests/providers.rs:321, high).

## What this does not settle

- Whether a property named `default`, `additionalProperties`, `$schema` or
  `x-meow` should survive; today it is dropped from `properties` while
  `required` may still name it (reasoned from
  crates/meow-llm/src/gemini.rs:256, medium).
- No live call to Gemini is tested, so whether its real endpoint accepts the
  cleaned bodies is unverified (from https://github.com/meowshed/meowg1k/pull/133,
  "What is not covered", high).
