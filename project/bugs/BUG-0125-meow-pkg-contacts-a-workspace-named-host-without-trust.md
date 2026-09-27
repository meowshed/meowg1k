---
id: BUG-0125
artifact: bug
status: approved
severity: major
violates: REQ-1223
found: 2026-09-27
revised: 2026-09-27
issue: 248
---

# `meow pkg update` contacts a host an untrusted workspace names

`meow pkg` is exempt from the trust gate, so in a freshly cloned workspace
`meow pkg update` downloads from whatever URL `meow.package` gives, before the
person has agreed to anything.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments; `cargo build -p meow-cli`, `HOME` set to
an empty scratch directory.

1. Listen on `127.0.0.1:18731` and log the first request line.
2. Create `.meow/meow.star`:

   ```python
   meow.package(name = "acme", source = "http://127.0.0.1:18731/acme-{version}.tar.gz", version = "1.0.0")
   ```

3. Run `meow pkg update` without running `meow trust`.
4. The listener logs `GET /acme-1.0.0.tar.gz HTTP/1.1`. `meow trust --list`
   prints `no workspaces are trusted`.

## What the system does

`gate_on_trust` at `crates/meow-cli/src/wire.rs:149-175` lets `pkg` through
with the describing commands. `fetch` at `crates/meow-cli/src/fetch.rs:84-112`
sends a GET to `package.source` with the version substituted, any host, any
port, over plain HTTP.

## What it should do, and why

REQ-1223: "The first invocation in a workspace this machine has not agreed to
MUST ask once." REQ-1222 gives the reason: "A `.meow/` directory is executable
code with tool access." A network request to a host the workspace chose is an
action, so `meow pkg update` and `meow pkg fetch` should ask; `meow pkg list`,
which only describes, can stay exempt.

## Triage

The defect enters in `gate_on_trust`. ADR-0459 records the exemption and notes
no requirement names it. It's major because the requirement's question isn't
asked before the binary reaches the network for the workspace, which lets a
cloned repository probe the person's local network. The same exemption for
`check`, `doctor`, `policy`, `models` and `providers` is a gap in REQ-1223 to
close at the requirements step, since those only describe.

## Closed by

A test in `crates/meow-cli/tests/pkg.rs`, named for example
`pkg_update_asks_for_trust_first`, that runs `meow pkg update` in an untrusted
workspace with no terminal and expects exit 7 and no request at the source.
