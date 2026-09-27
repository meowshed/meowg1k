---
id: BUG-0028
artifact: bug
status: approved
severity: major
violates: REQ-2437
found: 2026-09-27
revised: 2026-09-27
issue: 201
---

# `xml.parse` drops entity references and CDATA from an element's text

`xml.parse('<a>x &amp; y<![CDATA[z]]></a>')` returns an element whose `text` is
`"xy"`. The `&amp;` and the CDATA section are both lost, along with the spaces
around the entity.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Create a workspace whose `.meow/meow.star` loads `parse` from `@std//xml` and
   declares a command whose handler writes
   `repr(parse("<a>x &amp; y<![CDATA[z]]></a>").text)`.
2. Run `meow trust`, then the command, with `HOME` pointed at a scratch
   directory.

It prints `xml="xy"`. It should print `"x & yz"`. This was run against
`target/debug/meow`.

## What the system does

`read_xml` in `crates/meow-star/src/modules.rs:499-547` handles `Start`,
`Empty`, `End` and `Text`, and drops every other event at line 539. quick-xml
0.41 reports `&amp;` as `Event::GeneralRef` and a CDATA section as
`Event::CData`, so both fall into `_ => {}`. `trim_text(true)` at line 503 then
trims the text pieces on each side of the entity, which removes the spaces.

## What it should do, and why

REQ-2437: "`xml.parse` MUST return a tree in which every element carries its
`tag`, its `attrs`, its `children`, and its `text`." Text with an entity in it
is the element's text, and a feed or a build report with `&amp;` or `&lt;` in it
is ordinary input.

## Triage

The defect enters in `meow-star`'s `read_xml`. It is major because the text of
any element with an escaped character comes back wrong, silently. The fix is to
resolve `GeneralRef` for the five predefined entities and character references,
append `CData` as it is, and trim the element's whole text once instead of each
piece.

## Closed by

A test in `crates/meow-star/tests/running.rs`, named for example
`xml_parse_keeps_entities_and_cdata_in_text`, that expects `"x & yz"` for the
input above.
