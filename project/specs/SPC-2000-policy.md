---
id: SPC-2000
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-2000, REQ-2001, REQ-2002, REQ-2003, REQ-2004, REQ-2005, REQ-2006, REQ-2007, REQ-2008, REQ-2009, REQ-2010, REQ-2011, REQ-2012, REQ-2013, REQ-2014, REQ-2015, REQ-2016, REQ-2017, REQ-2018, REQ-2019, REQ-2020, REQ-2021, REQ-2022, REQ-2023, REQ-2024, REQ-2025, REQ-2026, REQ-2027, REQ-2028, REQ-2029, REQ-2030, REQ-2031, REQ-2032, REQ-2033, REQ-2034, REQ-2035, REQ-2036, REQ-2037, REQ-2038, REQ-2039, REQ-2040, REQ-2041, REQ-2042, REQ-2043, REQ-2044, REQ-2045, REQ-2046, REQ-2047, REQ-2048, REQ-2049, REQ-2050, REQ-2051, REQ-2052, REQ-2053]
---

# The policy layer that decides whether a tool call runs

## Scope

`meow-policy` decides whether a tool call runs. It matches a call against
declared rules, yields allow, ask or deny, and records why (from
docs/spec/policy.md, high).

It doesn't execute tools, prompt the user or write to the session: the engine
acts on the decision and the renderer asks the human (from docs/spec/policy.md,
high).

The policy layer lives in the runtime, outside any Starlark library (from
docs/spec/policy.md, high).

## Boundary

| Surface | What it is |
| --- | --- |
| `Policy` | The type built from a declaration (from docs/spec/policy.md, high) |
| `evaluate` | The call that returns a decision (from docs/spec/policy.md, high) |
| `meow policy explain` | The command that prints an explanation of a decision (from docs/spec/policy.md, high) |

## Behaviour

### Rules

A rule matches on a tool name pattern [REQ-2000] and can narrow further with a
selector belonging to that tool: `paths` for file tools, `commands` for shell
tools [REQ-2001]. A tool name pattern is an exact name such as `fs.read` or ends
in a trailing wildcard such as `fs.*` [REQ-2002]; a leading wildcard isn't
supported [REQ-2003].

A `paths` selector matches an absolute path with symlinks already resolved, with
glob semantics where `**` crosses directory boundaries [REQ-2004]. The caller
resolves the path before evaluation [REQ-2005], and the tool then acts on the
exact path the policy evaluated [REQ-2006] without resolving it a second time
[REQ-2007].

A `commands` selector matches the full command line as one string, with glob
semantics [REQ-2008]. Network tools support a `hosts` selector matching the host
of the request [REQ-2015].

A call that touches several paths is evaluated once per resolved path
[REQ-2010].

### Decisions

Evaluation checks deny rules first, then ask rules, then allow rules [REQ-2016],
and returns the first match [REQ-2017]. Each decision names the rule that
produced it, or records that no rule matched [REQ-2019].

Evaluation touches no filesystem, network or clock [REQ-2020], and returns the
same decision for the same resolved call, the same policy and the same set of
session grants [REQ-2021].

### Ask

An approval prompt shows the tool name and the exact arguments the call would
use, verbatim [REQ-2023], with no summary or paraphrase [REQ-2024], and names
the rule that caused it [REQ-2025].

An approval granted as "always" applies to the current process only [REQ-2026],
and the policy layer writes no grant back to any file [REQ-2027].

An approval prompt waits indefinitely by default [REQ-2028]. A timeout can be
configured [REQ-2029].

### Narrowing

An agent can declare a policy of its own [REQ-2031]. For every call, the
effective decision is the more restrictive of the workspace policy and the agent
policy, ordering `deny` above `ask` above `allow` [REQ-2032]. An agent policy
doesn't allow a call the workspace policy denies [REQ-2033] and doesn't turn an
`ask` into an `allow` [REQ-2034].

A sub-agent inherits its caller's effective policy [REQ-2035] and can narrow it
further by the same rule [REQ-2036].

### Enforcement

The engine evaluates the policy before the tool executes [REQ-2037]. Every
evaluation produces a `Policy` session event, whatever the decision [REQ-2040].

A tool call reaches the policy with the arguments named `path`, `file`, `paths`
and `files` as its paths, each resolved against the workspace root [REQ-2050],
the argument named `command` as its command line [REQ-2051] and the host of the
argument named `url` as its host [REQ-2052]. It reaches the policy as a write
when the tool's name contains `write`, `remove`, `append` or `mkdir`, and as a
read otherwise [REQ-2053].

### Sensitive values

A rule can mark an argument or a result field sensitive [REQ-2041]. A value
marked sensitive is redacted in the approval prompt, the transcript and every
export [REQ-2042], and is never written to the session log in the clear
[REQ-2043]. Redaction replaces the value with a fixed placeholder [REQ-2044] and
doesn't reveal its length [REQ-2045].

### Explanation

`meow policy explain <tool> <argument>` returns the rule decision a real call
with those arguments would receive: `allow`, `ask` or `deny` [REQ-2046]. It
doesn't predict how an `ask` would be answered [REQ-2047]. The explanation names
the matching rule and the file and line it was declared on [REQ-2048], and
reports how many higher-precedence rules were checked without matching
[REQ-2049].

## Failure paths

| Condition | What happens |
| --- | --- |
| A selector that no tool matching the rule's name pattern supports | Building the policy fails, before any call is evaluated [REQ-2009] |
| A call matches no rule | The call is denied [REQ-2018] |
| A read hits a denied path | The read returns the allowed paths [REQ-2011] and names every path it skipped [REQ-2012] |
| A write hits a denied path | The whole write is denied [REQ-2013] and no path is written [REQ-2014] |
| An `ask` decision when standard input isn't a terminal, or when the invocation set `--yes` | The decision resolves to `deny` [REQ-2022] |
| A configured approval timeout expires | The prompt resolves to `deny` [REQ-2030] |
| A decision is `deny` | The engine doesn't execute the tool [REQ-2038] and returns a message to the model naming the tool and stating that policy denied it [REQ-2039] |
