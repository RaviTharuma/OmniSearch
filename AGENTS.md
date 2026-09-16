# AGENTS.md

Canonical instructions for coding agents working in this repository.

**Do not create or commit `CLAUDE.md`, `.claude/`, or other vendor-specific agent instruction files.** This project standardizes on **`AGENTS.md` only**. If a tool looks for `CLAUDE.md`, point it here or symlink locally outside git — never add those files to the repo.

## Project

- **omnisearch** — Rust MCP (and optional HTTP) multi-provider search server.
- Parallel provider fan-out, RRF merge, named accounts, OmniRoute gateways.
- License: Apache-2.0. Read [DISCLAIMER.md](DISCLAIMER.md) before shipping user-facing claims.

## Hard constraints

- No secrets in source, logs, tests, fixtures, health tools, or errors.
- Prefer extending existing adapters + `OMNISEARCH_ACCOUNTS` over new auth systems.
- Do not weaken SSRF / private-IP / metadata-host protections without an explicit reviewed design.
- Fail closed on bad account pins and invalid gateway config.
- Do not force-push `main` / `master`.
- Do not add `CLAUDE.md` (any casing) or Claude-only project instruction trees.

## Provider priority (when choosing work)

Web (in order): Linkup → Exa → Tavily → Brave → Firecrawl.

Social (same priority class): X → Reddit → Discord.

For each: multi-account via named credentials where applicable. See `docs/accounts-and-gateways.md`.

## Dev commands

Rust **1.88+** (`rust-toolchain.toml`).

```bash
cargo fmt
cargo clippy --all-targets -- -D warnings
cargo test --locked --all
cargo build --release
```

Live-key benches can burn paid credits — the human pays those bills, not the project.

## Docs map

| File | Use |
| --- | --- |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Modules, fan-out, merge, transports |
| [STACK.md](STACK.md) | Rust/crates/CI stack |
| [DISCLAIMER.md](DISCLAIMER.md) | Liability / supply chain / spend — do not soften |
| [LICENSE](LICENSE) | Apache-2.0 |
| [SECURITY.md](SECURITY.md) | Private vuln reporting only |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Human + agent contribution rules, DCO |
| [README.md](README.md) | User-facing install and tools |
| `docs/accounts-and-gateways.md` | Named accounts / gateways |

## Shipping

- Keep PRs focused; update `CHANGELOG.md` for user-visible changes.
- Align `Cargo.toml` / `Cargo.lock` versions with release tags when bumping.
- Green CI is not a security audit.
- Squash-merge is fine; follow the repo’s normal PR flow.
