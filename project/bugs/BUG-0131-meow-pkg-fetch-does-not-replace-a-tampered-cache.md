---
id: BUG-0131
artifact: bug
status: approved
severity: major
violates: REQ-1812
found: 2026-09-27
revised: 2026-09-27
issue: 254
---

# `meow pkg fetch` doesn't replace a cached package whose contents changed

When a cached package no longer matches `meow.lock`, loading fails and tells
the person to run `meow pkg fetch`. `meow pkg fetch` skips every package whose
cache directory exists, so it reports `fetched 0`, and the workspace stays
broken until someone deletes the cache by hand.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments; `cargo build -p meow-cli`, `HOME` set to
a scratch directory.

1. Serve `acme-1.0.0.tar.gz`, holding `lib.star` with `def hi(): return "hi"`,
   with `python3 -m http.server 18732 --bind 127.0.0.1`.
2. Create `.meow/meow.star`:

   ```python
   meow.package(name = "acme", source = "http://127.0.0.1:18732/acme-{version}.tar.gz", version = "1.0.0")
   load("@acme//lib.star", "hi")
   def go(ctx):
       ctx.out.write(hi())
       return "true"
   meow.command(meow.tool(name = "go", about = "s", run = go))
   ```

3. Run `meow pkg update`, then `meow check`. It prints `ok` and exits 0.
4. Append `# edited` to `.meow/.data/pkg/<hash>/lib.star`.
5. Run `meow check`. It exits 7: "`acme` in the cache does not match
   `meow.lock` ... Run `meow pkg fetch` to replace it."
6. Run `meow pkg fetch`. It prints `fetched 0` and exits 0.
7. Run `meow check`. It fails as in step 5.

## What the system does

`ensure` at `crates/meow-cli/src/fetch.rs:306-308` skips a package when
`cached_at(config_dir, &pin.hash).exists()`, without checking what's inside.
`fetch` does the same after a download, `fetch.rs:169-173`. `meow pkg` reaches
this point because the loader's stand-in covers every strict-load failure,
hash mismatches included, `crates/meow-star/src/loader.rs:137-141`, so the
mismatch isn't reported there either. The advice comes from
`crates/meow-star/src/package.rs:202-209`.

## What it should do, and why

REQ-1812: "`meow pkg fetch` MUST download what the lockfile already pins." A
cache directory whose contents don't hash to the pin doesn't hold what the
lockfile pins, so `meow pkg fetch` should replace it, as the load error
promises.

## Triage

The defect enters in `ensure`, which takes a directory's name for its contents.
REQ-1807 and REQ-1808 hold: the tampered code never runs. It's major because
the documented recovery doesn't work and the person is stuck with a workspace
that won't load.

## Closed by

A test in `crates/meow-cli/tests/pkg.rs`, named for example
`fetch_replaces_a_cached_package_that_no_longer_matches`, running steps 3 to 7
and expecting step 7 to pass.
