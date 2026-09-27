---
id: BUG-0120
artifact: bug
status: approved
severity: minor
violates: REQ-2511
found: 2026-09-27
revised: 2026-09-27
issue: 243
---

# Gemini's schema filter strips a property named `default`

The filter that removes keywords Gemini refuses matches keys at every depth,
including property names inside `properties`, so an argument named `default`
is removed from the schema while `required` still lists it.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare a tool with
   `args = {"default": meow.arg.string(about = "the fallback")}`.
2. Offer it to an agent whose model is served by a `gemini` provider, and
   capture the request body.
3. The tool's `parameters.properties` has no `default` key, and
   `parameters.required` holds `"default"`.

This follows from the code below.

## What the system does

`clean_schema` at `crates/meow-llm/src/gemini.rs:248-262` drops every object key
in `["additionalProperties", "$schema", "x-meow", "default"]` and recurses into
every value. A `properties` object is an object like any other, so its
`default` entry goes. The same happens to a property named `x-meow`,
`$schema` or `additionalProperties`.

## What it should do, and why

REQ-2511: "One argument declaration MUST produce the command-line flag, the help
text, and the JSON Schema sent to the model, so the three cannot drift." The
filter should drop keywords only where they're keywords, and keep the keys of a
`properties` object.

## Triage

The defect enters in the Gemini provider. It's minor: it needs an argument with
one of four names, and only Gemini is affected. When it happens, Gemini either
refuses the request or the model can't supply the argument.

## Closed by

A test in `crates/meow-llm/tests/`, named for example
`gemini_keeps_a_property_named_default`, that sends the schema above through a
mock transport and expects `properties.default` in the body.
