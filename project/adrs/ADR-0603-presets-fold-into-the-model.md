---
id: ADR-0603
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2533]
supersedes: []
---

# 0603. Presets fold into `meow.model`, which is the named configuration

## Decision

A workspace declares providers and models and no presets: a `meow.model` is the
named configuration, carrying its own temperature, and `"smart"` and `"fast"`
are models (from docs/design/0.3.0-starlark-api.md sections 4 and 11, high).

Once this is accepted, `meow.model` takes `temperature` and the `meow` global
has no preset declaration (from crates/meow-star/src/declare.rs:183 and
crates/meow-star/src/registry.rs:97, high). The design's "an agent may override
any sampling parameter inline" isn't built: an agent declaration has no
sampling field (from docs/design/0.3.0-starlark-api.md section 4 and
crates/meow-star/src/agent.rs:113-138, high).

## Why

v0.2.x had three layers, provider, model and preset, where the third existed
only to name a model-plus-temperature pair, which is what a named model already
is; removing it drops one indirection and loses nothing (from
docs/design/0.3.0-starlark-api.md section 4, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep `meow.preset(name, model, temperature)` over models, as v0.2.x did | Two sampling settings share one model declaration, so its context and output limits are written once (reasoned from docs/design/0.3.0-starlark-api.md section 11, low) | The preset only names a model and a temperature, which a named model already is (from docs/design/0.3.0-starlark-api.md section 4, high) |
| No named configuration: an agent names a provider's model id and sets its sampling inline | One declaration fewer per workspace (reasoned from docs/design/0.3.0-starlark-api.md section 4, low) | Every agent repeats the id, the context and the output limit, and an agent could no longer be pointed at `"smart"` (reasoned from .meow/meow.star, low) |

## What it costs

Two temperatures on one model mean two `meow.model` declarations that repeat
its id, context and output limit (reasoned from
crates/meow-star/src/declare.rs:183, low). A v0.2.x workspace's presets have to
be rewritten as models by hand (from docs/design/0.3.0-starlark-api.md section
11, high).

## What would reverse it

- A model gains enough sampling settings that workspaces declare the same model
  several times to vary them (reasoned from crates/meow-star/src/declare.rs:183,
  low).

## Consequences

- `.meow/meow.star` declares two models where v0.2.x had presets (from
  .meow/meow.star, high).
- `meow models` lists the named configurations, the only kind a workspace
  has (from crates/meow-cli/tests/surface.rs,
  `models_and_providers_list_what_was_declared`, medium).

## How I will know it was realised

1. `meow.preset(...)` in a workspace fails to load with an unknown-name error,
   and `grep -rn preset crates/meow-star/src` finds only the comment in
   registry.rs (from crates/meow-star/src/registry.rs:97, high).

## What this does not settle

- Whether an agent can override a sampling parameter inline, which the design
  promises and the agent declaration doesn't offer (from
  docs/design/0.3.0-starlark-api.md section 4 and crates/meow-star/src/agent.rs,
  high).
