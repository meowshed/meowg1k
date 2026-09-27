---
id: SPC-1000
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-1000, REQ-1001, REQ-1002, REQ-1003, REQ-1004, REQ-1005, REQ-1006, REQ-1007, REQ-1008, REQ-1009, REQ-1010, REQ-1011, REQ-1012, REQ-1013, REQ-1014, REQ-1015, REQ-1016, REQ-1017, REQ-1018, REQ-1019, REQ-1020, REQ-1021, REQ-1022, REQ-1023, REQ-1024, REQ-1025, REQ-1026, REQ-1027, REQ-1028, REQ-1029, REQ-1030, REQ-1031, REQ-1032, REQ-1033, REQ-1034, REQ-1035, REQ-1036, REQ-1037, REQ-1038, REQ-1039, REQ-1040, REQ-1041, REQ-1042, REQ-1043, REQ-1044, REQ-1045, REQ-1046, REQ-1047, REQ-1048, REQ-1049, REQ-1050, REQ-1051, REQ-1052, REQ-1053, REQ-1054, REQ-1055, REQ-1056, REQ-1057, REQ-1058, REQ-1059, REQ-1060, REQ-1061, REQ-1062, REQ-1063, REQ-1064, REQ-1065, REQ-1066, REQ-1067, REQ-1068, REQ-1069, REQ-1070, REQ-1071, REQ-1072, REQ-1073, REQ-1074, REQ-1075, REQ-1076, REQ-1077, REQ-1078]
---

# The agent loop in meow-agent

## Scope

`meow-agent` runs the loop: it sends a prompt and the tool schemas to a model,
executes the tool calls the model asks for, feeds the results back, and stops
when the model is done or a bound is reached. It owns budgets, cancellation,
compaction and the outcome it returns. (from docs/spec/agent.md, high)

It doesn't know Starlark exists, doesn't render anything and doesn't talk to a
vendor API. `meow-star` declares agents to it, `meow-ui` observes it and
`meow-llm` carries its requests. (from docs/spec/agent.md, high)

## Boundary

| Surface | What it is |
| --- | --- |
| `AgentSpec` | The declaration of an agent the engine runs |
| `Engine::run` | A run of an agent (from crates/meow-agent/src/engine.rs, high) |
| `Outcome` | What every run returns (from crates/meow-agent/src/outcome.rs, high) |
| `Sink` | The trait the engine emits every transition to (from crates/meow-agent/src/event.rs, high) |
| Engine errors | The errors the engine returns |

## Behaviour

### Outcome

- Every run that starts returns an `Outcome`, whatever stopped it
  [REQ-1000] (from docs/spec/agent.md [R-AGENT-001], high).
- The stop reason is one of `finished`, `budget`, `cancelled`, `denied`,
  `tool_aborted` and `failed` [REQ-1001] (from docs/spec/agent.md [R-AGENT-002],
  high).
- `Outcome.text` carries the model's last text output for every stop
  reason, including empty text [REQ-1002] (from docs/spec/agent.md
  [R-AGENT-003], high).
- An outcome carries the step transcript, the usage totals, the session
  identifier and a detail string naming the budget axis, the tool or the rule
  that stopped the run [REQ-1003] (from docs/spec/agent.md [R-AGENT-004], high).
- A model response with no tool calls stops the run with `finished`, empty text
  included [REQ-1004]; empty text isn't a failure [REQ-1005], and the detail
  records that the model returned no text [REQ-1006] (from docs/spec/agent.md
  [R-AGENT-005], high).
- With error policy `abort`, a policy denial stops the run with `denied`
  [REQ-1007] and a tool error stops it with `tool_aborted` [REQ-1008] (from
  docs/spec/agent.md [R-AGENT-006], high).

### Budget

- A budget bounds a run on total tokens, steps, wall-clock duration and
  estimated cost [REQ-1009] (from docs/spec/agent.md [R-AGENT-010], high).
- A run stops with `budget` as soon as any configured axis is reached
  [REQ-1010], and the outcome names the axis [REQ-1011] (from docs/spec/agent.md
  [R-AGENT-011], high).
- An unset axis is unbounded [REQ-1012], and a spec with no budget takes the
  configured default [REQ-1013] (from docs/spec/agent.md [R-AGENT-012], high).
- The default budget is 200,000 tokens, 40 steps and 30 minutes, with cost
  unbounded [REQ-1019] (from docs/spec/agent.md [R-AGENT-016], high).
- A sub-agent's spend counts against its caller's remaining budget, transitively
  [REQ-1014], and a sub-agent never gets a budget larger than that remainder
  [REQ-1017] (from docs/spec/agent.md [R-AGENT-013] [R-AGENT-014], high).
- The engine reserves budget before a call [REQ-1015], so concurrent invocations
  sharing one budget can't overspend it [REQ-1016] (from docs/spec/agent.md
  [R-AGENT-017], high).
- The engine checks the budget before each model call and after each tool result
  [REQ-1018] (from docs/spec/agent.md [R-AGENT-015], high).
- A budget can cap cost only when something supplies the price of the calls it
  bounds [REQ-1077], and that price is the model's declared one [REQ-1078]
  (from https://github.com/meowshed/meowg1k/issues/168, high).

### Request ceilings

- A provider or a model declaration accepts a ceiling of requests per minute
  and a ceiling of requests per day [REQ-1073] (from
  https://github.com/meowshed/meowg1k/issues/167, high).
- Requests are counted in the workspace database, so every invocation in the
  workspace counts against the same ceiling, whichever process it runs in
  [REQ-1074] (from https://github.com/meowshed/meowg1k/issues/167, high).
- A request that would exceed a ceiling is refused before the call is made
  [REQ-1075], and the refusal names the limit it hit and when that limit frees
  up [REQ-1076] (from https://github.com/meowshed/meowg1k/issues/167, high).

### Tool calls

- The engine validates the arguments against the tool's schema before it invokes
  the tool [REQ-1020] (from docs/spec/agent.md [R-AGENT-020], high).
- A required argument the model omitted gets no default or zero value
  [REQ-1021]; the engine tells the model the argument's name and type [REQ-1022]
  and continues the run [REQ-1023] (from docs/spec/agent.md [R-AGENT-021],
  high).
- An omitted optional argument takes its declared default [REQ-1025], and one
  with no default stays absent [REQ-1026] (from docs/spec/agent.md
  [R-AGENT-022], high).
- With error policy `report`, a tool error goes back to the model as a tool
  result [REQ-1029] and the run continues [REQ-1030] (from docs/spec/agent.md
  [R-AGENT-024], high).
- Tool calls within one model response run in the order the model returned them
  [REQ-1032] (from docs/spec/agent.md [R-AGENT-025], high).

### Cancellation

- Every run accepts a cancellation token [REQ-1033] and checks it before each
  model call, before each tool call and while awaiting either [REQ-1034] (from
  docs/spec/agent.md [R-AGENT-030], high).
- Cancelling a parent cancels its running sub-agents [REQ-1038] (from
  docs/spec/agent.md [R-AGENT-032], high).

### Compaction

- When the rebuilt message list exceeds the configured fraction of the model's
  context window, the engine compacts before the next model call [REQ-1039]
  (from docs/spec/agent.md [R-AGENT-040], high).
- Compaction keeps the configured number of most recent messages verbatim
  [REQ-1040] (from docs/spec/agent.md [R-AGENT-041], high).
- Compaction records a `Compaction` session event [REQ-1041] and deletes no
  events [REQ-1042] (from docs/spec/agent.md [R-AGENT-042], high).
- Compaction summarises the superseded range with a model call [REQ-1044],
  records the tokens the summary saved [REQ-1045] and never drops a message it
  didn't summarise [REQ-1046] (from docs/spec/agent.md [R-AGENT-044], high).
- The compaction policy accepts a model of its own [REQ-1047] and falls back to
  the agent's model [REQ-1048] (from docs/spec/agent.md [R-AGENT-045], high).

### Sub-agents

- An agent used as a tool runs in its own session, with the calling session
  recorded as its parent [REQ-1049] (from docs/spec/agent.md [R-AGENT-050],
  high).
- A sub-agent's outcome goes back to the calling model as a tool result with its
  text, its stop reason and, when it declared an output schema, its parsed value
  [REQ-1050] (from docs/spec/agent.md [R-AGENT-051], high).
- A sub-agent's message list holds none of its caller's messages [REQ-1053];
  only the task and the arguments the caller passed cross [REQ-1054] (from
  docs/spec/agent.md [R-AGENT-053], high).

### Concurrency

- The engine runs a list of agent invocations concurrently and returns the
  results in the order given [REQ-1055] (from docs/spec/agent.md [R-AGENT-060],
  high).
- Concurrent invocations share the caller's budget [REQ-1058] (from
  docs/spec/agent.md [R-AGENT-062], high).

### Events

- The engine emits run start and end, step start and end, text and thinking
  deltas, tool call start and end, policy decisions and usage to its `Sink`
  [REQ-1060] (from docs/spec/agent.md [R-AGENT-070], high).
- A sink can decline delta events [REQ-1061], and the engine then delivers the
  completed text once per step [REQ-1062] (from docs/spec/agent.md
  [R-AGENT-073], high).
- The engine runs with no sink attached [REQ-1069] (from docs/spec/agent.md
  [R-AGENT-072], high).

### Structured output

- When a spec declares an output schema, the parsed value is present when the
  stop reason is `finished` [REQ-1070] and absent for every other stop reason
  [REQ-1071] (from docs/spec/agent.md [R-AGENT-080], high).

## Failure paths

| Condition | What happens |
| --- | --- |
| Any stop condition, cancellation included | The run returns an outcome, never an error [REQ-1000] (from docs/spec/agent.md [R-AGENT-001], high) |
| Policy denies a call under `abort` | The run stops with `denied` [REQ-1007] (from docs/spec/agent.md [R-AGENT-006], high) |
| A tool returns an error under `abort` | The run stops with `tool_aborted` [REQ-1008] [REQ-1031] (from docs/spec/agent.md [R-AGENT-006] [R-AGENT-024], high) |
| A budget axis is reached | The run stops with `budget` and names the axis [REQ-1010] [REQ-1011] (from docs/spec/agent.md [R-AGENT-011], high) |
| The model omits a required argument | The engine names the argument and its type to the model and continues, under every error policy [REQ-1022] [REQ-1023] [REQ-1024] (from docs/spec/agent.md [R-AGENT-021], high) |
| The model calls a tool the agent wasn't given | The model gets an error, and no wider registry is searched [REQ-1027] [REQ-1028] (from docs/spec/agent.md [R-AGENT-023], high) |
| The run is cancelled | The run stops with `cancelled`, writes its `Finished` event and returns the transcript so far [REQ-1035] [REQ-1036] [REQ-1037] (from docs/spec/agent.md [R-AGENT-031], high) |
| Compaction fails | The run fails [REQ-1043] (from docs/spec/agent.md [R-AGENT-043], high) |
| A sub-agent would exceed the maximum nesting depth | The engine refuses to start it and reports a tool error, not a panic [REQ-1051] [REQ-1052] (from docs/spec/agent.md [R-AGENT-052], high) |
| One concurrent invocation fails | The others continue, and its result is an outcome with a stop reason other than `finished` [REQ-1056] [REQ-1057] (from docs/spec/agent.md [R-AGENT-061], high) |
| The shared budget is exhausted during concurrent invocations | The engine starts no new invocation [REQ-1059] (from docs/spec/agent.md [R-AGENT-062], high) |
| The sink returns an error | For every event kind alike, the engine stops delivering to that sink, records a `Note` event naming the error and continues the run [REQ-1063] [REQ-1064] [REQ-1065] [REQ-1066] [REQ-1067] [REQ-1068] (from docs/spec/agent.md [R-AGENT-071], high) |
| A budget caps cost for an agent whose model declares no price | The workspace fails to load [REQ-1077] [REQ-1078] (from https://github.com/meowshed/meowg1k/issues/168, high) |
| A request would exceed a declared ceiling | The call isn't made, and the refusal names the ceiling and when it frees up [REQ-1075] [REQ-1076] (from https://github.com/meowshed/meowg1k/issues/167, high) |
| An earlier process in the workspace spent the ceiling | A request in the next process is refused, because the count lives in the workspace database [REQ-1074] [REQ-1075] (from https://github.com/meowshed/meowg1k/issues/167, high) |
| The final response fails schema validation | The engine retries it per [R-LLM-051], then stops with `failed` [REQ-1072] (from docs/spec/agent.md [R-AGENT-081], high) |
