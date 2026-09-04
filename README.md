# shard-external-tools

`shard-external-tools` contains host-free implementations of Shard tools backed
by external HTTP services or explicitly supplied processes. It currently owns
portable weather, finance, Wikipedia, arXiv, web-search, URL-reading, and
YouTube transcript acquisition/parsing behavior.

The Shard host still owns tool registration and availability, credentials,
hooks, caching, persistence, UI events, model selection, long-transcript LLM
summarization, and heartbeat policy. The generic dispatcher intentionally does
not execute YouTube because the host must compose optional summarization. See
[AGENTS.md](AGENTS.md) for the complete boundary and GUI regression matrix.

## Repository graph

```text
shard-v2 ──> shard-tool-api
    ├──────> shard-external-tools ──> shard-tool-api
    └──────> shard-provider ─────────> shard-tool-api
```

Sibling repositories:

- [`shard-v2`](https://github.com/oupadhyay/shard-v2) — Tauri desktop host and UI
- [`shard-tool-api`](https://github.com/oupadhyay/shard-tool-api) — provider-neutral tool contracts
- [`shard-provider`](https://github.com/oupadhyay/shard-provider) — host-free provider transports

This crate may depend only on one immutable `shard-tool-api` revision among its
siblings. It must never depend on `shard-provider`.

## Consumption and releases

This crate is distributed from GitHub, not crates.io. Consumers must pin a
reviewed, immutable 40-character commit SHA:

```toml
[dependencies]
shard-external-tools = { git = "https://github.com/oupadhyay/shard-external-tools", rev = "<reviewed-commit-sha>" }
```

Its `shard-tool-api` dependency is likewise pinned by full Git SHA. The host,
this crate, and `shard-provider` must use that same tool-API revision to avoid
duplicate nominal Rust types. `publish = false` intentionally prevents
accidental crates.io publication.

## Host cutover status

The standalone-crate cutover was completed in
[`shard-v2` PR #123](https://github.com/oupadhyay/shard-v2/pull/123). The
initial host cutover consumed `shard-external-tools` revision
`b0fc45572caf17af2e8f46fb3f0a181f084ef9dd`. The authoritative record of the
revisions currently consumed by the host is the host's
[`Cargo.toml`](https://github.com/oupadhyay/shard-v2/blob/main/src-tauri/Cargo.toml)
and resolved
[`Cargo.lock`](https://github.com/oupadhyay/shard-v2/blob/main/src-tauri/Cargo.lock),
not this documentation branch or this repository's current HEAD.

Future portable external-tool changes must be merged and validated here first.
The host must then pin the resulting immutable revision and validate its
updated lockfile and dependency graph.

## Development

The repository pins its Rust toolchain in `rust-toolchain.toml` and commits
`Cargo.lock` so local and CI validation resolve the same registry and Git
dependencies.

```bash
cargo fmt --all -- --check
cargo check --locked --all-targets
cargo test --locked --all-targets
cargo clippy --locked --all-targets -- -D warnings
RUSTDOCFLAGS="-D warnings" cargo doc --locked --no-deps
cargo tree --locked -e normal
cargo tree --locked -d
cargo tree --locked -i shard-tool-api
python3 scripts/audit_dependency_boundary.py
```

Prefer deterministic mocked HTTP/process tests. Portable behavior changes also
require integration testing through the real `shard-v2` Tauri application as
described in [AGENTS.md](AGENTS.md).

## License

No open-source license has been selected for this repository. All rights are
reserved. The absence of a license file is deliberate; availability of the
source does not grant permission to use, copy, modify, or distribute it.
