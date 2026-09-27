---
id: ADR-0118
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1601, REQ-1604]
supersedes: []
---

# 0118. Every OpenAI-shaped vendor is served by one provider implementation

## Decision

OpenAI, OpenRouter and llama are one implementation; they differ in the address
and in whether `response_format` is real, and both are declarations in place of
code (from docs/design/0.3.0-plan.md M11, high). Copilot is the same shape with
a different address and a credential the provider exchanges for itself (from
docs/design/0.3.0-plan.md M11, high). Gemini and Anthropic have their own,
because their request shapes differ (from docs/design/0.3.0-plan.md M11, high).
A declaration's `kind` decides the shape of the API and `base_url` decides the
address (from docs/design/0.3.0-starlark-api.md section 4, high). Whether
`response_format` is real is the provider's declared structured-output
capability, native or emulated (from crates/meow-llm/src/openai.rs:100-110,
high).

## Why

Five copies of a streaming parser is five places for a tool-call index to be
handled differently (from docs/design/0.3.0-plan.md M11, high). A streamed tool
call arrives by position, with only the first chunk carrying the identifier,
which is the kind of detail that gets copied wrong (from
https://github.com/meowshed/meowg1k/pull/133, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: one implementation per vendor, as v0.2.x had with `openai.go`, `openrouter.go`, `llama.go` and `copilot.go` | Each vendor's quirks stay in its own file, and a change for one can't break another (reasoned from v0.2.1:internal/adapters/gateway/, low) | Five copies of a streaming parser are five places to handle a tool-call index differently (from docs/design/0.3.0-plan.md M11, high); the four v0.2.1 files total 2,320 lines (from `git show v0.2.1:internal/adapters/gateway/<file>.go \| wc -l`, high) |
| One implementation for every vendor, Gemini and Anthropic included | One parser for the whole provider layer (reasoned from crates/meow-llm/src/openai.rs:13-14, low) | Gemini's shape differs: contents in place of messages, a system prompt in its own field and a tool result with no identifier; Anthropic's blocks differ too (from https://github.com/meowshed/meowg1k/pull/133 and crates/meow-llm/src/openai.rs:13-14, high) |

The pull request and the plan weigh no third split (from
https://github.com/meowshed/meowg1k/pull/133 and docs/design/0.3.0-plan.md M11,
high).

## What it costs

A vendor that copied OpenAI's shape and differs in one more place needs a
builder option on the shared type, as Copilot's editor headers did (from
crates/meow-llm/src/openai.rs:71-79 and crates/meow-llm/src/copilot.rs:25-40,
high). No live call is tested, so what is untested is that each real endpoint
accepts the bodies built here (from https://github.com/meowshed/meowg1k/pull/133,
high).

## What would reverse it

- An OpenAI-compatible vendor whose streaming or tool-call format diverges
  from OpenAI's, so that one parser can't read both (reasoned from
  crates/meow-llm/src/openai.rs:1-14, low).

## Consequences

Switching providers is a configuration change and never a code change, and a
local `llama.cpp` server is a first-class provider (from docs/philosophy.md
section 4, high). `copilot.rs` is about sixty lines: an address, four headers
and emulated schema (from https://github.com/meowshed/meowg1k/pull/156 and
crates/meow-llm/src/copilot.rs, high). The binary maps `openai`, `openrouter`,
`llama` and `copilot` onto `OpenAi` with different addresses and schema modes
(from crates/meow-cli/src/wire.rs:1331-1366, high).

## How I will know it was realised

1. `one_request_becomes_each_vendors_shape` in
   crates/meow-llm/tests/providers.rs passes: one request becomes the OpenAI body and the Gemini body (from
   crates/meow-llm/tests/providers.rs:61, high).
2. `a_native_schema_goes_in_the_request` in the same file passes: the OpenAI
   shape sends `response_format` of type `json_schema` (from
   crates/meow-llm/tests/providers.rs:293, high).
3. `an_expired_token_is_renewed` in crates/meow-llm/tests/bearer.rs passes, so
   the shared implementation serves Copilot's exchanged credential (from
   crates/meow-llm/tests/bearer.rs:118, high).

## What this does not settle

- The embedding model: `OpenAi::embed` hard-codes `text-embedding-3-small`,
  because `Provider::embed` has no model parameter (from
  https://github.com/meowshed/meowg1k/pull/133, high).
- Whether each vendor's real endpoint accepts the bodies built here, since the
  tests run against recordings (from https://github.com/meowshed/meowg1k/pull/133,
  high).
