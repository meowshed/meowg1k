---
id: SPC-1600
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-1600, REQ-1601, REQ-1602, REQ-1603, REQ-1604, REQ-1605, REQ-1606, REQ-1607, REQ-1608, REQ-1609, REQ-1610, REQ-1611, REQ-1612, REQ-1613, REQ-1614, REQ-1615, REQ-1616, REQ-1617, REQ-1618, REQ-1619, REQ-1620, REQ-1621, REQ-1622, REQ-1623, REQ-1624, REQ-1625, REQ-1626, REQ-1627, REQ-1628, REQ-1629, REQ-1630, REQ-1631, REQ-1632, REQ-1633, REQ-1634, REQ-1635, REQ-1636, REQ-1637, REQ-1638, REQ-1639, REQ-1640, REQ-1641, REQ-1642, REQ-1643, REQ-1644, REQ-1645, REQ-1646, REQ-1647, REQ-1648, REQ-1649, REQ-1650, REQ-1651, REQ-1652, REQ-1653]
---

# The model provider contract

## Scope

`meow-llm` defines what a model provider is: the request and response types, the
streaming event stream, the error classification and the retry policy. Each
`meow-provider-*` crate implements the trait over one vendor's HTTP API (from
docs/spec/llm.md, high).

It doesn't run an agent loop, decide what tools exist or persist anything. It
turns a request into a response or a typed error (from docs/spec/llm.md, high).

## Boundary

No vendor type crosses this boundary, which is what lets the engine be written
once (from docs/spec/llm.md, high).

| Surface | What it is |
| --- | --- |
| `Provider` trait | Generation, plus the declared capabilities: streaming, tool calling, embeddings, native or emulated structured output [REQ-1600] [REQ-1601] |
| Request and response types | Messages, tool calls, the optional JSON Schema, usage counts [REQ-1607] [REQ-1641] [REQ-1637] |
| Stream event enum | `Text`, `Thinking`, `ToolCallStart`, `ToolCallDelta`, `ToolCallEnd`, `Usage`, `Error`, `Done` [REQ-1617] |
| Error enum | `Transient`, `Fatal`, `QuotaExhausted`, and a distinct cancellation error [REQ-1624] [REQ-1649] |

(from docs/spec/llm.md, high)

## Behaviour

### The trait

A provider implements generation and declares whether it supports streaming,
tool calling and embeddings, and whether its structured output is native or
emulated [REQ-1600] [REQ-1601]. A trait method signature names no type from a
vendor SDK or wire format [REQ-1603]. A provider obtains what it sends to
authenticate per request, not when it is built [REQ-1604], and renews a
credential at most once for that credential's lifetime [REQ-1605].

### Messages and tools

A message carries exactly one role: system, user, assistant or tool [REQ-1607].
An assistant message can carry text and a list of tool calls together
[REQ-1608], and a tool result message carries the identifier of the tool call it
answers [REQ-1609]. A provider assigns an identifier to every returned tool call
that lacks one [REQ-1610], keeps those identifiers unique within a response
[REQ-1611], and removes tool calls repeated with the same identifier before it
returns, whatever the session or history setting [REQ-1612].

A message can carry a cache hint [REQ-1613]. A provider with explicit cache
breakpoints turns the hint into one [REQ-1614], a provider that caches
automatically ignores it [REQ-1615], and the hint never changes the message's
content [REQ-1616].

### Streaming

The stream carries exactly the eight event kinds the boundary lists [REQ-1617].
Aggregating a recorded stream produces the same response value as parsing the
non-streaming body for the same model output [REQ-1618]. A provider with no
native streaming endpoint declares streaming unsupported and synthesises no
events [REQ-1619]. Thinking content arrives as `Thinking` events [REQ-1622] and
stays on the assistant message it belongs to [REQ-1623].

### Errors and retry

Every provider error is exactly one of `Transient`, `Fatal` or `QuotaExhausted`
[REQ-1624]. HTTP 408, 429, 500, 502, 503 and 504, and connection or read
timeouts, are `Transient` [REQ-1625]. HTTP 400, 401, 403, 404 and 422, and any
schema rejection, are `Fatal` [REQ-1626]. A provider tells `QuotaExhausted` from
a rate limit by its own documented signal and never by matching message text
[REQ-1634]; without that signal a 429 is `Transient` [REQ-1635].

Only `Transient` errors are retried [REQ-1627]. Retry uses exponential backoff
with random jitter [REQ-1629], honours a `Retry-After` header [REQ-1630], stops
after a configured maximum attempt count [REQ-1631], and checks the cancellation
token before sleeping and before the next attempt [REQ-1636].

### Usage

A response reports prompt and completion tokens [REQ-1637] and cached prompt
tokens as a separate optional count [REQ-1638]. A provider that doesn't report
caching leaves that count absent, never zero [REQ-1639], and a response from a
provider that reports no usage says so explicitly [REQ-1640].

### Structured output

A request can carry a JSON Schema [REQ-1641]. A provider with native structured
output uses the schema [REQ-1642]; one with emulated structured output requests
JSON and validates the result against the schema [REQ-1643]. A response that
fails validation is retried with the validation error in the follow-up request,
up to a configured attempt count [REQ-1644]. A response that satisfies the
schema is returned parsed, not as a string [REQ-1646].

### Transport

The provider transport verifies a vendor's certificate against the operating
system's root certificate store [REQ-1650]. It gives up connecting after 30
seconds [REQ-1652] and puts no deadline on a whole response [REQ-1653] (from
https://github.com/retran/meowg1k/pull/125, high).

### Cancellation

Every request takes a cancellation token [REQ-1647] and aborts the in-flight
HTTP request when the token fires [REQ-1648].

## Failure paths

| Condition | What happens |
| --- | --- |
| A caller uses a capability the provider doesn't declare | The call fails before any request is sent, naming the provider and the capability [REQ-1602] |
| A credential renewal fails | The request fails, naming the provider and the command that re-authenticates [REQ-1606] |
| The stream consumer raises an error | The request aborts [REQ-1620] and the error reaches the caller unchanged [REQ-1621] |
| A `Fatal` error | It surfaces to the caller on the first occurrence, without delay [REQ-1628] |
| A `QuotaExhausted` error | It surfaces immediately with the provider named [REQ-1632] and isn't retried [REQ-1633] |
| A `Transient` error persists | Retry stops after the configured maximum attempt count [REQ-1631] |
| A response still fails schema validation after the configured attempts | The request fails with the last validation error [REQ-1645] |
| A vendor answers a streaming request with a non-success status | The call fails with the status and the vendor's body, and returns no stream [REQ-1651] |
| The cancellation token fires | The in-flight HTTP request aborts [REQ-1648] and the request returns a distinct cancellation error, never a timeout or a transport error [REQ-1649] |
