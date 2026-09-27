---
id: SPC-1400
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-1400, REQ-1401, REQ-1402, REQ-1403, REQ-1404, REQ-1405, REQ-1406, REQ-1407, REQ-1408, REQ-1409, REQ-1410, REQ-1411, REQ-1412, REQ-1413, REQ-1414, REQ-1415, REQ-1416, REQ-1417, REQ-1418, REQ-1419, REQ-1420, REQ-1421, REQ-1422, REQ-1423, REQ-1424, REQ-1425, REQ-1426, REQ-1427, REQ-1428, REQ-1429, REQ-1430, REQ-1431, REQ-1432, REQ-1433, REQ-1434, REQ-1435, REQ-1436, REQ-1437, REQ-1438, REQ-1439]
---

# The index that makes a workspace searchable by meaning

## Scope

`meow-index` makes a workspace searchable by meaning. It walks files, splits
them into chunks, embeds the chunks, stores the vectors, and answers a query
with ranked chunks (from docs/spec/index.md, high).

It doesn't decide what to do with a result and doesn't call a generation model.
`search.code` is a thin tool over it (from docs/spec/index.md, high).

## Boundary

| Surface | What it is |
| --- | --- |
| The index API | build, update, query, stats and clear (from docs/spec/index.md, high) |
| Index rows | What the index writes through `meow-store` (from docs/spec/index.md, high) |
| Ranked results | What a tool returns to a model (from docs/spec/index.md, high) |

## Behaviour

### Walking

Indexing respects `.gitignore`, `.meowignore` and the workspace's own
`.meow/.data/` exclusion [REQ-1400]. `.meowignore` supports negation, so a path
`.gitignore` excludes can be indexed on purpose [REQ-1401], and `.meow/.data/`
can't be re-included [REQ-1402].

Indexing skips files it detects as binary [REQ-1403] and records how many it
skipped [REQ-1404]. It skips a file larger than a configured size [REQ-1405] and
reports each such skip [REQ-1406]. It doesn't follow a symlink that leaves the
workspace [REQ-1407]. Markdown and plain text are indexed on the same terms as
code [REQ-1408].

### Chunking

Chunking is deterministic [REQ-1409]: the same file content gives the same
chunks, with the same boundaries, on every run [REQ-1410]. A chunk carries its
source path, its byte range and its line range [REQ-1411]. Chunks overlap by a
configured number of lines [REQ-1412]. A chunk fits within the embedding model's
input limit [REQ-1413].

Chunk boundaries fall on line boundaries [REQ-1414], and a chunk never begins or
ends part-way through a line [REQ-1415], except that a single line longer than
the embedding model's input limit is split [REQ-1416] and the split is reported
[REQ-1417].

### Embedding

Chunks are embedded in batches [REQ-1418]. Embedding is resumable [REQ-1421]: a
build that was interrupted doesn't re-embed chunks whose vectors are already
stored [REQ-1422].

### Incremental update

Update re-embeds a file only when its content hash differs from the stored one
[REQ-1423], and the hash covers the chunking parameters as well as the content
[REQ-1424]. Update removes the chunks of a file that no longer exists or has
become excluded [REQ-1425], and reports how many files were added, changed,
removed and unchanged [REQ-1426].

### Query

A query returns results ranked by descending similarity, each carrying the path,
the line range, the chunk text and the score [REQ-1427]. It accepts a result
limit and a minimum score [REQ-1430] and applies both [REQ-1431]. It accepts a
path filter as globs [REQ-1433] and applies the filter before ranking
[REQ-1434]. When the embedding of the query itself is cached, a query needs no
network call [REQ-1432].

### Storage

Vectors are stored through `meow-store` in the same database as everything else
[REQ-1435]. The index records which embedding model produced it [REQ-1436].
`clear` removes every vector and chunk [REQ-1438] and leaves sessions and the
key-value store untouched [REQ-1439].

## Failure paths

A batch that a provider rejects for exceeding a limit is split and retried
[REQ-1419]. A single chunk that a provider rejects as too large fails with an
error naming the file and the chunk's line range [REQ-1420].

A query against an empty or absent index returns no results and says the index
is empty [REQ-1428], and doesn't build an index [REQ-1429].

A query against an index built by a different embedding model fails with both
model names [REQ-1437].
