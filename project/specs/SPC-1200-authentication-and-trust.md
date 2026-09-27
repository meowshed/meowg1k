---
id: SPC-1200
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-1200, REQ-1201, REQ-1202, REQ-1203, REQ-1205, REQ-1206, REQ-1207, REQ-1208, REQ-1209, REQ-1210, REQ-1211, REQ-1212, REQ-1213, REQ-1214, REQ-1215, REQ-1216, REQ-1217, REQ-1218, REQ-1219, REQ-1220, REQ-1221, REQ-1222, REQ-1223, REQ-1224, REQ-1225, REQ-1226, REQ-1227, REQ-1228, REQ-1229, REQ-1230, REQ-1231, REQ-1232, REQ-1233]
---

# Authentication and trust

## Scope

This part covers where a credential comes from, where it's kept, how a provider
that uses OAuth in place of an API key obtains and refreshes one, and whether
this machine has agreed to run the scripts in a given workspace. (from
docs/spec/auth.md, high)

Credentials and trust are one area because they answer one question twice: what
this process may use, and who said so. A credential answers it for a provider
and trust answers it for a workspace, and both are facts about the machine, not
the repository. (from docs/spec/auth.md, high)

## Boundary

| Surface | What it is |
| --- | --- |
| The platform secret store | Where credentials are kept: Keychain on macOS, Credential Manager on Windows and Secret Service on Linux |
| `~/.meow/auth.json` | The fallback credential store, used only where no platform secret store is running, owned by the user and written only by `meow auth` |
| `~/.meow/trust.json` | The trust record, owned by the user and written only by `meow trust` |
| `meow auth login <provider>`, `meow auth logout <provider>`, `meow auth list` | The commands that manage the credential store |
| `meow trust` | The command that manages the trust record |

Nothing in `.meow/` can read or write either file, and no Starlark builtin
exposes them. (from docs/spec/auth.md, high)

## Behaviour

### Where a credential comes from

- A credential resolves from the declaration's `api_key`, then the global store,
  then the provider's environment variable, and the first that is present and
  not empty wins [REQ-1200] (from docs/spec/auth.md [R-AUTH-001], high).
- Resolution doesn't happen while `.meow/` is evaluated beyond what
  `[R-STAR-084]` allows: a declaration can call `env.get`, and the store is read
  when a provider is built [REQ-1202] (from docs/spec/auth.md [R-AUTH-003],
  high).
- No runtime module exposes the credential store; a handler reads a secret from
  the environment [REQ-1203] (from docs/spec/auth.md [R-AUTH-004], high).

### The store

- Credentials are kept in the platform secret store: Keychain on macOS,
  Credential Manager on Windows and Secret Service on Linux [REQ-1230] (from
  https://github.com/meowshed/meowg1k/issues/166, high).
- A credential is held in memory only for the call that uses it [REQ-1231]
  (from https://github.com/meowshed/meowg1k/issues/166, high).
- Where no platform secret store is running, credentials fall back to
  `~/.meow/auth.json` [REQ-1232], and the credential store says so when it uses
  that file [REQ-1233] (from https://github.com/meowshed/meowg1k/issues/166,
  high).
- The fallback file is created with owner-only permissions [REQ-1205] (from
  docs/spec/auth.md [R-AUTH-011], high).
- A write of the fallback file is atomic [REQ-1207] and never leaves a
  truncated file [REQ-1208] (from docs/spec/auth.md [R-AUTH-012], high).
- `meow auth login <provider>` stores a credential [REQ-1209], `meow auth logout
  <provider>` removes one [REQ-1210], and `meow auth list` shows which providers
  have one without showing any credential [REQ-1211] (from docs/spec/auth.md
  [R-AUTH-013], high).
- No command prints a credential, in full or in part; a listing says that a
  credential is present and when it was stored [REQ-1212] (from
  docs/spec/auth.md [R-AUTH-014], high).

### OAuth providers

- A provider kind that authenticates by OAuth gets its credential through a
  device-code flow: the command shows a code and a URL, waits for the person to
  approve and stores the result [REQ-1213] (from docs/spec/auth.md [R-AUTH-020],
  high).
- The flow honours the interval the authorisation server asks for [REQ-1214]
  (from docs/spec/auth.md [R-AUTH-021], high).
- A refresh token lives in the same store as an API key [REQ-1217], and an
  expired access token is refreshed without asking the person again [REQ-1218]
  (from docs/spec/auth.md [R-AUTH-022], high).

### Trusting a workspace

- The first invocation in a workspace this machine hasn't agreed to shows what
  the workspace declares - its agents, its tools and the policy it asks for
  [REQ-1222] - and asks once [REQ-1223] (from docs/spec/auth.md [R-AUTH-030],
  high).
- Trust is recorded per workspace path in `~/.meow/trust.json` [REQ-1226] and
  can be withdrawn [REQ-1227] (from docs/spec/auth.md [R-AUTH-032], high).
- Loading `.meow/` to ask the question runs nothing beyond declaration
  [REQ-1228] (from docs/spec/auth.md [R-AUTH-033], high).
- A workspace whose declarations changed loses its trust and asks again, because
  what is recorded is what was shown [REQ-1229] (from docs/spec/auth.md
  [R-AUTH-034], high).

## Failure paths

| Condition | What happens |
| --- | --- |
| No platform secret store is running | Credentials are kept in `~/.meow/auth.json`, and the credential store says it is using that file [REQ-1232] [REQ-1233] (from https://github.com/meowshed/meowg1k/issues/166, high) |
| A provider has no credential from any of the three places | It fails naming all three, with the environment variable spelled out [REQ-1201] (from docs/spec/auth.md [R-AUTH-002], high) |
| The fallback file is readable by anyone but the owner | `meow auth` refuses to read it, naming the file and the permission it expects [REQ-1206] (from docs/spec/auth.md [R-AUTH-011], high) |
| The process dies during a write of the fallback file | No truncated file is left [REQ-1208] (from docs/spec/auth.md [R-AUTH-012], high) |
| The authorisation server says the device code expired | The flow stops [REQ-1215] (from docs/spec/auth.md [R-AUTH-021], high) |
| The person presses Ctrl-C during the device-code flow | The flow stops and leaves nothing half-written [REQ-1216] (from docs/spec/auth.md [R-AUTH-021], high) |
| A token refresh fails | The command says so and names the command that re-authenticates [REQ-1219] [REQ-1220] (from docs/spec/auth.md [R-AUTH-022], high) |
| `meow auth login` for an OAuth provider runs with no terminal | It is refused [REQ-1221] (from docs/spec/auth.md [R-AUTH-023], high) |
| A run in an untrusted workspace has no terminal | It fails without proceeding or blocking, and names the command that grants trust [REQ-1224] [REQ-1225] (from docs/spec/auth.md [R-AUTH-031], high) |
