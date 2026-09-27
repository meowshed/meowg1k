---
id: SPC-2400
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-2400, REQ-2401, REQ-2402, REQ-2403, REQ-2404, REQ-2405, REQ-2406, REQ-2407, REQ-2408, REQ-2409, REQ-2410, REQ-2411, REQ-2412, REQ-2413, REQ-2414, REQ-2415, REQ-2416, REQ-2417, REQ-2418, REQ-2419, REQ-2420, REQ-2421, REQ-2422, REQ-2423, REQ-2424, REQ-2425, REQ-2426, REQ-2427, REQ-2428, REQ-2429, REQ-2430, REQ-2431, REQ-2432, REQ-2433, REQ-2434, REQ-2435, REQ-2436, REQ-2437, REQ-2438, REQ-2439, REQ-2440, REQ-2441, REQ-2442, REQ-2443, REQ-2444, REQ-2445, REQ-2446, REQ-2447, REQ-2448, REQ-2449, REQ-2450, REQ-2451, REQ-2452, REQ-2453, REQ-2454, REQ-2455, REQ-2456, REQ-2457, REQ-2458, REQ-2459, REQ-2460, REQ-2461, REQ-2462, REQ-2463, REQ-2464, REQ-2465, REQ-2466, REQ-2467, REQ-2468, REQ-2469, REQ-2470, REQ-2471, REQ-2472, REQ-2473, REQ-2474, REQ-2475, REQ-2476, REQ-2477, REQ-2478, REQ-2479, REQ-2480, REQ-2481, REQ-2482, REQ-2483, REQ-2484, REQ-2485, REQ-2486, REQ-2487, REQ-2488, REQ-2489, REQ-2490, REQ-2491, REQ-2492, REQ-2493, REQ-2494, REQ-2495, REQ-2496, REQ-2497, REQ-2498, REQ-2499, REQ-2500, REQ-2501, REQ-2502, REQ-2503, REQ-2504, REQ-2505, REQ-2506, REQ-2507, REQ-2508, REQ-2509, REQ-2510, REQ-2511, REQ-2512, REQ-2513, REQ-2514, REQ-2515, REQ-2516, REQ-2517, REQ-2518, REQ-2519, REQ-2520, REQ-2521, REQ-2522, REQ-2523, REQ-2524, REQ-2525, REQ-2526, REQ-2527, REQ-2528, REQ-2529, REQ-2530, REQ-2531, REQ-2532, REQ-2533, REQ-2534, REQ-2535, REQ-2536, REQ-2537, REQ-2538, REQ-2539, REQ-2540, REQ-2541, REQ-2542, REQ-2543, REQ-2544, REQ-2545, REQ-2546, REQ-2547, REQ-2548, REQ-2549, REQ-2550, REQ-2551, REQ-2552, REQ-2553]
---

# The Starlark surface users write against

## Scope

`meow-star` is the surface users write against. It loads `.meow/`, evaluates the
declarations it finds, turns them into the specs and tools the engine runs, and
implements the standard library modules a handler imports (from
docs/spec/starlark.md, high).

`meow-star` depends on `meow-agent` and never the other way round (from
docs/spec/starlark.md, high). Packages loaded by `@<pkg>//` have a
specification of their own, and this document covers only how the scheme
resolves.

## Boundary

The boundary is the `meow` global, the `@std//` modules, the handler context,
the loading scheme, and the diagnostics a user sees when any of them is wrong
(from docs/spec/starlark.md, high).

## Behaviour

### Discovery and loading

- The workspace root is the nearest ancestor of the working directory that
  contains `.meow/meow.star`. Discovery stops at the first match and never
  merges with a global configuration [REQ-2400] [REQ-2401] [REQ-2402] (from
  docs/spec/starlark.md [R-STAR-001], high).
- `load("@std//<name>", ...)` resolves to a runtime module [REQ-2405] (from
  docs/spec/starlark.md [R-STAR-003], high).
- `load("//<path>", ...)` resolves relative to `.meow/` [REQ-2407] (from
  docs/spec/starlark.md [R-STAR-004], high).
- `load("@<pkg>//<path>", ...)` resolves to a declared package and is never read
  as a local path [REQ-2409] [REQ-2410] (from docs/spec/starlark.md
  [R-STAR-005], high).
- A file is evaluated at most once per invocation, however many `load`
  statements name it [REQ-2413] (from docs/spec/starlark.md [R-STAR-007], high).

### The module table

- Runtime modules are registered in one table, and every consumer takes its
  modules from that table [REQ-2414] [REQ-2415] (from docs/spec/starlark.md
  [R-STAR-010], high).
- A tool handler inside an agent loop sees the same modules, with the same
  behaviour, as a handler run from the command line [REQ-2416] (from
  docs/spec/starlark.md [R-STAR-011], high).
- The table holds `re` and `time`, and both behave the same from the command
  line and inside an agent loop [REQ-2417] [REQ-2418] (from
  docs/spec/starlark.md [R-STAR-012], high).
- `re.match`, `re.find_all`, `re.replace` and `re.split` take a pattern and a
  subject. A pattern that compiles and matches nothing isn't an error: `match`
  returns `None`, and `find_all` and `split` return a list [REQ-2419] [REQ-2421]
  [REQ-2422] [REQ-2423] (from docs/spec/starlark.md [R-STAR-013], high).
- `time.now`, `time.parse`, `time.format` and `time.since` work in UTC seconds
  and never take or return a local time, so a duration is the same number in
  every zone [REQ-2424] [REQ-2425] [REQ-2426] (from docs/spec/starlark.md
  [R-STAR-014], high).
- The table holds `yaml`, `toml`, `csv` and `xml`, each exposing `parse` and
  `encode` [REQ-2427] [REQ-2428] (from docs/spec/starlark.md [R-STAR-015],
  high).
- `yaml.parse`, `toml.parse` and `json.parse` produce the same value for the
  same data, and each `encode` takes what the other two produce, so a handler
  converts formats without knowing which it read [REQ-2431] [REQ-2432]
  [REQ-2433] (from docs/spec/starlark.md [R-STAR-016], high).
- `csv.parse` returns a list of dictionaries when the first record names the
  columns and a list of lists when it doesn't, and `csv.encode` takes either
  shape [REQ-2434] [REQ-2436] (from docs/spec/starlark.md [R-STAR-017], high).
- `csv.parse` takes `header`, `True` by default, saying whether the first record
  names the columns [REQ-2553] (from https://github.com/retran/meowg1k/pull/141,
  high).
- `xml.parse` returns a tree whose elements carry `tag`, `attrs`, `children` and
  `text`, never a flattened dictionary. `xml.encode` takes that tree or a tree
  of dictionaries with the same four keys, and escapes text and attribute values
  [REQ-2437] [REQ-2438] [REQ-2439] [REQ-2440] (from docs/spec/starlark.md
  [R-STAR-018], high).
- `search.text` and `search.files` search the workspace with no index and obey
  the walk the index obeys. `search.text` takes a literal by default and a
  regular expression when asked, and reports the path and line number of every
  hit [REQ-2441] [REQ-2442] [REQ-2443] [REQ-2444] (from docs/spec/starlark.md
  [R-STAR-019], high).
- `search.code` ranks the workspace against a question by meaning and returns at
  most `limit` hits, 10 by default, whatever their score [REQ-2545] [REQ-2546].
  Each hit carries `path`, `first_line`, `last_line`, `text` and `score`
  [REQ-2547], and `paths`, a list of globs, narrows the search to the files they
  match [REQ-2548] (from crates/meow-star/src/modules.rs:781-829, high).
- The table holds no `crypto`, `template` or `math` module [REQ-2535] (from
  docs/design/0.3.0-starlark-api.md section 10, high).

### Files, commands and the repository

- `fs.glob` returns, sorted, every file in the workspace whose path relative to
  the workspace root matches the pattern, and never walks into a directory named
  `.data` [REQ-2550] [REQ-2551] (from
  https://github.com/retran/meowg1k/pull/135, high).
- `fs` checks a relative path lexically, so it can follow a symbolic link inside
  the workspace that points outside it [REQ-2539] (from
  https://github.com/retran/meowg1k/pull/135, high).
- `@std//git` runs the user's own `git` program as a subprocess [REQ-2540].
  `git.commit` commits what is staged and nothing else [REQ-2542], and
  `git.status` returns `git`'s porcelain v1 output as text [REQ-2543] (from
  https://github.com/retran/meowg1k/pull/136, high).

### The handler context

- The handler context exposes `args`, `session`, `out`, `ask`, `stdin` and
  `workspace`, plus `cancelled()`, and nothing else [REQ-2445] (from
  docs/spec/starlark.md [R-STAR-020], high).
- A handler reaches a runtime capability by `load`, never as a member of the
  context [REQ-2446] (from docs/spec/starlark.md [R-STAR-021], high).

### The network

- The table holds `http`, exposing `get`, `post`, `put` and `delete` [REQ-2447]
  (from docs/spec/starlark.md [R-STAR-022], high).
- Every `http` call carries a deadline and caps how much of a response it reads
  [REQ-2448] [REQ-2450] (from docs/spec/starlark.md [R-STAR-022], high).
- A response the server sent is returned whatever its status, with the status,
  the headers and the body, so a 404 isn't a failure [REQ-2452] [REQ-2453]
  [REQ-2454] (from docs/spec/starlark.md [R-STAR-023], high).

### The index

- The table holds `index`, exposing `build`, `update`, `stats` and `query`.
  `build` and `update` report what changed as counts, and `query` takes the
  score floor below which a hit isn't returned [REQ-2456] [REQ-2457] [REQ-2458]
  (from docs/spec/starlark.md [R-STAR-024], high).

### The store

- The table holds `store`, exposing `get`, `put`, `delete` and `keys`. A value
  survives the run that wrote it and survives collecting every session in the
  workspace [REQ-2463] [REQ-2464] [REQ-2465] (from docs/spec/starlark.md
  [R-STAR-026], high).
- `store.get` returns what `store.put` was given, of the same type. A key never
  written gives the caller's default, or `None` when the caller named none
  [REQ-2466] [REQ-2467] (from docs/spec/starlark.md [R-STAR-027], high).
- `store.delete` says whether the key was there and succeeds when it wasn't.
  `store.keys` returns the keys sorted and takes a prefix [REQ-2468] [REQ-2469]
  [REQ-2470] [REQ-2471] (from docs/spec/starlark.md [R-STAR-028], high).

### Paths

- `path` uses `/` on every platform: it accepts `\` in its input and never
  returns it [REQ-2472] [REQ-2473] [REQ-2474] (from docs/spec/starlark.md
  [R-STAR-029], high).

### Declarations

- `meow.provider`, `meow.model`, `meow.agent`, `meow.tool`, `meow.command` and
  `meow.policy` are callable only while `.meow/` is being evaluated [REQ-2475]
  (from docs/spec/starlark.md [R-STAR-030], high).
- References are resolved after every declaration file has been evaluated, so
  declaration order inside and between files doesn't matter [REQ-2479] (from
  docs/spec/starlark.md [R-STAR-032], high).
- `meow.model` takes a `kind` of `chat` or `embedding`, defaulting to `chat`
  [REQ-2482] (from docs/spec/starlark.md [R-STAR-034], high).
- `meow.model` takes an input and an output price per million tokens, as
  `input_per_mtok` and `output_per_mtok` [REQ-2530] (from
  https://github.com/retran/meowg1k/issues/168, high).
- The `meow` global offers no preset declaration, because a named `meow.model`
  carries its own temperature [REQ-2533] (from docs/design/0.3.0-starlark-api.md
  section 4, high).
- A declaration builtin treats an explicit `None`, given for an optional keyword
  argument whose default is absent, as if the argument were left out [REQ-2536]
  (from https://github.com/retran/meowg1k/pull/126, high).
- `meow.index` names the model that embeds the workspace, can set the chunk
  size, the overlap and the file-size limit, and is declared at most once
  [REQ-2484] [REQ-2485] [REQ-2486] (from docs/spec/starlark.md [R-STAR-035],
  high).

### Agents

- `meow.agent` requires `name`, `model` and `system`, and takes `about`,
  `tools`, `budget`, `compaction`, `output`, `on_tool_error` and `policy`
  [REQ-2488] [REQ-2489] (from docs/spec/starlark.md [R-STAR-040], high).
- A `budget` that omits an axis takes that axis from the default budget, in
  Starlark and in frontmatter alike [REQ-2528] (from
  https://github.com/retran/meowg1k/pull/122, high).
- An agent value goes in another agent's `tools` list and presents the same
  schema there as a tool declared with `meow.tool` [REQ-2490] [REQ-2491] (from
  docs/spec/starlark.md [R-STAR-041], high).
- `agent.run(task, ...)` returns `text`, `value`, `stop`, `detail`, `ok`,
  `usage`, `steps` and `session`, with `stop` one of the six values in
  [R-AGENT-002] [REQ-2492] (from docs/spec/starlark.md [R-STAR-042], high).
- `agent.run` takes `on = meow.on(text = ..., tool = ..., step = ...)`, each
  callback optional [REQ-2531], and draws the run with the built-in renderer
  when given no `on` [REQ-2532] (from docs/design/0.3.0-starlark-api.md section
  5.5, high).
- `agent.call(...)` builds an invocation without running it, and
  `meow.parallel([...])` takes a list of them [REQ-2493] [REQ-2494] (from
  docs/spec/starlark.md [R-STAR-043], high).

### Markdown agents

- A `.md` file under `.meow/agents/` declares an agent. Its frontmatter takes
  every keyword argument of `meow.agent` except `system`, plus `include`, and
  its body is the system prompt [REQ-2496] (from docs/spec/starlark.md
  [R-STAR-050], high).
- A markdown agent and a Starlark agent produce the same value and can't be told
  apart by a caller [REQ-2498] [REQ-2499] (from docs/spec/starlark.md
  [R-STAR-051], high).
- A `.md` file under `.meow/lib/` loads as a string [REQ-2501] (from
  docs/spec/starlark.md [R-STAR-053], high).
- Frontmatter can carry `include`, naming `.meow/lib/*.md` files whose contents
  are prepended to the system prompt in the order given, separated by a blank
  line. `include` is the only composition a markdown agent has: there is no
  substitution, conditional or loop [REQ-2502] [REQ-2503] [REQ-2504] [REQ-2505]
  (from docs/spec/starlark.md [R-STAR-054], high).

### Tools and arguments

- `meow.arg` supports `string`, `int`, `float`, `bool`, `enum` and `list`, each
  with an optional default and `about`. `string` takes `max_len` and `pattern`,
  `int` and `float` take `min` and `max`, `enum` requires its values and `list`
  requires an element type [REQ-2506] [REQ-2507] [REQ-2508] [REQ-2509]
  [REQ-2510] (from docs/spec/starlark.md [R-STAR-060], high).
- One argument declaration produces the command-line flag, the help text and the
  JSON Schema sent to the model [REQ-2511] (from docs/spec/starlark.md
  [R-STAR-061], high).
- A `positional` argument takes an index, and the indices within one tool are
  unique and contiguous from zero [REQ-2512] [REQ-2513] (from
  docs/spec/starlark.md [R-STAR-062], high).
- A declared constraint is enforced on the command line and on a model-supplied
  argument alike [REQ-2514] (from docs/spec/starlark.md [R-STAR-063], high).

### Schemas

- `meow.schema` builds object, list, string, int, float, bool and enum schemas
  and emits valid JSON Schema [REQ-2515] [REQ-2516] (from docs/spec/starlark.md
  [R-STAR-070], high).

### Evaluation model

- Each invocation owns one evaluator, and the runtime never moves an evaluator
  or a Starlark value between threads [REQ-2518] [REQ-2519] (from
  docs/spec/starlark.md [R-STAR-080], high).
- A builtin that performs input or output blocks the calling script thread and
  exposes no future or callback to Starlark [REQ-2520] [REQ-2521] (from
  docs/spec/starlark.md [R-STAR-081], high).
- A declaration file writes no files, runs no commands and makes no network
  requests [REQ-2523] (from docs/spec/starlark.md [R-STAR-083], high).
- While `.meow/` is being evaluated, only `load` and `@std//env` are callable
  [REQ-2524] (from docs/spec/starlark.md [R-STAR-084], high).

## Failure paths

### Discovery and loading

- Outside any workspace, the run fails naming the directories searched and
  suggests `meow init` [REQ-2403] [REQ-2404] (from docs/spec/starlark.md
  [R-STAR-002], high).
- `load("@std//<name>", ...)` with no such module fails listing the available
  modules [REQ-2406] (from docs/spec/starlark.md [R-STAR-003], high).
- `load("//<path>", ...)` fails for a path that escapes `.meow/` [REQ-2408]
  (from docs/spec/starlark.md [R-STAR-004], high).
- `load("@<pkg>//<path>", ...)` with a name no declaration gives fails saying so
  [REQ-2411] (from docs/spec/starlark.md [R-STAR-005], high).
- An import cycle fails with the cycle listed in order [REQ-2412] (from
  docs/spec/starlark.md [R-STAR-006], high).

### Modules

- A `re` pattern that doesn't compile fails at the call with the pattern and the
  reason [REQ-2420] (from docs/spec/starlark.md [R-STAR-013], high).
- Text that `yaml`, `toml`, `csv` or `xml` rejects fails at the call with the
  reason and is never returned as a partial value [REQ-2429] [REQ-2430] (from
  docs/spec/starlark.md [R-STAR-015], high).
- `toml.encode` given a value holding `None` fails saying TOML has no null, and
  drops no key [REQ-2529] (from https://github.com/retran/meowg1k/pull/141,
  high).
- A CSV record whose length disagrees with the header fails naming the record
  number [REQ-2435] (from docs/spec/starlark.md [R-STAR-017], high).
- An `http` call fails when its deadline passes, and cancelling the run leaves
  no request in flight [REQ-2449] [REQ-2451] (from docs/spec/starlark.md
  [R-STAR-022], high).
- An `http` URL that doesn't begin with `http://` or `https://` is refused by
  name before any request [REQ-2544] (from
  https://github.com/retran/meowg1k/pull/145, high).
- An `http` call that never reached a response - a name that doesn't resolve, a
  refused connection, a deadline - fails with what went wrong [REQ-2455] (from
  docs/spec/starlark.md [R-STAR-023], high).
- An `index` call in a workspace that declares no index fails saying so and
  chooses no model. A `query` against an index never built fails saying to build
  it, and never returns an empty list [REQ-2459] [REQ-2460] [REQ-2461]
  [REQ-2462] (from docs/spec/starlark.md [R-STAR-025], high).
- `search.code` in a workspace with no usable index fails saying why [REQ-2549]
  (from crates/meow-cli/src/index.rs:403, high).
- `fs.remove` refuses the workspace root and every path inside `.meow/.data/`
  [REQ-2537] [REQ-2538] (from https://github.com/retran/meowg1k/pull/135, high).
- `fs.glob` with a pattern that isn't a valid glob fails naming it [REQ-2552]
  (from crates/meow-star/src/capability.rs:150, high).
- `@std//git` refuses a revision or a path a caller supplies that begins with
  `-` [REQ-2541] (from https://github.com/retran/meowg1k/pull/136, high).

### Declarations

- A `meow.*` declaration call inside a handler fails [REQ-2476] (from
  docs/spec/starlark.md [R-STAR-030], high).
- Two providers, models, agents or tools with the same name fail naming both
  declaration sites [REQ-2477] (from docs/spec/starlark.md [R-STAR-031], high).
- A declaration naming a provider, model or tool that doesn't exist fails at
  load time [REQ-2478] (from docs/spec/starlark.md [R-STAR-032], high).
- A command whose name collides with a built-in fails at load time naming the
  collision, and the built-in stays in place [REQ-2480] [REQ-2481] (from
  docs/spec/starlark.md [R-STAR-033], high).
- An agent naming an embedding model, or an index naming a chat model, fails at
  load time naming the model and its kind [REQ-2483] (from docs/spec/starlark.md
  [R-STAR-034], high).
- `env.require` naming an unset variable fails the load, naming the variable
  [REQ-2534] (from docs/design/0.3.0-starlark-api.md section 4, high).
- An index command in a workspace that declares no index fails saying so and
  chooses no model [REQ-2487] (from docs/spec/starlark.md [R-STAR-035], high).

### Agents

- `meow.parallel` rejects anything that isn't an invocation, a Starlark function
  included, explaining that Starlark values can't cross a thread boundary
  [REQ-2495] (from docs/spec/starlark.md [R-STAR-044], high).
- Frontmatter carrying `system` fails [REQ-2497] (from docs/spec/starlark.md
  [R-STAR-050], high).
- Frontmatter that isn't valid YAML, or carries a key outside the allowed set,
  fails at load time with the file and line [REQ-2500] (from
  docs/spec/starlark.md [R-STAR-052], high).

### Schemas and evaluation

- A schema naming a required field it doesn't declare fails when it is built
  [REQ-2517] (from docs/spec/starlark.md [R-STAR-071], high).
- `print` fails with an error directing the caller to `ctx.out` [REQ-2522] (from
  docs/spec/starlark.md [R-STAR-082], high).
- During declaration, every runtime module other than `load` and `@std//env`
  fails saying it is unavailable during declaration [REQ-2525] (from
  docs/spec/starlark.md [R-STAR-084], high).

### Diagnostics

- Every load-time and run-time error carries the file, line and column of the
  Starlark expression that caused it [REQ-2526] (from docs/spec/starlark.md
  [R-STAR-090], high).
- An error naming an unknown parameter, module or type suggests the closest
  declared name when one is within a small edit distance [REQ-2527] (from
  docs/spec/starlark.md [R-STAR-091], high).
