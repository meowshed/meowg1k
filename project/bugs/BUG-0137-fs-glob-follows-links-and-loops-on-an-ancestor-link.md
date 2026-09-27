---
id: BUG-0137
artifact: bug
status: approved
severity: minor
violates: REQ-2550
found: 2026-09-27
revised: 2026-09-27
issue: 260
---

# `fs.glob` follows linked directories and doesn't finish on two ancestor links

`fs.glob` descends into every directory `Path::is_dir` accepts, which follows
symbolic links. One link to an ancestor returns each file many times over, and
two such links make the walk grow faster than it ends.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments; `cargo build -p meow-cli`, `HOME` set to
a scratch directory.

1. Create `.meow/meow.star`:

   ```python
   load("@std//fs", "glob")
   def go(ctx):
       found = glob("**/*.txt")
       ctx.out.write("%d matches, first %s" % (len(found), found[:3]))
       return "true"
   meow.command(meow.tool(name = "go", about = "s", run = go))
   ```

2. Create `sub/a.txt` and `ln -s .. sub/up`, run `meow trust`, then `meow go`.
   It prints `33 matches, first ["sub/a.txt", "sub/up/sub/a.txt",
   "sub/up/sub/up/sub/a.txt"]`.
3. Add `ln -s .. sub/up2` and run `meow go` again. It hadn't finished after 60
   seconds and was killed.

## What the system does

`glob` at `crates/meow-star/src/capability.rs:143-184` pushes every entry for
which `path.is_dir()` holds, with no record of directories already visited and
no check that a link stays inside the workspace. The walk stops only when the
path grows too long for the operating system.

## What it should do, and why

REQ-2550: "`fs.glob` MUST return, sorted, every file in the workspace whose
path relative to the workspace root matches the pattern." `sub/a.txt` is one
file, and the call should return it once, and return at all. The walk should
not follow directory links, or should track visited directories by their
canonical path.

## Triage

The defect enters in `fs.glob`. It's minor because a link to an ancestor inside
a workspace is unusual; where one exists, the handler hangs.

## Closed by

A test in `crates/meow-star/tests/running.rs`, named for example
`glob_returns_a_file_once_through_an_ancestor_link`, running step 3 and
expecting one match within a second.
