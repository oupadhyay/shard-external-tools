# shard-external-tools Repository Guidance

## Purpose

`shard-external-tools` contains host-free implementations of Shard tools backed
by external services or explicitly supplied processes. It turns explicit tool
inputs, clients, credentials, and process configuration into portable outputs.

Sibling repositories:

- [`shard-v2`](https://github.com/oupadhyay/shard-v2) — desktop host and UI.
- [`shard-tool-api`](https://github.com/oupadhyay/shard-tool-api) — canonical,
  provider-neutral tool contracts.
- [`shard-provider`](https://github.com/oupadhyay/shard-provider) — host-free
  provider wire contracts and transports.

## Ownership

This repository owns:

- stateless transport/parsing for weather, finance, Wikipedia, arXiv, web
  search, and URL reading;
- YouTube video-ID parsing, `yt-dlp` acquisition behind explicit process
  configuration, caption selection and URL validation, caption XML parsing,
  transcript formatting, and bounded final rendering;
- portable external-tool dispatch from `shard-tool-api` invocation inputs.

It does **not** own tool availability, persona filtering, registry composition,
hooks, cache TTLs, parallelism, UI events, persistence, credentials/config
lookup, or heartbeat approval policy. Shard also retains model selection and
long-transcript LLM summarization. The generic dispatcher intentionally returns
`None` for YouTube so host summary composition cannot be bypassed.

## Dependency Rules

The one-way graph is:

```text
shard-v2 ──> shard-tool-api
    ├──────> shard-external-tools ──> shard-tool-api
    └──────> shard-provider ─────────> shard-tool-api
```

- This crate may depend on `shard-tool-api` at one immutable Git revision.
- It must never depend on `shard-provider`.
- Do not add Tauri, UI emitters, SQLite/persistence, OS keychain/config lookup,
  host model calls, memory/session state, retry orchestration, persona policy,
  or heartbeat gating.
- Network/process inputs must remain explicit. Preserve URL validation and
  bounded output/error handling at external trust boundaries.

## Build and Validation

Run from the repository root:

```bash
cargo fmt --all -- --check
cargo check --all-targets
cargo test --all-targets
cargo clippy --all-targets -- -D warnings
RUSTDOCFLAGS="-D warnings" cargo doc --no-deps
cargo tree -e normal
cargo tree -d
cargo tree -i shard-tool-api
```

Prefer deterministic mocked HTTP/process tests. A live-service smoke test may
supplement them, but must not replace wire-shape, validation, and failure-path
coverage.

## Updating Pinned Revisions

`Cargo.toml` pins `shard-tool-api` by immutable `rev`. To update it:

1. merge and validate the tool-contract change first;
2. replace `rev` with the resulting commit SHA—never a branch name;
3. run `cargo update -p shard-tool-api` and the complete checks above;
4. confirm `cargo tree -i shard-tool-api` shows exactly one expected Git
   source;
5. coordinate the same tool-API revision with `shard-provider` and `shard-v2`
   before host cutover so Cargo cannot create duplicate nominal types.

When this crate changes, merge and validate it standalone before updating
`shard-v2` to the exact new Git revision.

## Host GUI Regression Matrix

This crate has no GUI. Validate behavior through the real `shard-v2` Tauri
application after portable changes:

- changed external tool: invocation, hook/event visibility, rendered output,
  and cache behavior owned by the host;
- dispatcher changes: one affected tool and one unrelated external tool;
- YouTube changes: a normal captioned video, rendered short transcript, long
  transcript with host-side summarization when practical, and unchanged
  heartbeat prohibition/bypass behavior;
- tool-contract changes: normal provider tool calling and UI event/output flow.

Use deterministic fixtures where live services are unstable, and state clearly
when credentials or live provider behavior were not exercised.
