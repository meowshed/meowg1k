---
id: SPC-2800
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-2800, REQ-2801, REQ-2802, REQ-2803, REQ-2804, REQ-2805, REQ-2806, REQ-2807, REQ-2808, REQ-2809, REQ-2810, REQ-2811, REQ-2812, REQ-2813, REQ-2814, REQ-2815, REQ-2816, REQ-2817, REQ-2818, REQ-2819, REQ-2820, REQ-2821, REQ-2822, REQ-2823, REQ-2824, REQ-2825, REQ-2826, REQ-2827, REQ-2828, REQ-2829, REQ-2830, REQ-2831, REQ-2832, REQ-2833, REQ-2834, REQ-2835, REQ-2836, REQ-2837, REQ-2838, REQ-2839, REQ-2840, REQ-2841, REQ-2842, REQ-2843, REQ-2844, REQ-2845, REQ-2846, REQ-2847, REQ-2848, REQ-2849, REQ-2850, REQ-2851, REQ-2852, REQ-2853, REQ-2854, REQ-2855, REQ-2856, REQ-2857, REQ-2858, REQ-2859]
---

# The terminal renderers and the command line

## Scope

`meow-ui` turns the engine's event stream into something a person or a program
can read, and `meow-cli` turns a command line into a run. Together they're
everything a user touches (from docs/spec/tui.md, high).

`meow-ui` depends on `meow-core` for the event types and on nothing else in the
workspace, and doesn't know the engine exists (from docs/spec/tui.md, high).

## Boundary

| Surface | What it is |
| --- | --- |
| stdout and stderr | What each renderer writes (from docs/spec/tui.md, high) |
| The exit code | The process status a shell branches on (from docs/spec/tui.md, high) |
| `--format json` | The event stream as one JSON object per line (from docs/spec/tui.md, high) |
| The approval prompt | What a person sees when policy asks (from docs/spec/tui.md, high) |

## Behaviour

### Renderer selection

The runtime chooses the renderer and a script never does [REQ-2800]. `--format
json` selects the JSON renderer whatever the terminal is [REQ-2801]. A stdout
that isn't a terminal, `NO_COLOR` or `--color=never` selects the plain renderer
[REQ-2802], and in every other case the inline terminal renderer runs
[REQ-2803]. All three renderers accept the same event stream [REQ-2804], and
each runs from a recorded log with no terminal attached [REQ-2805].

### Inline rendering

The terminal renderer draws in an inline viewport [REQ-2806] and never switches
to the alternate screen buffer [REQ-2807]. A finalized transcript line goes into
scrollback [REQ-2808] and is never redrawn [REQ-2809]; only the live region is
redrawn [REQ-2810]. The live region shows the current tool, the elapsed time,
the step count and the budget consumed [REQ-2811]. It's three rows high
[REQ-2817] and keeps that height [REQ-2818]. A terminal resize reflows the live
region and nothing else [REQ-2814].

An approval prompt is committed to the transcript and isn't drawn in the live
region [REQ-2819]; while the prompt is open, the live region says an answer is
awaited [REQ-2820].

Diagnostics from the logging layer enter scrollback in order [REQ-2815] and are
never drawn over the live region [REQ-2816].

### Plain rendering

The plain renderer emits no ANSI escape sequences [REQ-2821] and never moves the
cursor [REQ-2822]. It carries the same information as the terminal renderer:
every step, every tool call with its policy decision, and the totals [REQ-2823].

### JSON rendering

The JSON renderer emits one JSON object per line, each with a `type` field
[REQ-2824], and the stream opens with an event carrying the schema version
[REQ-2825]. The live stream and `meow session export --format json` share one
schema definition and one version number [REQ-2826], and every persisted event
kind serialises identically in both [REQ-2827]. A kind that exists only while a
run is in flight, such as a text delta, is absent from an export [REQ-2830], and
an export emits only kinds the live stream can emit [REQ-2831]. With `--format
json`, stdout carries only the event stream [REQ-2828] and diagnostics go to
stderr [REQ-2829].

### Script output

`ctx.out` exposes exactly `write`, `markdown`, `note`, `warn`, `error`, `step`,
`table`, `diff`, `finding` and `json` [REQ-2832], and no call that positions the
cursor, draws a frame or paginates [REQ-2833]. Each call produces a typed event
that all three renderers handle [REQ-2834].

### Interaction

`ctx.ask` exposes exactly `text`, `confirm` and `select` [REQ-2835].

### Approval

An approval prompt overwrites nothing already in the transcript [REQ-2838] and
stays readable after it's answered [REQ-2839]. It shows the tool name, the exact
arguments, the matching rule, and the agent and step [REQ-2840], and offers
once, always, deny and stop [REQ-2841]. Choosing "always" lasts for the current
process only [REQ-2842].

### Command surface

A user's agents and tools sit at the top level of the command line with no
prefix [REQ-2843]. Built-in commands are grouped under `session`, `auth`,
`index`, `pkg` and `policy`, except `init`, `run`, `check`, `models`,
`providers`, `doctor`, `trust`, `completions` and `version` [REQ-2844].

`--dry-run` evaluates policy and plans tool calls without executing any, feeding
the model a placeholder result for each [REQ-2845]. Its transcript records what
would have run and how policy would have decided [REQ-2846], and states that the
run diverges from a real one after the first tool call [REQ-2847].

`--yes` resolves every `ask` decision to `deny` [REQ-2848] and makes no decision
more permissive [REQ-2849]. `--continue` resumes the most recent session of the
invoked command in this workspace [REQ-2850].

### Exit codes

The exit code comes from the stop reason and the handler's return value, by the
table in [REQ-2852]. No code is shared between two stop reasons [REQ-2853], and
every stop reason maps to exactly one code [REQ-2854].

### Theme and accessibility

`NO_COLOR` turns colour off whatever the theme declares [REQ-2855]. The renderer
detects the colour depth and quantises the palette to it [REQ-2856]. Colour is
never the only carrier of meaning [REQ-2857]: every severity and status also
carries a word or a sigil [REQ-2858]. A terminal that can't be shown to support
the box-drawing and spinner characters gets the ASCII fallback [REQ-2859].

## Failure paths

On any exit the process can observe, an interrupt included, the live region is
replaced by a final line naming the stop reason [REQ-2812]. On an exit the
process can't observe, the transcript already in scrollback stays valid
[REQ-2813].

Any `ctx.ask` call fails with an error when stdin isn't a terminal or `--yes`
was given [REQ-2836], and doesn't block [REQ-2837].

`--continue` fails when the invoked command has no session in this workspace,
and doesn't start a fresh run [REQ-2851].

A usage error, a configuration error, a provider or credential failure and each
failing stop reason exit with their own code from the table in [REQ-2852].
