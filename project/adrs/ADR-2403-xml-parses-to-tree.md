---
id: ADR-2403
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2437, REQ-2438, REQ-2439]
supersedes: []
---

# 2403. XML is a tree and not a dictionary

## Decision

`xml.parse` returns what XML is - a tag, attributes, ordered children and text -
and never flattens an element into a dictionary (from docs/spec/starlark.md
[R-STAR-018], high).

## Why

Every library that maps XML onto the shape JSON has must decide what to do when
an element has both attributes and children, or two children with one tag, and
every such decision is wrong for some document (from docs/spec/starlark.md,
Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Map XML onto the shape JSON has | A handler reads a value with `doc["feed"]["item"]` and `yaml`, `toml` and `json` all return that shape (reasoned from docs/spec/starlark.md [R-STAR-016], low). | It has to decide how to map an element with both attributes and children, or two children with one tag, and every such decision is wrong for some document (from docs/spec/starlark.md, Decisions, high). |
| Return a tree of dictionaries from `parse` | A handler builds and reads one shape, the one `encode` takes (reasoned from crates/meow-star/src/modules.rs `xml_module`, low). | `parse` returns structs because `root.tag` reads better than `root["tag"]`; `encode` takes both, because a handler building a document from nothing has only dicts (from crates/meow-star/src/modules.rs `xml.encode`, high). |
| Do nothing: no `xml` module, as before PR #141 | No XML parser to keep safe against untrusted input (reasoned from https://github.com/meowshed/meowg1k/pull/141, low). | The design table promised `xml`, and [R-STAR-015] requires `yaml`, `toml`, `csv` and `xml` in the table (from https://github.com/meowshed/meowg1k/pull/141 and docs/spec/starlark.md [R-STAR-015], high). |

## What it costs

A handler that wants a dictionary writes the three lines that build one, knowing
its own document (from docs/spec/starlark.md, Decisions, high). Mixed content
loses its order: an element's text pieces are joined into one `text`, so text
between two children can't be placed (from crates/meow-star/src/modules.rs
`read_xml`, high).

## What would reverse it

A document shape where the tree can't carry what the handler needs, for example
namespace resolution, which [R-STAR-018] doesn't promise (from
https://github.com/meowshed/meowg1k/pull/141, medium).

## Consequences

Two children sharing a tag both survive, in order (from
crates/meow-star/tests/running.rs
`xml_parse_keeps_what_a_dictionary_would_lose`, high). Namespaces are read as
ordinary attributes: `xmlns:x` is a key in `attrs`, and a tag keeps its prefix
(from https://github.com/meowshed/meowg1k/pull/141, high). `quick-xml` is pinned
at 0.41, because older releases take quadratic time on duplicate attribute
names and a handler parsing model output parses untrusted XML (from
https://github.com/meowshed/meowg1k/pull/141, high).

## How I will know it was realised

`xml_parse_keeps_what_a_dictionary_would_lose`,
`xml_encode_escapes_text_and_attributes` and `xml_round_trips_a_tree` in
crates/meow-star/tests/running.rs pass (from those tests, high). No test parses
an entity reference or a CDATA section, and a probe shows both are lost:
`xml.parse('<a>x &amp; y<![CDATA[z]]></a>').text` is `xy`, where the document
says `x & yz` (from `target/debug/meow` built on 2026-09-27, and
crates/meow-star/src/modules.rs `read_xml`, which ignores every `quick-xml`
event except `Start`, `Empty`, `End` and `Text`, high).

## What this does not settle

Namespace resolution, which [R-STAR-018] doesn't promise; a handler that needs
it resolves it from what the tree carries (from
https://github.com/meowshed/meowg1k/pull/141, high). How mixed content and
whitespace are kept: `read_xml` trims text and joins an element's pieces (from
crates/meow-star/src/modules.rs `read_xml`, high).
