---
id: ADR-0105
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2460, REQ-2484, REQ-2487]
supersedes: []
---

# 0105. Nothing chooses an embedding model for the index on the workspace's behalf

## Decision

`meow.index` names the model that embeds the workspace, and `meow index build`
in a workspace that declares no index says to declare one (from
docs/design/0.3.0-starlark-api.md section 4, high). The binary answers such a
command with "this workspace declares no index; add `meow.index(model = ...)`
to .meow/meow.star" (from crates/meow-cli/src/index.rs:66-78, high).

## Why

Choosing a model is how an index gets built by one model and queried by another
(from docs/design/0.3.0-starlark-api.md section 4, high). The index needed an
embedding model and nothing said how a workspace names one, so [R-INDEX-051]
could record which model built an index that no declaration could choose (from
docs/design/0.3.0-plan.md M10, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: each call names an embedding preset, as v0.2.1's `search.star` did with `preset="embeddings"` | No index declaration to write; a script picks a model where it embeds (from `git show v0.2.1:.meowg1k/commands/search.star`, lines 358-360, high) | Building and querying name the model separately, so they can disagree (reasoned from `git show v0.2.1:.meowg1k/commands/search.star`, lines 358-360, low); choosing is how an index gets built by one model and queried by another (from docs/design/0.3.0-starlark-api.md section 4, high) |
| The runtime picks an embedding model when the workspace names none | `meow index build` works in a fresh workspace with no extra line (reasoned from crates/meow-cli/src/index.rs:66-78, low) | It lets an index be built by one model and queried by another (from docs/design/0.3.0-starlark-api.md section 4, high) |
| Catch a mismatch only at query time, with [R-INDEX-051] alone | Nothing to declare up front, and the mismatch is still reported (from docs/spec/starlark.md [R-STAR-035], high) | [R-INDEX-051] catches the mismatch after the fact, and the declaration prevents it (from docs/spec/starlark.md [R-STAR-035], high) |

## What it costs

A workspace that wants semantic search writes one more declaration, and needs a
model of `kind = "embedding"` from a provider that offers one (from
docs/design/0.3.0-starlark-api.md section 4, high). Until it does, `meow index`
commands fail and `search.code` says there is no index (from
crates/meow-cli/src/index.rs:384 and crates/meow-cli/tests/surface.rs:811-813,
high).

## What would reverse it

- Embedding vectors become comparable across models, so an index built by one
  model gives meaningful scores to a query embedded by another; the whole
  reason is that they don't (reasoned from docs/spec/index.md [R-INDEX-051],
  low).

## Consequences

`meow.model` takes a `kind` of `chat` or `embedding`, and an agent given an
embedding model, or an index given a chat model, fails at load time naming both
(from docs/design/0.3.0-starlark-api.md section 4, high). [R-STAR-034] and
[R-STAR-035] were added on 2026-09-20 for this (from docs/design/0.3.0-plan.md
M10, high). A workspace with no index still loads, and only an index command
fails (from crates/meow-star/tests/loading.rs:514, high).

## How I will know it was realised

1. `an_index_command_without_a_declaration_says_so` in
   crates/meow-cli/tests/surface.rs passes (from
   crates/meow-cli/tests/surface.rs:534-536, high).
2. `an_index_declares_its_model_and_parameters`,
   `an_index_may_not_use_a_chat_model` and `an_index_may_be_declared_only_once`
   in crates/meow-star/tests/loading.rs pass (from
   crates/meow-star/tests/loading.rs:462-496, high).
3. `a_query_by_the_wrong_model_names_both` in crates/meow-index/tests/index.rs
   passes, as the backstop for an index built before a model change (from
   crates/meow-index/tests/index.rs:425, high).

## What this does not settle

- What happens to an existing index when the declared model changes: the
  query fails naming both models (from docs/spec/index.md [R-INDEX-051], high),
  and this decision doesn't say who rebuilds it.
- The chunk size, overlap and file-size limit, which `meow.index` may set but
  doesn't have to (from docs/spec/starlark.md [R-STAR-035], high).
