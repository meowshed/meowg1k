---
id: ADR-0607
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1650]
supersedes: []
---

# 0607. HTTPS trusts the operating system's root certificates

## Decision

The provider transport verifies a vendor against the machine's native root
certificate store and not the bundled Mozilla set (from
https://github.com/retran/meowg1k/pull/125, high).

Once this is accepted, it holds for every HTTP client in the binary: `meow-llm`,
`meow-star`'s `http` module and `meow-cli`'s package fetch all build `reqwest`
with `rustls-tls-native-roots` and no default features (from
crates/meow-llm/Cargo.toml:14, crates/meow-star/Cargo.toml:22 and
crates/meow-cli/Cargo.toml:32, high).

## Why

A developer behind a TLS-inspecting proxy otherwise gets "certificate unknown"
with no way to fix it, and honouring the machine's trust store is what every
other developer tool on that machine does (from
https://github.com/retran/meowg1k/pull/125, high). It also avoids
`webpki-roots`, whose `CDLA-Permissive-2.0` licence `cargo deny` rejects; that
prompted the question and isn't the reason for the answer (from
https://github.com/retran/meowg1k/pull/125, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep only the `Recorded` transport | Every provider test stays deterministic and nothing reaches the network (from https://github.com/retran/meowg1k/pull/125, high) | Nothing outside a test can use a provider (from https://github.com/retran/meowg1k/pull/125, high) |
| The bundled Mozilla set through `webpki-roots` | Works in a minimal container with no CA bundle (from https://github.com/retran/meowg1k/pull/125, high) | Fails behind a TLS-inspecting proxy with no fix, and its licence fails `cargo deny` (from https://github.com/retran/meowg1k/pull/125, high) |

## What it costs

A minimal container with no CA bundle can't reach a vendor, where the bundled
set would have worked (from https://github.com/retran/meowg1k/pull/125, high).

## What would reverse it

- A supported install target ships without a CA bundle, or `cargo deny` comes
  to accept `webpki-roots`' licence and a bundled fallback is wanted (reasoned
  from https://github.com/retran/meowg1k/pull/125, low).

## Consequences

- Proxy settings come only from what `reqwest` reads in the environment,
  `HTTPS_PROXY` and `NO_PROXY` (from https://github.com/retran/meowg1k/pull/125,
  high).

## How I will know it was realised

1. `cargo tree -e features -i rustls-native-certs` shows each of the three
   crates pulling it in, and `cargo deny check licenses` passes (from
   crates/meow-llm/Cargo.toml:14, high).

## What this does not settle

- Whether the binary should say, when a connection fails for want of roots,
  that the machine has no CA bundle (reasoned from
  crates/meow-llm/src/http.rs:44-53, low).
