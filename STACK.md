# Stack

Runtime and library stack for omnisearch. Architecture and module map: [ARCHITECTURE.md](ARCHITECTURE.md). Legal: [DISCLAIMER.md](DISCLAIMER.md), [LICENSE](LICENSE). Vulns: [SECURITY.md](SECURITY.md).

## Language and toolchain

| Piece | Choice |
| --- | --- |
| Language | **Rust** (edition **2024**) |
| MSRV | **1.88+** (`Cargo.toml` `rust-version`) |
| Toolchain file | `rust-toolchain.toml` → `stable` + `rustfmt` + `clippy` |
| Package / binary | crate + bin name **`omnisearch`** |
| Lockfile | `Cargo.lock` (builds use `--locked` in CI/release) |

## Application shape

| Layer | Tech |
| --- | --- |
| CLI | `clap` (derive) — `stdio` / `http` / `bench` |
| Async runtime | `tokio` (multi-thread) |
| MCP server | `rmcp` 3.x — stdio + streamable HTTP server features |
| Optional HTTP | `axum` 0.8 + `tower` / `tower-http` (CORS, limits) |
| Outbound HTTP | `reqwest` (rustls, JSON/form/query) — **no** default native-TLS |
| Serialization | `serde` / `serde_json` / `schemars` (tool schemas) |
| Concurrency helpers | `futures`, `dashmap`, `tokio-util` |
| Errors / logging | `thiserror`, `anyhow`, `tracing` + `tracing-subscriber` |
| Config | `dotenvy` + process environment only |
| Crypto-ish utils | `sha2`, `hex`, `base64` (checksums / encoding — not a KMS) |
| Network policy | `ipnet`, `url` — SSRF / URL helpers |

## Domain features (in-tree)

| Concern | Implementation |
| --- | --- |
| Multi-provider search | `src/providers/*` + `Registry` |
| Named accounts | `src/accounts.rs` + `OMNISEARCH_ACCOUNTS` |
| OmniRoute gateways | `src/providers/omniroute.rs` |
| Result fusion | RRF in `src/merge.rs` |
| Caching | In-process TTL (`src/cache.rs`) |
| Health / cooldown | `src/health.rs` |
| SSRF guard | `src/ssrf.rs` on fetch/extract/ground |

## Explicitly not in this stack

- No application database (Postgres/SQLite/etc.)
- No Redis / shared cache service (process-local only)
- No React/Svelte frontend — this is an MCP/HTTP server binary
- No Clerk/Auth0 product auth — HTTP mode uses shared bearer `AUTH_TOKENS`
- No Stripe or billing subsystem — **your** provider invoices are yours ([DISCLAIMER.md](DISCLAIMER.md))

## Test stack

| Tool | Use |
| --- | --- |
| `cargo test` | Rust unit + integration tests |
| `wiremock` | HTTP provider mocks (dev-dep) |
| `http` crate | Test helpers (dev-dep) |
| `tests/protocol_smoke.py` | MCP stdio protocol smoke (Python 3, CI) |

## CI / release

| Workflow | Role |
| --- | --- |
| `.github/workflows/ci.yml` | Forbid `CLAUDE.md`, `fmt`, `clippy -D warnings`, `test --locked`, build, protocol smoke |
| `.github/workflows/release.yml` | Tagged `v*` native binaries (Linux/macOS targets) + SHA256SUMS when tag matches `Cargo.toml` |

## Deploy / run targets

| Mode | Typical host |
| --- | --- |
| Stdio MCP | Local desktop (Cursor, Claude Desktop, etc.) |
| HTTP MCP | Localhost or private network only — not a public multi-tenant SaaS |
| Release artifacts | GitHub Releases tarballs (glibc Linux / macOS; see README) |

## Version pin policy

- Prefer updating via intentional PRs with lockfile review (supply-chain surface).
- Avoid drive-by dependency bumps unrelated to the change ([CONTRIBUTING.md](CONTRIBUTING.md), [AGENTS.md](AGENTS.md)).

## Related docs

- [README.md](README.md) — install, tools, providers
- [ARCHITECTURE.md](ARCHITECTURE.md) — modules and data flow
- [DISCLAIMER.md](DISCLAIMER.md) · [LICENSE](LICENSE) · [SECURITY.md](SECURITY.md)
