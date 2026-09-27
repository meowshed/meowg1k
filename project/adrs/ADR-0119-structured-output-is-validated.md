---
id: ADR-0119
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1070, REQ-1072, REQ-1642, REQ-1643, REQ-1644, REQ-1646, REQ-2515, REQ-2516]
supersedes: []
---

# 0119. Structured output reaches the script parsed and validated

## Decision

An agent declares its output with `meow.schema`, and the schema builder emits
provider-native structured output where available and falls back to a validated
retry loop where not; either way the script sees a parsed value in `.value`
(from docs/design/0.3.0-starlark-api.md section 7, high). The provider parses
and validates the answer against the schema in both modes; an emulated schema
gets up to `SCHEMA_ATTEMPTS`, three, with the validation error in each
follow-up, and a native one gets one attempt (from
crates/meow-llm/src/openai.rs:322-372 and crates/meow-llm/src/schema.rs:9,
high).

## Why

v0.2.x took a raw JSON Schema dictionary and returned whatever the model
produced, leaving validation to the caller (from
docs/design/0.3.0-starlark-api.md section 7, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: take a raw JSON Schema dictionary and return the model's text, as in v0.2.x | Any JSON Schema a caller writes goes to the provider, and the runtime has no validator to maintain (reasoned from docs/design/0.3.0-starlark-api.md section 7, low) | Every caller has to validate what the model produced (from docs/design/0.3.0-starlark-api.md section 7, high) |
| Native structured output only, refusing a schema on a provider that lacks it | No retry loop, and every accepted schema is enforced by the vendor's API (reasoned from crates/meow-llm/src/openai.rs:330-333, low) | Schema support is never refused under [R-LLM-002], only satisfied two ways, and the draft that both refused and fell back was one of the contradictions the spec reconciled (from docs/requirements/REQ-1643-emulated-structured-output-validates-json.md and https://github.com/meowshed/meowg1k/pull/111, high) |
| A full JSON Schema validator as a dependency | Every JSON Schema keyword is checked, not only those `meow.schema` emits (reasoned from crates/meow-llm/src/schema.rs:13-16, low) | A dependency this doesn't need yet, since the schemas it checks are the ones this workspace emits (from crates/meow-llm/src/schema.rs:13-16, high) |

## What it costs

An emulated schema can cost up to three model calls for one answer (from
crates/meow-llm/src/schema.rs:9, high). The validator covers types, required
fields, enumerations and the numeric and string bounds only, so a schema
written outside `meow.schema` can pass keywords it doesn't check (from
crates/meow-llm/src/schema.rs:13-16, high). Gemini rejects schema keywords it
doesn't know, so `additionalProperties`, `default` and `x-meow` are dropped
before it is sent (from https://github.com/meowshed/meowg1k/pull/133, high).

## What would reverse it

- A provider whose native mode can't carry the schemas `meow.schema` emits and
  whose answers the validator can't check, so neither route yields a parsed
  value (reasoned from crates/meow-llm/src/provider.rs:121-126, low).

## Consequences

A run that finishes carries the parsed value, and a run that stops early carries
none, because a partial answer would wear the shape of a complete one (from
crates/meow-agent/src/engine.rs:420-430, high). A final answer the provider
couldn't make fit ends the run with `failed` (from
crates/meow-agent/tests/spec.rs:730, high). A provider that produces no text,
Voyage, declares `Structured::None`, so a schema sent to it fails before sending
(from crates/meow-llm/src/voyage.rs:76 and crates/meow-llm/src/provider.rs:121,
high).

## How I will know it was realised

1. `a_schema_is_satisfied_by_validation_and_returned_parsed` and
   `validation_covers_the_constraints_the_schema_builder_can_express` in
   crates/meow-llm/tests/spec.rs pass (from crates/meow-llm/tests/spec.rs:594
   and 652, high).
2. `a_native_schema_goes_in_the_request` in crates/meow-llm/tests/providers.rs
   passes (from crates/meow-llm/tests/providers.rs:293, high).
3. `a_parsed_value_arrives_only_with_a_finished_run` and
   `a_schema_the_provider_gave_up_on_fails_the_run` in
   crates/meow-agent/tests/spec.rs pass (from
   crates/meow-agent/tests/spec.rs:699 and 730, high).
4. `a_schema_emits_json_schema` in crates/meow-star/tests/arguments.rs passes
   (from crates/meow-star/tests/arguments.rs:264, high).

## What this does not settle

- Whether the attempt count is configurable: REQ-1644 says "a configured
  attempt count", and the code fixes it at the constant three (from
  docs/requirements/REQ-1644-schema-failure-retried-with-error.md and
  crates/meow-llm/src/schema.rs:9, high).
- A native answer that fails validation gets no retry, because the API is
  trusted to have enforced the schema (from
  crates/meow-llm/src/openai.rs:330-333, high).
