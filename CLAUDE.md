# CLAUDE.md

Instructions for AI coding agents (Claude Code and others) working in this
repository. Humans are welcome to read it too. `AGENTS.md`-style tools can
symlink to this file.

## What this project is

faultline is a Rust CLI that compares two git revisions and reports what got
**worse**: architecture violations, complexity increase, duplication, tests
falling behind. See `README.md` for the why and the roadmap.

This project is itself an experiment in AI-assisted development, so how you
work here matters as much as what you produce.

## Layout

- `crates/faultline-core`: shared types only. No I/O, no parsing, no git.
- `crates/faultline-lang`: language adapters, tree-sitter based.
- `crates/faultline-analyzers`: one module per analyzer, implementing `Analyzer`.
- `crates/faultline-report`: renders findings. No analysis logic here.
- `crates/faultline-cli`: the `faultline` binary. Thin; wires the others together.
- `docs/adr/`: architecture decision records. Read them before changing structure.

Dependencies flow one way: `cli -> report, analyzers -> lang -> core`.
`core` depends on nothing in the workspace. Do not add reverse or sideways
dependencies; if you think you need one, stop and propose an ADR instead.

## Before you finish any change

Run all of these and make sure they pass:

```sh
cargo fmt --all
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

CI runs the same checks with warnings as errors.

## Conventions

- Edition 2024, MSRV 1.85. Clippy `pedantic` is on; fix warnings rather than
  silencing them. An `#[allow]` needs a comment saying why.
- No `unsafe`. It is forbidden at the workspace level.
- Every public item has a doc comment (`missing_docs` is on).
- Library crates return errors; only `faultline-cli` prints and picks exit codes.
- Rule ids are `analyzer/rule`, lowercase, e.g. `complexity/increase`. Treat
  them as a public API: never rename one silently.
- Tests live next to the code (`#[cfg(test)] mod tests`). Every analyzer gets
  fixture-based tests with a before/after pair of source files.

## Things not to do

- Do not add a dependency without saying why in the commit message. Prefer
  well-maintained crates with few transitive dependencies.
- Do not make the core analysis depend on an LLM or the network. AI features
  are optional layers on top (see README, design principle 1).
- Do not report absolute scores. faultline reports deltas.
- Do not write large, sweeping changes. Keep pull requests small and focused;
  faultline would flag a huge one, and so will the reviewer.

## Working style

- For anything non-trivial, write a short plan first (what changes, which
  crates, how it will be tested) and wait for approval.
- If a requirement is ambiguous, ask instead of guessing.
- Commit messages: imperative mood, short subject line, body explains why.
